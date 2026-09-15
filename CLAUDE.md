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

- **TP1 — Análisis y visualización de datos**: ✅ entregado. Notebook final:
  [`TP1_BlackFriday_Analisis.ipynb`](TP1_BlackFriday_Analisis.ipynb).
- **TP2 — Análisis exploratorio y curación de datos**: ✅ entregado, usando la versión de
  **Sergio** como notebook final: [`TP2_BlackFriday_AEC_Sergio.ipynb`](TP2_BlackFriday_AEC_Sergio.ipynb).
  Quedan además como referencia/historial dos borradores propios previos
  ([`TP2.ipynb`](TP2.ipynb), [`TP2_v2.ipynb`](TP2_v2.ipynb)) que **no** son la entrega final.
- **TP3 — Análisis supervisado y no supervisado**: 🔜 enunciado recién incorporado
  ([`TP3 BlackFriday Sales Prediction - Análisis Supervisado y No Supervisado.md`](<TP3 BlackFriday Sales Prediction - Análisis Supervisado y No Supervisado.md>)),
  todavía sin empezar a resolver.

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

## TP3 — Análisis supervisado y no supervisado (pendiente)

Enunciado recién agregado por la cátedra:
[`TP3 BlackFriday Sales Prediction - Análisis Supervisado y No Supervisado.md`](<TP3 BlackFriday Sales Prediction - Análisis Supervisado y No Supervisado.md>).
Todavía no se empezó a resolver — es el próximo trabajo a encarar, presumiblemente
construyendo modelos (supervisados para predecir `Purchase` y no supervisados para
segmentación) sobre el dataset curado del TP2.

## Estructura del repo

```
README.md                                  Consigna general / diccionario de datos original
CLAUDE.md                                  Este archivo
TP1_BlackFriday_Analisis.ipynb             Notebook final del TP1
TP2_BlackFriday_AEC_Sergio.ipynb           Notebook final del TP2 (entregado)
TP2.ipynb, TP2_v2.ipynb                    Borradores previos del TP2 (no entregados)
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
- `datos/bf_dataset_temporal_curado.csv.gz` pesa ~69MB; GitHub lo acepta pero avisa que
  supera los 50MB recomendados. Si el repo sigue creciendo con más datasets, evaluar
  migrar a Git LFS.
- La carpeta `.claude/` (memoria/config local de Claude Code) está en `.gitignore` y no se
  versiona.
