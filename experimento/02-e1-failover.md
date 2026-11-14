# Experimento E1: caída de un nodo maestro (failover)

**Concepto de la Unidad 1:** tolerancia a fallos (y relación con CAP)
**Entorno:** clúster de `experimento/01-levantar-cluster.md` (3 maestros + 3 réplicas, `cluster-node-timeout 5000`)
**Equipo:** PC-DARWIN · **Usuario:** djsan
**Fecha:** 13/11/2026

## 1. Objetivo

Comprobar qué pasa con el servicio y con los datos cuando se cae un nodo maestro de Redis Cluster: cómo se detecta el fallo, cuánto tarda la recuperación y si se pierde el dato guardado.

## 2. Hipótesis

Según la especificación oficial de Redis Cluster, cuando un maestro deja de responder durante más tiempo que `cluster-node-timeout`, los otros maestros lo marcan como caído y su réplica es elegida como nuevo maestro. Por eso se espera que:

- el failover tarde algo más de 5 segundos (el `cluster-node-timeout` configurado);
- la réplica `172.30.0.16` (redis-6) pase a ser maestro de los slots 5461–10922;
- la clave `usuario:25` (slot 7554) siga disponible después del failover.

## 3. Procedimiento y comandos

Toda la sesión se grabó con `Start-Transcript` en `salidas/e1-failover.txt`.

### 3.1 Estado inicial

```powershell
docker exec redis-1 redis-cli cluster nodes
docker exec redis-1 redis-cli -c GET usuario:25
```

### 3.2 Caída del maestro B y medición del failover

```powershell
docker stop redis-2
$t0 = Get-Date
do {
  Start-Sleep -Milliseconds 500
  $seg = [math]::Round(((Get-Date) - $t0).TotalSeconds, 1)
  $estado = (docker exec redis-1 redis-cli cluster info | Select-String "cluster_state").ToString().Trim()
  $nodo16 = (docker exec redis-1 redis-cli cluster nodes | Select-String "172.30.0.16").ToString()
  $rol16 = if ($nodo16 -match " master ") { "MASTER" } else { "replica" }
  Write-Host "$seg s | $estado | redis-6: $rol16"
} until ($rol16 -eq "MASTER")
Write-Host "Failover completado en $seg segundos"
```

### 3.3 Estado después del failover

```powershell
docker exec redis-1 redis-cli cluster nodes
docker exec redis-1 redis-cli -c GET usuario:25
```

### 3.4 Recuperación del nodo caído

```powershell
docker start redis-2
Start-Sleep -Seconds 10
docker exec redis-1 redis-cli cluster nodes
```

## 4. Resultados

### 4.1 Estado inicial (23:15)

| Nodo | Rol | Slots |
|---|---|---|
| 172.30.0.12 (redis-2) | master | 5461 – 10922 |
| 172.30.0.16 (redis-6) | slave de 172.30.0.12 | — |

`GET usuario:25` → `Darwin Sanchez`

Captura: `capturas/sistema/05-e1-estado-inicial.png`

### 4.2 Línea de tiempo del failover

| Tiempo desde `docker stop` | `cluster_state` | Rol de redis-6 |
|---|---|---|
| 0.6 s – 5.5 s | ok | réplica |
| 6.2 s – 7.1 s | **fail** | réplica |
| 7.8 s | ok | **MASTER** |

**Failover completado en 7.8 segundos.**

Captura: `capturas/sistema/06-e1-failover.png`

### 4.3 Estado después del failover (23:19)

| Nodo | Rol | Slots |
|---|---|---|
| 172.30.0.12 (redis-2) | `master,fail` (caído) | — |
| 172.30.0.16 (redis-6) | **master** | 5461 – 10922 |

- El número de época de configuración de redis-6 pasó de **2** a **7**.
- `GET usuario:25` → `Darwin Sanchez`: el dato siguió disponible.

Captura: `capturas/sistema/06-e1-failover.png` (misma captura que 4.2)

