# REPO-TA-IND-01

Informe individual **TA-IND-01** de Aplicaciones Distribuidas (UTEQ):
análisis crítico de **Redis Cluster** y comparación con **Hazelcast**.

**Autor:** Sánchez Vera Darwin Joel · 7mo B

## Video

PENDIENTE

## Estructura del repositorio

- `informe/`: fuente LaTeX (`main.tex`) y bibliografía (`referencias.bib`).
- `figuras/`: diagrama de arquitectura (draw.io) y esquema a mano.
- `capturas/sistema/`: capturas del clúster y del experimento E1.
- `capturas/lecturas/`: lecturas subrayadas (Anexo B).
- `capturas/entorno/`: ejemplos de mi propio entorno.
- `experimento/`: comandos, procedimientos y salidas en texto.
- `BITACORA.md`: registro de las sesiones de trabajo.

## Compilación del informe

Desde la carpeta `informe/`:

```
pdflatex main
biber main
pdflatex main
pdflatex main
```