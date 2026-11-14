# REPO-TA-IND-01

Informe individual **TA-IND-01** de Aplicaciones Distribuidas (UTEQ):
análisis crítico de **Redis Cluster** como sistema distribuido y comparación con **Hazelcast**.

- **Autor:** Darwin Joel Sánchez Vera
- **Carrera:** Ingeniería de Software · 7mo semestre, paralelo B
- **Docente:** Guerrero Ulloa Gleiston Ciceron
- **Informe final (PDF):** [`informe/main.pdf`](informe/main.pdf)

## Video

Presentación del trabajo (3–5 minutos):
[https://drive.google.com/drive/folders/1EXjpkWFxgiclZxOAC9IKSGzo-7Hj4GJy?usp=sharing](https://drive.google.com/drive/folders/1EXjpkWFxgiclZxOAC9IKSGzo-7Hj4GJy?usp=sharing)

## Contenido del informe

1. Introducción
2. Arquitectura de Redis Cluster (diagrama propio y evolución desde el esquema a mano)
3. Análisis según la Unidad 1: transparencias, CAP/PACELC, modelos de comunicación y tolerancia a fallos
4. Experimentación propia: despliegue de un clúster de 6 nodos con Docker, experimento E1 (caída de un maestro) y dos ejemplos de mi propio entorno
5. Comparación con Hazelcast
6. Postura crítica
7. Conclusiones
- Declaración de uso de inteligencia artificial, referencias (IEEE) y anexos A y B

## Estructura del repositorio

```
REPO-TA-IND-01/
├── informe/
│   ├── main.tex              # fuente LaTeX del informe
│   ├── referencias.bib       # bibliografía (biblatex, estilo IEEE)
│   └── main.pdf              # informe compilado
├── figuras/                  # diagrama de arquitectura (draw.io + PDF) y esquema a mano (Anexo A)
├── capturas/
│   ├── sistema/              # despliegue del clúster y experimento E1
│   ├── lecturas/             # lecturas subrayadas con número de página (Anexo B)
│   └── entorno/              # ejemplos de mi propio entorno (Git/GitHub y sitio web de la UTEQ)
├── experimento/              # comandos, procedimientos y salidas en texto
│   └── salidas/
├── BITACORA.md               # registro de las sesiones de trabajo
└── README.md
```

## Compilación del informe

| | |
|---|---|
| **Compilador** | pdfLaTeX |
| **Gestor de bibliografía** | Biber (biblatex, estilo IEEE) |
| **Archivo principal** | `informe/main.tex` |
| **Bibliografía** | `informe/referencias.bib` |
| **Resultado** | `informe/main.pdf` |

### Dependencias

- Distribución LaTeX: [MiKTeX](https://miktex.org/) (Windows) o [TeX Live](https://tug.org/texlive/) (Linux/macOS).
- Programa `biber` (incluido en MiKTeX y TeX Live).
- Paquetes LaTeX utilizados: `babel` (español e inglés), `fontenc`, `inputenc`, `lmodern`, `microtype`, `geometry`, `setspace`, `parskip`, `graphicx`, `float`, `booktabs`, `tabularx`, `array`, `caption`, `xcolor`, `listings`, `biblatex`, `biblatex-ieee`, `xpatch`, `hyperref` y `csquotes`. MiKTeX instala automáticamente los que falten la primera vez que se compila.

### Orden de compilación

Desde la carpeta `informe/`, ejecutar en este orden:

```
pdflatex main
biber main
pdflatex main
pdflatex main
```

1. **pdflatex**: primera pasada; genera `main.bcf` con las citas del documento.
2. **biber**: procesa `referencias.bib` y genera la bibliografía (`main.bbl`).
3. **pdflatex**: inserta la bibliografía en el documento.
4. **pdflatex**: actualiza el índice, los números de figuras y tablas y las referencias cruzadas.

Las imágenes se cargan desde `figuras/` y `capturas/` con rutas relativas, por lo que se debe compilar desde `informe/` sin mover las carpetas del repositorio.

**En Overleaf:** subir el repositorio completo, seleccionar `informe/main.tex` como documento principal y pdfLaTeX como compilador (Overleaf ejecuta `biber` automáticamente).


## Reproducir el clúster del experimento

Requiere Docker Desktop. Los comandos completos y las salidas están en [`experimento/01-levantar-cluster.md`](experimento/01-levantar-cluster.md) y [`experimento/02-e1-failover.md`](experimento/02-e1-failover.md).

```powershell
docker network create --subnet=172.30.0.0/16 redisnet
1..6 | ForEach-Object { docker run -d --name "redis-$_" --net redisnet --ip "172.30.0.1$_" redis:8 redis-server --cluster-enabled yes --cluster-node-timeout 5000 --appendonly yes --protected-mode no }
docker exec -it redis-1 redis-cli --cluster create 172.30.0.11:6379 172.30.0.12:6379 172.30.0.13:6379 172.30.0.14:6379 172.30.0.15:6379 172.30.0.16:6379 --cluster-replicas 1
```