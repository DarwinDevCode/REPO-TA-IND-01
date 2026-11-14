# Bitácora de trabajo — TA-IND-01

**Estudiante:** Sánchez Vera Darwin Joel

**Asignatura:** Aplicaciones Distribuidas

**Sistema analizado:** Redis Cluster

**Sistema de comparación:** Hazelcast

---

## 12/11/2026 — Sesión 1

**Horario:** 19:44 – 22:22 (aprox. 2 h 40 min)

**Qué hice**

- Leí las instrucciones de la actividad TA-IND-01 y los requisitos de bibliografía y evidencias.

- Revisé varios sistemas distribuidos candidatos. Primero consideré Apache Cassandra con MongoDB, y al final elegí Redis Cluster como sistema principal y Hazelcast como sistema de comparación.

- Busqué bibliografía de acceso abierto en Crossref, DOAJ y en las páginas de las editoriales, con apoyo de IA (Claude). Abrí los DOI para comprobar autores, título, revista, volumen y páginas.

- Creé informe/referencias.bib con los artículos y la documentación oficial de Redis y Hazelcast.

- Creé la estructura de carpetas del repositorio (informe/, figuras/, capturas/, experimento/) e hice los primeros commits.

**Problemas encontrados y cómo los resolví**

1. Cassandra y MongoDB pasaron a la lista de sistemas excluidos debido a que ya habían sido escogidos. Busqué otra pareja con buena documentación de su arquitectura, con artículos disponibles y que se pudiera ejecutar con Docker, y así llegué a Redis Cluster y Hazelcast.

2. Muchos artículos sobre Redis estaban en revistas de calidad dudosa procedencia o eran de pago. Me quedé con revistas indexadas de acceso abierto (*Journal of Cloud Computing*, *IEEE Access*, *Alexandria Engineering Journal*) y con la documentación oficial, y descarté los artículos de pago.

3. Git no guarda carpetas vacías. Lo resolví poniendo un archivo .gitkeep en las carpetas que todavía no tienen contenido.

**Tiempo dedicado:** aprox. 2 h 40 min

---

## 13/11/2026 — Sesión 2

**Horario:** 18:56pm - 20:19pm

**Qué hice**

- Revisé el estado del repositorio frente a las instrucciones y apunté lo pendiente: la bitácora estaba vacía, el README incompleto y faltaban commits en otros días.

- Revisé la especificación oficial de Redis Cluster (slots, nodos maestros, réplicas, bus del clúster, failover) para preparar el esquema inicial de la arquitectura.

- Dibujé a mano el esquema inicial de la arquitectura para el Anexo A.

**Problemas encontrados y cómo los resolví**

- - No entendía qué eran los slots, ni para qué servían las réplicas. Lo resolví repasando la especificación con apoyo de IA (Claude) y con un diagrama interactivo de prueba, en el que vi en qué slot cae cada clave y cómo una réplica reemplaza a su maestro cuando este falla.

- Estuve a punto de subir la foto del esquema sin mi nombre y la fecha, que son obligatorios. Los agregué en la hoja antes de subirla.

**Tiempo dedicado:** aprox. 1 h 23 min

---

## 13/11/2026 — Sesión 3

**Horario:** 21:04 – 21:32

**Qué hice**
- Instalé Docker Desktop con WSL 2.
- Levanté un clúster Redis de 6 nodos (3 maestros y 3 réplicas) con Docker y tomé 4 capturas: contenedores activos, creación del clúster, estado de los nodos y escritura/lectura con redirección entre nodos.
- Guardé los comandos y las salidas en la carpeta `experimento/`.

**Problemas encontrados y cómo los resolví**
- Docker no estaba instalado y WSL estaba dañado (error `Wsl/CallMsi/Install/REGDB_E_CLASSNOTREG`). Dejé que WSL se reparara solo y reinicié el equipo.
- `wsl --status` decía que la virtualización no estaba habilitada, aunque en el Administrador de tareas aparecía como habilitada. Faltaba activar la "Plataforma de máquina virtual" con `wsl.exe --install --no-distribution` y reiniciar.

**Tiempo dedicado:** aprox. 0 h 26 min

---

## 13/11/2026 — Sesión 4

**Horario:** 23:14 – 23:52

