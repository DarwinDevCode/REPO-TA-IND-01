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

**Horario:** 21:04 – 21:30

**Qué hice**
- Instalé Docker Desktop con WSL 2.
- Levanté un clúster Redis de 6 nodos (3 maestros y 3 réplicas) con Docker y tomé 4 capturas: contenedores activos, creación del clúster, estado de los nodos y escritura/lectura con redirección entre nodos.
- Guardé los comandos y las salidas en la carpeta `experimento/`.

**Problemas encontrados y cómo los resolví**
- Docker no estaba instalado y WSL estaba dañado (error `Wsl/CallMsi/Install/REGDB_E_CLASSNOTREG`). Dejé que WSL se reparara solo y reinicié el equipo.
- `wsl --status` decía que la virtualización no estaba habilitada, aunque en el Administrador de tareas aparecía como habilitada. Faltaba activar la "Plataforma de máquina virtual" con `wsl.exe --install --no-distribution` y reiniciar.

**Tiempo dedicado:** aprox. 0 h 26 min