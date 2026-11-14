# Levantar el clúster Redis con Docker

**Entorno:** Windows, PowerShell 5.1, Docker Desktop (WSL 2), imagen `redis:8`
**Equipo:** PC-DARWIN · **Usuario:** djsan
**Fecha:** 13/11/2026

## 1. Red propia con IPs fijas

```powershell
docker network create --subnet=172.30.0.0/16 redisnet
```

Redis Cluster no funciona bien con IPs o puertos remapeados (NAT). Por eso cada nodo tiene una IP fija dentro de una red propia y el cliente se ejecuta dentro de esa misma red.

## 2. Seis nodos en modo clúster

```powershell
1..6 | ForEach-Object { docker run -d --name "redis-$_" --net redisnet --ip "172.30.0.1$_" redis:8 redis-server --cluster-enabled yes --cluster-node-timeout 5000 --appendonly yes --protected-mode no }
docker ps
```

- `--cluster-enabled yes`: el nodo trabaja en modo clúster.
- `--cluster-node-timeout 5000`: si un nodo no responde en 5 segundos, se le considera caído.
- `--appendonly yes`: los datos se guardan en disco.

Captura: `capturas/sistema/01-docker-ps.png` · Salida: `salidas/01-docker-ps.txt`

## 3. Crear el clúster (3 maestros + 3 réplicas)

```powershell
docker exec -it redis-1 redis-cli --cluster create 172.30.0.11:6379 172.30.0.12:6379 172.30.0.13:6379 172.30.0.14:6379 172.30.0.15:6379 172.30.0.16:6379 --cluster-replicas 1
```

Resultado: `[OK] All 16384 slots covered.`

| Maestro | Slots | Réplica |
|---|---|---|
| 172.30.0.11 (redis-1) | 0 – 5460 | 172.30.0.15 (redis-5) |
| 172.30.0.12 (redis-2) | 5461 – 10922 | 172.30.0.16 (redis-6) |
| 172.30.0.13 (redis-3) | 10923 – 16383 | 172.30.0.14 (redis-4) |

Captura: `capturas/sistema/02-cluster-create.png`

## 4. Estado del clúster

```powershell
docker exec redis-1 redis-cli cluster info
docker exec redis-1 redis-cli cluster nodes
```

Resultado: `cluster_state:ok`, `cluster_slots_assigned:16384`, `cluster_known_nodes:6`, `cluster_size:3`.

Observaciones:
- Los contadores `cluster_stats_messages_ping_sent` y `cluster_stats_messages_pong_received` muestran los mensajes que los nodos se intercambian entre sí (gossip).
- En `cluster nodes`, cada nodo aparece como `IP:6379@16379`. El puerto 16379 es el bus del clúster, que los nodos usan para comunicarse entre ellos.

Captura: `capturas/sistema/03-cluster-info-nodes.png` · Salidas: `salidas/02-cluster-info.txt`, `salidas/03-cluster-nodes.txt`

## 5. Escritura y lectura con redirección entre nodos

```
docker exec -it redis-1 redis-cli -c
CLUSTER KEYSLOT usuario:25
SET usuario:25 "Darwin Sanchez"
GET usuario:25
SET materia:aplicaciones "Distribuidas"
GET materia:aplicaciones
```

| Clave | Slot | Nodo que la guarda |
|---|---|---|
| `usuario:25` | 7554 | 172.30.0.12 (maestro B) |
| `materia:aplicaciones` | 13574 | 172.30.0.13 (maestro C) |

El cliente se conectó a redis-1 (172.30.0.11). Como ninguna de las dos claves le pertenecía, el clúster respondió con una redirección (`-> Redirected to slot [...]`) y el cliente repitió la petición en el nodo dueño del slot.

Captura: `capturas/sistema/04-set-get-redireccion.png`