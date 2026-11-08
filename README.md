# Análisis y predicción del rendimiento productivo en pozos de petróleo y gas de Argentina

Proyecto final del programa **EnergIA Digital — Data Science** (Fundación YPF).

**Integrantes:** Mariapia Ortiz y Antonella Fontanetto · **Comisión:** 2

## Dataset

*Producción de petróleo y gas por pozo (Capítulo IV)*, publicado por la Secretaría de Energía de la Nación en
[datos.energia.gob.ar](http://datos.energia.gob.ar/dataset/produccion-de-petroleo-y-gas-por-pozo). Usamos el
archivo del año 2025, descargado el 02/10/2026 y guardado sin cambios, sólo comprimido, en
`data/produccion_pozos_2025.csv.gz` (991.936 filas, 39 columnas). Cada fila es la declaración jurada mensual de un
pozo: petróleo en m³, gas en miles de m³, agua en m³, días efectivos de producción y datos del pozo (cuenca,
provincia, formación, sistema de extracción, profundidad, tipo de recurso).

## Resumen

Queremos entender por qué algunos pozos de petróleo de Argentina producen mucho y otros poco, armar modelos que lo
predigan y agrupar los pozos que se comportan de forma parecida. El trabajo tiene tres etapas: un análisis
exploratorio de los volúmenes de petróleo, gas y agua por provincia, cuenca, formación y sistema de extracción; un
modelo supervisado que predice la producción mensual de petróleo de un pozo; y un modelo no supervisado que agrupa
pozos según su perfil productivo.

## Objetivo

Predecir la producción mensual de petróleo de un pozo (`prod_pet`) a partir de sus características técnicas y su
ubicación, y segmentar los pozos en grupos con comportamiento productivo similar.

## Análisis exploratorio (pre-entrega 2)

1. **Limpieza.** Sacamos las columnas administrativas que no describen al pozo ni a su producción. Revisamos los
   faltantes uno por uno, porque no todos son iguales: el subtipo de recurso (shale o tight) falta porque no aplica
   a los pozos convencionales; la clasificación falta según qué provincia carga el dato; y el 12 % de las
   profundidades aparece como 0 m, que en realidad es un dato que no se cargó. También eliminamos unas 110 filas
   con errores imposibles, como producción negativa.
2. **Recorte.** Nos quedamos sólo con los pozos petroleros en extracción efectiva: 263.937 filas de 24.388 pozos.
   Los pozos parados, abandonados o de inyección declaran 0 m³ casi siempre y no aportan a la pregunta.
3. **Hallazgo principal.** Los pozos no convencionales son el 8,4 % de las declaraciones, pero aportan el 62,3 %
   del petróleo: la mediana de un pozo no convencional es 665 m³/mes, contra 39 m³/mes de uno convencional. Por
   eso la cuenca Neuquina, con menos pozos que Golfo San Jorge, produjo el 73 % del petróleo.
4. **Valores atípicos.** El rango intercuartílico marca como atípicos, en su mayoría, a pozos no convencionales.
   No son errores, son los que más producen, así que no se eliminaron.
5. **Variables nuevas.** Producción por día efectivo (`caudal_diario`), fracción de agua en el líquido extraído
   (`corte_agua`, alta en pozos maduros) y relación gas-petróleo (`rgp`).
6. **Preparación para modelar.** La producción es muy asimétrica (skew 9,07); Yeo-Johnson la lleva a 0,01. La
   separación en entrenamiento y test se hace por pozo, para que un mismo pozo no quede de los dos lados.

El detalle está en el notebook 01.

## Modelos

- **Supervisado (pre-entrega 3).** Predecimos la producción de un pozo-mes sólo con sus características
  (cuenca, provincia, formación, tipo y subtipo de recurso, sistema de extracción, operadora, profundidad, días
  efectivos y mes). Dejamos afuera el gas y el agua del mismo mes para no filtrar la respuesta. Lo planteamos de
  dos formas: **regresión** sobre `log1p(prod_pet)` y **clasificación** en tres rangos de productividad (terciles
  calculados sólo con entrenamiento, así que las clases quedan balanceadas por construcción). Comparamos un modelo
  de referencia, un modelo lineal, un árbol de decisión y un Random Forest con validación cruzada por pozo
  (`GroupKFold`). El árbol y el Random Forest por defecto sobreajustaban y validaban peor que el modelo lineal.
  Optimizamos el Random Forest con **Optuna** y en test obtuvimos **R² 0,71** (la predicción típica se equivoca por
  un factor de 1,9) y **F1 macro 0,61 con AUC 0,80**; la regresión logística, también en test, queda en F1 macro
  0,60. Los días efectivos y la profundidad son las variables más usadas. Como los días efectivos son del mismo
  mes que la producción, medimos el modelo sin ellos: el R² en validación baja de 0,68 a 0,50 y el F1 macro de
  0,60 a 0,57. El detalle está en el notebook 02.
- **No supervisado (pre-entrega 4).** Agrupamos los pozos según su perfil productivo de 2025 (petróleo y gas por
  día efectivo, corte de agua y profundidad; con logaritmo en los caudales y `StandardScaler`). Con **K-Means**
  elegimos k = 3 por el método del codo, aunque el silhouette es máximo con k = 2 (0,68 contra 0,47), porque el
  tercer grupo tiene lectura operativa. Así quedan tres perfiles: convencional maduro con mucha agua, convencional
  de bajo caudal con poca agua y no convencional de alto caudal (el 8 % de los pozos, que da el 62 % del
  petróleo). **DBSCAN**, con `eps` elegido por el gráfico de distancia al vecino n.º 20, encuentra sólo la
  separación entre convencional y no convencional y marca como ruido un 3,4 % de pozos atípicos. Los grupos
  coinciden con el tipo de recurso y la cuenca, que el modelo no vio. El detalle está en el notebook 03.

## Foco Neuquén: Vaca Muerta e YPF

Somos de Neuquén y hacemos el curso en la Fundación YPF, así que miramos de cerca lo que más pesó en todo el
proyecto. **Vaca Muerta** (formación `VMUT`) tiene el 8,8 % de los pozos petroleros pero dio el **62 % del petróleo
de 2025**; en el año creció un 35 % mientras el resto del país cayó un 9 %, y pasó del 58 % al 67 % del total. El
crecimiento viene de **más pozos** (de 1.661 a 2.008 en producción) y no de pozos más productivos: el pozo típico
rindió unos 25 m³ por día efectivo todo el año. **YPF** produce el 49 % del petróleo del país y opera el 59 % del de
Vaca Muerta, de donde ya sale el 75 % de su petróleo. Su producción convencional cayó a la mitad, pero no por
declinación: siguiendo cada pozo, 5.636 de sus 9.735 pozos convencionales pasaron a otras operadoras durante 2025
(4.324 en diciembre, en Santa Cruz), en línea con el Plan Andes. Sólo se analizan pozos petroleros: el gas de Vaca
Muerta queda afuera. El detalle está en el notebook 04.

## Estructura

```text
├── data/
│   ├── produccion_pozos_2025.csv.gz          # dataset original, comprimido
│   └── processed/
│       └── pozos_petroleros_2025.csv.gz      # salida del notebook 01
├── notebooks/
│   ├── 01_analisis_exploratorio.ipynb        # pre-entrega 2
│   ├── 02_modelo_supervisado.ipynb           # pre-entrega 3
│   ├── 03_clustering.ipynb                   # pre-entrega 4
│   └── 04_vaca_muerta_ypf.ipynb              # foco Neuquén: Vaca Muerta e YPF
├── requirements.txt
└── README.md
```

## Cómo ejecutarlo

**Google Colab:** en una celda al comienzo del notebook, clonar el repositorio y ubicarse en `notebooks/`:

```python
!git clone <url-del-repositorio>
%cd energia-digital-proyecto-final/notebooks
!pip install optuna  # sólo para el notebook 02
```

**Anaconda o Python local:**

```bash
git clone <url-del-repositorio>
cd energia-digital-proyecto-final
pip install -r requirements.txt
jupyter notebook notebooks/01_analisis_exploratorio.ipynb
```

Los notebooks leen los datos con rutas relativas (`../data/...`), así que deben ejecutarse desde la carpeta
`notebooks/`. El notebook 02 tarda unos 8 minutos completo, la mayor parte en las dos búsquedas con Optuna; el 03, menos de
un minuto.