### 4.4 Recuperación del nodo caído

Después de `docker start redis-2` (23:20) y 10 segundos de espera:

| Nodo | Rol | Slots |
|---|---|---|
| 172.30.0.12 (redis-2) | **slave** de 172.30.0.16 | — |
| 172.30.0.16 (redis-6) | master | 5461 – 10922 |

El nodo caído volvió al clúster, pero no recuperó su papel de maestro: quedó como réplica del nuevo maestro, con la misma época de configuración (7).

Captura: `capturas/sistema/07-e1-recuperacion.png`

## 5. Interpretación

**Detección del fallo.** Cuando apagué redis-2, los demás nodos dejaron de recibir sus respuestas a los mensajes de control (ping/pong) que se envían por el bus del clúster. Redis no da por caído a un nodo de inmediato: primero espera el cluster-node-timeout, que en este clúster es de 5 segundos. Pasado ese tiempo, cada maestro lo marca como posiblemente caído y, cuando la mayoría de maestros coincide, el fallo se confirma. Eso explica que el estado cambiara a fail recién a los 6.2 s y no antes. Después, la réplica redis-6 pidió el voto de los maestros restantes, fue elegida y subió su época de configuración de 2 a 7. Con eso el clúster sabe que la configuración nueva es la válida. El tiempo total, 7.8 s, coincide con lo esperado: el timeout configurado más el tiempo de confirmación y de votación.

**Los primeros 5.5 segundos.** Aunque el nodo ya estaba apagado, el clúster seguía informando cluster_state:ok, porque todavía no había confirmado el fallo. En ese intervalo, el clúster todavía tenía registrado a redis-2 como dueño de los slots 5461–10922, así que una petición sobre usuario:25 se habría dirigido a ese nodo apagado y habría fallado. No lo comprobé con un cliente durante esa ventana, pero se deduce de la especificación. Esto muestra que el fallo no es invisible para el usuario: hay un periodo en el que una parte de los datos no está disponible.

**El estado fail y CAP.** Entre 6.2 y 7.1 s el clúster completo pasó a fail, no solo los slots del nodo caído. Según la documentación, por defecto Redis Cluster deja de atender consultas si algún rango de slots se queda sin maestro (cluster-require-full-coverage yes). En ese momento prioriza no responder con una vista incompleta de los datos antes que seguir disponible, es decir, sacrifica disponibilidad durante unos segundos. Sin embargo, esto no lo convierte en un sistema CP: como la réplica se actualiza de forma asíncrona, no garantiza que todas las escrituras confirmadas sobrevivan a un failover.

**Por qué no se perdió el dato.** La clave usuario:25 se había escrito unas dos horas antes de apagar el nodo, así que la réplica ya tenía la copia. Si la escritura se hubiera hecho justo antes de la caída, el maestro podría haber respondido "OK" al cliente sin haber enviado todavía el cambio a su réplica, y ese dato se habría perdido al promoverla. Esto refleja la decisión que describe PACELC: en funcionamiento normal Redis Cluster prioriza la latencia (responde sin esperar a la réplica) por encima de la consistencia fuerte.

**Recuperación y transparencia de fallos.** Al volver a encender redis-2, regresó como réplica del nuevo maestro y no como maestro. Así se evita que existan dos maestros para los mismos slots: gana la configuración con la época más alta (7). El cliente no tuvo que cambiar nada para seguir leyendo usuario:25, porque el clúster lo redirige automáticamente al nuevo maestro. Por eso Redis Cluster ofrece una transparencia de fallos **parcial**: oculta el cambio de nodo una vez terminado el failover, pero no oculta los segundos de indisponibilidad mientras ocurre.

## 6. Limitaciones del experimento

- Se hizo una sola ejecución; el tiempo de failover puede variar entre repeticiones.
- No se midieron peticiones de un cliente durante la ventana de fallo; el efecto sobre el cliente se deduce de la documentación.
- Todos los nodos estaban en el mismo equipo, sin latencia de red real entre ellos.