**Qué hice**
- Realicé el experimento E1 de tolerancia a fallos: apagué el maestro redis-2 (172.30.0.12), que guardaba la clave `usuario:25`, y medí cuánto tardaba su réplica redis-6 en reemplazarlo.
- El failover tardó 7.8 segundos. Entre los 6.2 y 7.1 s el clúster estuvo en estado `fail`, y después redis-6 quedó como maestro de los slots 5461–10922.
- Comprobé que `usuario:25` seguía disponible después del failover.
- Volví a encender redis-2 y verifiqué que regresó como réplica del nuevo maestro.
- Grabé la sesión con `Start-Transcript` y documenté el experimento en `experimento/02-e1-failover.md`.

**Problemas encontrados y cómo los resolví**
- guardé las capturas del experimento con nombres equivocados y tuve que revisarlas y renombrarlas

**Tiempo dedicado:** aprox. 0 h 38 min

---

## 14/11/2026 — Sesión 5

**Horario:** 00:00 (aprox.) – 00:58

**Qué hice**
- Construí en draw.io el diagrama final de la arquitectura de Redis Cluster, partiendo de mi esquema a mano (Anexo A) y usando como referencia visual un diagrama generado con Claude.
- Incluí la red `redisnet`, los seis nodos con sus contenedores e IPs reales, los rangos de slots, la asignación real de réplicas, los puertos 6379 y 16379, la replicación asíncrona y la redirección MOVED.
- Guardé el archivo fuente `figuras/diagrama_arquitectura.drawio` y lo exporté a PDF recortado para el informe.
- Revisé qué páginas de los artículos de la bibliografía tenían información útil para las lecturas del Anexo B.

**Problemas encontrados y cómo los resolví**
- No sabía cómo guardar el archivo fuente ni exportarlo sin márgenes; lo guardé como `.drawio` y lo exporté a PDF con la opción Recortar.

**Tiempo dedicado:** aprox. 0 h 58 min

## 14/11/2026 — Sesión 6

**Horario:** 07:39 – 10:00 (aprox.)

**Qué hice**
- Leí los artículos de Ahmed et al. (2023), Ferreira et al. (2024) y Gencer et al. (2021), subrayé las partes útiles y tomé 4 capturas con el número de página visible (Anexo B).
- Encontré que Ahmed et al. clasifican a Redis como sistema CP, lo que no coincide con lo que observé en mi experimento E1; lo usé en la postura crítica.
- Creé `informe/main.tex` con la estructura completa del informe y, con apoyo de Claude, redacté la introducción, la arquitectura, el análisis de la Unidad 1, la experimentación, la evolución del diseño y la postura crítica.
- Completé en la carátula los datos del docente y del paralelo, y configuré los enlaces en azul.
- Realicé dos ejemplos de mi propio entorno: un conflicto entre réplicas en Git/GitHub (push rechazado, conflicto en `README.md` y merge) y el acceso al sitio web de la UTEQ con y sin red, grabado con `Start-Transcript`.
- Redacté la comparación con Hazelcast a partir de su documentación oficial.

**Problemas encontrados y cómo los resolví**
- La primera captura de la pág. 3112 no mostraba el número de página y en la de la pág. 172 faltaba la línea donde Redis aparece como CP; repetí ambas.
- Intercambié los nombres de las dos capturas de Gencer y una quedó con extensión duplicada (`.png.png`); las renombré.
- Escribí mal `Start-Transcript` y usé `-\` en lugar de `.\` en la ruta del archivo de salida.
- Sin red, la consola mostró un `200` que en realidad era la respuesta anterior guardada en la variable `$r`; lo documenté como un ejemplo de dato desactualizado.
- En un commit copié la palabra "Summary:" dentro del mensaje.

**Tiempo dedicado:** aprox. 2 h 21 min

## 14/11/2026 — Sesión 7

**Horario:** 12:53 – 13:45 (aprox.)

**Qué hice**
- Subí las capturas de los ejemplos de mi entorno, la transcripción y los cambios del informe.
- Redacté, con apoyo de Claude, el resumen (192 palabras) y las conclusiones, y actualicé la declaración de uso de inteligencia artificial.
- Completé el `README.md` con el enlace al video, las instrucciones de compilación y la estructura del repositorio.
- Preparé en LaTeX la carátula de una página para el SGA con la URL del repositorio.

**Problemas encontrados y cómo los resolví**
- El README no tenía todo lo que pide la actividad (enlace al video, compilador, archivo principal y dependencias); lo completé.

**Tiempo dedicado:** aprox. 0 h 52 min