# Análisis y predicción del rendimiento productivo en pozos de petróleo y gas de Argentina

Proyecto final del programa **EnergIA Digital — Data Science** (Fundación YPF).

**Integrantes:** _completar_ · **Comisión:** _completar_

## Dataset

*Producción de petróleo y gas por pozo (Capítulo IV)*, publicado por la Secretaría de Energía de la Nación en
[datos.energia.gob.ar](http://datos.energia.gob.ar/dataset/produccion-de-petroleo-y-gas-por-pozo). Usamos el
archivo del año 2025, descargado el 02/10/2026 y guardado sin cambios, sólo comprimido, en
`data/produccion_pozos_2025.csv.gz` (991.936 filas, 39 columnas). Cada fila es la declaración jurada mensual de un
pozo: petróleo en m³, gas en miles de m³, agua en m³, días efectivos de producción y datos del pozo (cuenca,
provincia, formación, sistema de extracción, profundidad, tipo de recurso).

## Resumen

Estudiamos la producción de petróleo de los pozos argentinos para entender qué características del pozo explican
cuánto produce. El trabajo tiene tres etapas: un análisis exploratorio de los volúmenes de petróleo, gas y agua por
provincia, cuenca, formación y sistema de extracción; un modelo supervisado que predice la producción mensual de
petróleo de un pozo; y un modelo no supervisado que agrupa pozos según su perfil productivo.

## Objetivo

Predecir la producción mensual de petróleo de un pozo (`prod_pet`) a partir de sus características técnicas y su
ubicación, y segmentar los pozos en grupos con comportamiento productivo similar.

## Hallazgos del análisis exploratorio

Los pozos no convencionales son el 8,4 % de las declaraciones de los pozos petroleros en producción, pero aportan
el 62,3 % del petróleo, con una mediana de 665 m³/mes contra 39 m³/mes de un pozo convencional. Por eso la
cuenca Neuquina, con menos pozos que Golfo San Jorge, produjo el 73 % del petróleo. La producción es muy
asimétrica (skew 9,07); Yeo-Johnson la lleva a 0,01. Los valores atípicos que marca el rango intercuartílico son,
en su mayoría, pozos no convencionales válidos, así que no se eliminaron. El dataset trae faltantes de tres
clases: estructurales (el subtipo de recurso no aplica a los convencionales), dependientes de la provincia que
carga el dato, y ocultos como ceros (12 % de las profundidades). El detalle está en el notebook 01.

## Modelos

- **Supervisado (pre-entrega 3):** pendiente.
- **No supervisado (pre-entrega 4):** pendiente.

## Estructura

```text
├── data/
│   ├── produccion_pozos_2025.csv.gz          # dataset original, comprimido
│   └── processed/
│       └── pozos_petroleros_2025.csv.gz      # salida del notebook 01
├── notebooks/
│   └── 01_analisis_exploratorio.ipynb        # pre-entrega 2
├── requirements.txt
└── README.md
```

## Cómo ejecutarlo

**Google Colab:** en una celda al comienzo del notebook, clonar el repositorio y ubicarse en `notebooks/`:

```python
!git clone <url-del-repositorio>
%cd energia-digital-proyecto-final/notebooks
```

**Anaconda o Python local:**

```bash
git clone <url-del-repositorio>
cd energia-digital-proyecto-final
pip install -r requirements.txt
jupyter notebook notebooks/01_analisis_exploratorio.ipynb
```

Los notebooks leen los datos con rutas relativas (`../data/...`), así que deben ejecutarse desde la carpeta
`notebooks/`.
