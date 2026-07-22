# Proyecto-II Programación II — International Football FIFA World Cup

### Integrantes:
### Isaac Rodríguez Zúñiga
### Sebastián Calvo Solano

### Colegio Universitario de Cartago — Programación II

---

# Descripción General del Proyecto

Este proyecto realiza un análisis exploratorio de datos (EDA) sobre los partidos
históricos de la Copa Mundial de la FIFA, aplicando los conceptos de
**clases y objetos** vistos en el curso. El proyecto está dividido en cuatro
capas independientes, cada una con una única responsabilidad:

1. **Ingesta** (`CargadorDatos`) — descarga, filtra, enriquece, valida y
   persiste los datos crudos.
2. **Gestión** (`GestorPartidos`) — expone consultas de solo lectura sobre
   partidos individuales, equipos, años y sedes.
3. **EDA** (`ProcesadorEDA`) — realiza análisis agregado sobre todo el
   dataset: rankings, totales por equipo, comparativas entre selecciones.
4. **Visualización** (`Visualizador`) — genera gráficos que traducen los
   resultados del EDA en historias visuales.

---

# Dataset

El dataset utilizado es: https://github.com/martj42/international_results

Se filtra por la columna `tournament`, quedándose únicamente con los partidos
cuya descripción sea `"FIFA World Cup"`. El archivo original se descarga desde:

```
https://raw.githubusercontent.com/martj42/international_results/master/results.csv
```

### Contenido del dataset filtrado — Columnas y variables

| # | Column | Non-Null Count | Dtype | Descripción |
| --- | --- | --- | --- | --- |
| - | id | *( )* | int64 | Índice original de la fila en `raw_results.csv`, usado para trazabilidad al archivo crudo |
| 0 | date | *( )* | datetime64[ns] | Fecha del partido (`YYYY-MM-DD`) |
| 1 | home_team | *( )* | str | Equipo local |
| 2 | away_team | *( )* | str | Equipo visitante |
| 3 | home_score | *( , con nulos)* | float64 | Goles del equipo local |
| 4 | away_score | *( , con nulos)* | float64 | Goles del equipo visitante |
| 5 | tournament | *( )* | str | Torneo (filtrado por `"FIFA World Cup"`) |
| 6 | city | *( )* | str | Ciudad sede del partido |
| 7 | country | *( )* | str | País sede del partido |
| 8 | neutral | *( )* | bool | Indica si el partido se jugó en cancha neutral (`True`/`False`) |
| 9 | clave_partido | *( )* | str | Clave estable del partido (`fecha_equipoLocal_equipoVisitante`), usada como identificador de negocio independiente del orden del archivo |

---

# Arquitectura del Proyecto

```
src/
├── ingesta/
│   └── CargadorDatos.py      # Descarga, filtra, enriquece, valida, persiste
├── gestor/
│   └── GestorPartidos.py     # Consultas de solo lectura (un partido, un equipo, un año...)
├── eda/
│   └── EDA.py                 # ProcesadorEDA: rankings y agregados sobre todo el dataset
├── visualizacion/
│   └── Visualizador.py        # Gráficos que consumen ProcesadorEDA
└── helpers/                   # Utilidades genéricas reutilizables

data/
├── raw/                       # CSV crudo descargado + procesado, según especificación del proyecto
└── processed/                 # Reservado para uso futuro

notebooks/
├── 01_Visualizador.ipynb      # Narrativa visual completa con Visualizador
├── 02_EDA.ipynb               # Exploración con ProcesadorEDA
├── 03_CheckGestor.ipynb       # Pruebas de GestorPartidos
└── 04_CleanRaw.ipynb          # Utilidades de limpieza de datos

dashboard/                      # Dashboard de Streamlit (ver sección de Puntos Extra)
docs/                           # Documentación del proyecto (enunciado, etc.)
```

### Flujo de dependencias

```
CargadorDatos  →  GestorPartidos  →  ProcesadorEDA  →  Visualizador
   (ingesta)         (consulta)         (análisis)        (gráficos)
```

Cada capa confía en que la anterior ya validó y dejó los datos en el formato
esperado — ninguna capa revalida el trabajo de la capa previa, siguiendo el
principio de responsabilidad única.

---

# Instalación del entorno (Conda)

1. Crear el entorno: `conda env create -f env.yml`
2. Activar el entorno: `conda activate FifaWC`
3. Desactivar el entorno: `conda deactivate`

Para sincronizar el entorno si `env.yml` cambia después de la creación inicial:
```bash
conda env update --name FifaWC --file env.yml --prune
```

---

# Cómo ejecutar el proyecto

Todos los comandos se ejecutan desde la **raíz del proyecto** (donde vive `env.yml`):

```bash
# Prueba del pipeline de ingesta
python -m src.ingesta.CargadorDatos

# Prueba del EDA
python -m src.eda.prueba

# Prueba de visualización
python -m src.visualizacion.pruebav
```

Alternativamente, todo el flujo puede explorarse de forma interactiva a través
de los notebooks en `notebooks/`, en el orden numérico sugerido por sus nombres.

---

# Clases principales

### `CargadorDatos`
Responsable de: descargar el CSV (con cache local para evitar descargas
repetidas), filtrar los partidos de Copa Mundial, enriquecer el dataset
(conversión de fechas, generación de `clave_partido`), validar la integridad
mínima de los datos, y persistir tanto la versión cruda como la procesada.

### `GestorPartidos`
Expone métodos de solo lectura: `get_partido`, `get_partido_por_id`,
`get_por_equipo`, `get_por_anio`, `get_por_sede`, `ventaja_local`,
`promedio_goles_por_partido`, `partido_mas_goles_por_partido`,
`get_goles_por_equipo`, entre otros.

### `ProcesadorEDA`
Genera tablas de análisis agregado: `goles_favor`, `goles_contra`,
`diferencia_goles`, `victorias`, `derrotas`, `empates`, `mas_gol_mundial`,
`menos_gol_mundial`, `veces_sede`, `ranking_mundial`, `correlacion`,
`descripcion`, `outliers`.

### `Visualizador`
Traduce los resultados de `ProcesadorEDA` en gráficos: diferencia de goles
(tornado), evolución de goles por edición (línea de tiempo), top 5 histórico
(radar), sede vs. campeón (barras comparativas), y subcampeonatos (barras
horizontales).

---

# Fuente de datos

Dataset original: *International football results from 1872 to present*,
de Mart Jürisoo — https://github.com/martj42/international_results