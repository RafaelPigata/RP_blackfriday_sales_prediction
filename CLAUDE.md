# CLAUDE.md

Guía de contexto para trabajar en este repositorio con Claude Code.

## Qué es este proyecto

Trabajos prácticos de la **Diplomatura de Datos (FAMAF 2026)**, cátedra Mentoría, sobre el
dataset sintético **Black Friday** (comportamiento de compra de clientes de la cadena
ficticia *ABC Private Limited*). El objetivo final de la serie de TPs es poder predecir el
monto de compra (`Purchase`) de un cliente para distintos productos.

Este repo es un **fork** de [dyvanoff/blackfriday_sales_prediction](https://github.com/dyvanoff/blackfriday_sales_prediction)
(remoto `upstream`), donde el profesor va subiendo los enunciados y datasets de cada TP a
medida que se publican. El remoto `origin` es el fork propio
(`RafaelPigata/RP_blackfriday_sales_prediction`), donde se entrega el trabajo.

Para traer enunciados/datasets nuevos que suba la cátedra:
```
git fetch upstream
git merge upstream/main
```

## Estado de la entrega

Los archivos que terminan en `_ENTREGAR` son exactamente lo que se entregó y lo que ve el
profesor. El resto de las notebooks son borradores o versiones individuales previas.

- **TP1 — Análisis y visualización de datos**: ✅ entregado. Notebook final:
  [`TP1_BlackFriday_Analisis_ENTREGAR.ipynb`](TP1_BlackFriday_Analisis_ENTREGAR.ipynb).
- **TP2 — Análisis exploratorio y curación de datos**: ✅ entregado. Notebook final:
  [`TP2_BlackFriday_AEC_G1_ENTREGAR.ipynb`](TP2_BlackFriday_AEC_G1_ENTREGAR.ipynb), que es la
  versión de **Sergio** ([`TP2_BlackFriday_AEC_Sergio.ipynb`](TP2_BlackFriday_AEC_Sergio.ipynb))
  sin la celda de exportación a PDF ni una celda auxiliar suelta. Quedan además como
  historial dos borradores propios previos ([`TP2.ipynb`](TP2.ipynb), [`TP2_v2.ipynb`](TP2_v2.ipynb))
  que **no** son la entrega final.
- **TP3 — Análisis supervisado y no supervisado**: ✅ entregado. Notebook final unificada del
  grupo: [`TP3_Unificado_ENTREGAR.ipynb`](TP3_Unificado_ENTREGAR.ipynb). El enlace de Colab
  entregado está en [`Link.txt`](Link.txt).

## TP1 — Análisis y visualización de datos

Enunciado: [`TP1 BlackFriday Sales Prediction - Análisis y visualización de datos.md`](<TP1 BlackFriday Sales Prediction - Análisis y visualización de datos.md>)
(también hay un `.pdf` equivalente).

Dataset de entrada: `datos/bf_dataset.csv` (dataset clásico Black Friday de
Kaggle/Analytics Vidhya, sin componente temporal).

Puntos cubiertos:
1. Familiarización con el dataset: tipos de dato, identificación única de clientes,
   ausencia de información temporal y sus implicancias, nulos (`Product_Category_2/3`) y
   presencia de variable objetivo (`Purchase`).
2. Estadística descriptiva y visualización: resumen numérico, distribuciones,
   comportamiento de `Purchase` (sesgo, outliers), comparaciones por género/edad/ciudad/
   categoría de producto, relación entre variables, segmentación del gasto (bajo/medio/
   alto) y selección preliminar de variables candidatas para un modelo predictivo.

## TP2 — Análisis exploratorio y curación de datos (versión de Sergio)

Enunciado: [`TP2 BlackFriday Sales Prediction - Análisis Exploratorio y Curación de datos.md`](<TP2 BlackFriday Sales Prediction - Análisis Exploratorio y Curación de datos.md>).

Dataset de entrada: `datos/bf_dataset_temporal.csv.gz` (versión sintética **temporal** del
dataset, con fechas de campaña, hora de transacción, indicadores económicos, etc. — más
rica que la de TP1).

Puntos cubiertos:
1. **Auditoría, calidad y trazabilidad**: unidad de observación, tipos, cardinalidades,
   continuidad diaria, duplicados exactos vs. duplicados por clave de negocio; análisis de
   nulos (cantidad/porcentaje/distribución); validaciones de consistencia (fechas, horas,
   `Is_Weekend`, `Days_To_Black_Friday`, dominios categóricos, importes positivos);
   detección de outliers en `Purchase`.
2. **Exploración temporal**: series de ventas/transacciones diarias, ticket promedio,
   estacionalidad semanal/mensual, comparación entre campañas, medias móviles (7 y 28
   días), tendencia, segmentaciones por canal, demografía, ciudad, categoría de producto y
   contexto económico.
3. **Curación, transformación y feature engineering**:
   - tratamiento de nulos e inconsistencias variable por variable;
   - imputación multivariada (`IterativeImputer` + `KNeighborsRegressor`), comparada
     contra un método simple;
   - nuevas variables a nivel transacción (edad ordinal, flags, interacciones, etc.);
   - tabla diaria auxiliar con rezagos (`lags` de 1/7/14/28 días) y medias/desvíos
     móviles;
   - codificación (`OneHotEncoder`) y escalado;
   - reducción de dimensionalidad con PCA (varianza explicada, `pca1` vs `pca2`).
4. **Relaciones y dependencia temporal**: correlación de Pearson y Spearman,
   autocorrelación de `Daily_Sales` (rezagos relevantes incluyendo 365 días), cuidado con
   asociaciones espurias del generador sintético.
5. **Balance de la curación**: registros conservados/eliminados, valores
   imputados/modificados por variable, comparación de distribuciones antes/después.
6. **Dataset curado final**, generado sin sobrescribir el original:
   - [`datos/bf_dataset_temporal_curado.csv.gz`](datos/bf_dataset_temporal_curado.csv.gz)
   - [`datos/diccionario_bf_curado.csv`](datos/diccionario_bf_curado.csv) (diccionario de
     variables originales, modificadas y generadas)

## TP3 — Análisis supervisado y no supervisado

Enunciado: [`TP3 BlackFriday Sales Prediction - Análisis Supervisado y No Supervisado.md`](<TP3 BlackFriday Sales Prediction - Análisis Supervisado y No Supervisado.md>).

La notebook entregada ([`TP3_Unificado_ENTREGAR.ipynb`](TP3_Unificado_ENTREGAR.ipynb)) junta
las partes que hizo cada integrante, sin modificar su contenido, más una comparación final:

1. **Predicción de ventas diarias** (Rafael Pignata y Constanza Bencharski). Parte del
   dataset curado del TP2 (`datos/bf_dataset_temporal_curado.csv.gz`, leído desde el raw de
   GitHub de este fork) agregado a nivel día. Cubre tabla de modelado con calendario, cercanía
   a Black Friday, contexto económico, lags y medias móviles; baseline estacional y SARIMA;
   validación temporal (`TimeSeriesSplit`); Ridge y LightGBM; evaluación sobre 2024; análisis
   de la ventana de Black Friday; interpretación; y predicción recursiva de todo 2024 desde el
   31/12/2023. Versión individual previa: [`TP3_Parte1_Prediccion_Ventas.ipynb`](TP3_Parte1_Prediccion_Ventas.ipynb),
   con su plan y bitácora en [`TP3_plan_punto1.md`](TP3_plan_punto1.md) y
   [`TP3_avance_punto1.md`](TP3_avance_punto1.md).
2. **Segmentación de clientes** (Sergio Santamarina), sobre `datos/bf_dataset.csv` del TP1:
   K-Means y Ward por comportamiento de compra, caracterización sociodemográfica y
   re-segmentación agregando la demografía (one-hot ingenuo, reponderado y vía PCA).
3. **Segmentación de productos** (Noelia Ramos), mismo dataset y mismos métodos, a nivel
   `Product_ID`, sin y con el perfil de los compradores.
4. **Comparación de las segmentaciones** (síntesis conjunta): las ocho particiones, ARI entre
   métodos y entre representaciones, e interpretabilidad.

Las Partes 2 y 3 escriben exports intermedios (`clusters_clientes_TP3.csv`,
`clusters_productos_TP3.csv`) en `salidas/`, que la Parte 4 vuelve a leer.

## Estructura del repo

```
README.md                                  Consigna general / diccionario de datos original
CLAUDE.md                                  Este archivo
TP1_BlackFriday_Analisis_ENTREGAR.ipynb    Notebook entregada del TP1
TP2_BlackFriday_AEC_G1_ENTREGAR.ipynb      Notebook entregada del TP2
TP2_BlackFriday_AEC_Sergio.ipynb           Versión de Sergio del TP2 (base de la entregada)
TP2.ipynb, TP2_v2.ipynb                    Borradores previos del TP2 (no entregados)
TP3_Unificado_ENTREGAR.ipynb               Notebook entregada del TP3 (4 partes)
TP3_Parte1_Prediccion_Ventas.ipynb         Versión individual previa de la Parte 1 del TP3
TP3_plan_punto1.md, TP3_avance_punto1.md   Plan y bitácora de la Parte 1 del TP3
Link.txt                                   Enlace de Colab entregado
TP1 ... .md / .pdf                         Enunciado del TP1
TP2 ... .md                                Enunciado del TP2
TP3 ... .md                                Enunciado del TP3
datos/
  bf_dataset.csv                           Dataset TP1 (clásico, sin fechas)
  bf_dataset_temporal.csv.gz               Dataset TP2/TP3 (sintético temporal, comprimido)
  bf_dataset_temporal_curado.csv.gz        Dataset curado, salida del TP2
  diccionario_bf_curado.csv                Diccionario del dataset curado
  DICCIONARIO_DATOS.md                     Diccionario del dataset temporal original
```

## Notas de repositorio

- **No versionar** `bf_dataset_temporal.csv` (versión sin comprimir, ~256MB, supera el
  límite de GitHub): está en `.gitignore` y se regenera descomprimiendo
  `datos/bf_dataset_temporal.csv.gz`.
- Tampoco versionar la versión descomprimida del dataset curado
  (`datos/bf_dataset_temporal_curado.csv/`, ~327MB) ni la carpeta `salidas/` que generan las
  Partes 2 y 3 del TP3: ambas están en `.gitignore`.
- `datos/bf_dataset_temporal_curado.csv.gz` pesa ~69MB; GitHub lo acepta pero avisa que
  supera los 50MB recomendados. Si el repo sigue creciendo con más datasets, evaluar
  migrar a Git LFS.
- La carpeta `.claude/` (memoria/config local de Claude Code) está en `.gitignore` y no se
  versiona.
