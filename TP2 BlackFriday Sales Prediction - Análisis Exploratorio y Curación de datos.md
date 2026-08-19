# Trabajo Práctico N.º 2

## Análisis Exploratorio y Curación de Datos: Black Friday

## Input

1. Dataset sintético temporal de Black Friday (datos/bf_dataset_temporal.csv.gz).

## Entregables

1. Se puede desarrollar cada punto en la misma notebook en la que se escriba el código.
2. Se debe subir el entregable a un repositorio de GitHub o enviar el enlace a un documento de Google Colab.
3. Si bien pueden realizarse diversos análisis y visualizaciones durante el desarrollo, en el entregable debe dejarse únicamente aquello que sea relevante.
4. Luego de cada análisis es importante expresar una conclusión sobre lo observado.

La notebook debe poder ejecutarse de principio a fin y debe registrar: archivo usado,
cantidad de filas y columnas, rango de fechas, semilla y decisiones de limpieza.

## 1. Auditoría, calidad y trazabilidad

### a. Estructura y granularidad

- Identificar la unidad de observación del archivo.
- Verificar tipos, cardinalidades, rango de fechas y continuidad diaria.
- Explicar por qué repetir un `User_ID` o un `Product_ID` no constituye por sí solo un
  duplicado.
- Buscar duplicados exactos y duplicados según una clave de negocio propuesta.

### b. Valores nulos

- Calcular cantidad y porcentaje de nulos por variable.
- Analizar cómo se distribuyen los valores faltantes.
- Evaluar si los nulos parecen representar el mismo tipo de ausencia de información en todas las variables.
- Identificar variables para las cuales sería necesario evaluar una estrategia de imputación.

### c. Valores erróneos y consistencia temporal

Validar, como mínimo:

- `Purchase_Date` válida y perteneciente a `Campaign_Year`;
- `Transaction_Hour` entre 0 y 23;
- correspondencia entre fecha, `Is_Weekend` y `Days_To_Black_Friday`;
- consistencia temporal de los indicadores económicos;
- categorías de edad, ciudad, residencia, estado civil y producto dentro de los
  dominios esperados;
- importes positivos.

### d. Outliers

- Analizar outliers de `Purchase` globalmente y por campaña/categoría.
- Distinguir posibles errores, transacciones de alto valor y picos comerciales legítimos.
- Proponer qué tratamiento sería razonable para los casos detectados, sin modificar todavía el dataset.

## 2. Análisis exploratorio temporal

### a. Evolución y estacionalidad

Construir gráficos de:

- ventas y cantidad de transacciones por día;
- ticket promedio por día y por mes;
- ventas por día de la semana y mes;
- evolución por campaña, usando índices comparables;
- ventas según `Days_To_Black_Friday`, superponiendo los tres años.

Usar agregaciones (`sum`, `count`, `mean`) conscientemente: el promedio de tickets y
la venta total responden preguntas distintas.

### b. Tendencia y medias móviles

- Calcular medias móviles de 7 y 28 días sobre la serie diaria.
- Analizar tendencia, estacionalidad semanal y picos de evento.
- Comparar la serie original con una descomposición o con diferencias temporales.
- Aclarar cómo se tratan los bordes donde una ventana móvil aún no tiene historia.

### c. Segmentación

Comparar monto, frecuencia y ticket promedio por:

- canal de venta;
- grupo etario y género;
- ciudad y categoría de producto;
- campaña y cercanía a Black Friday;
- contexto económico y actividad del rubro.

Justificar las agregaciones y transformaciones utilizadas cuando las variables
comparadas tengan distinta granularidad temporal.

## 3. Curación, transformación y feature engineering

### a. Tratamiento de valores faltantes e inconsistencias

A partir de los problemas detectados durante la auditoría:

- Definir y aplicar una estrategia de tratamiento para cada variable con valores faltantes.
- Justificar las decisiones a partir del significado de la variable, su granularidad y la información disponible en el dataset.
- Evaluar cuándo un valor puede ser reconstruido a partir de otras observaciones y cuándo requiere una técnica de imputación.
- Justificar si corresponde mantener un valor faltante, utilizar una categoría explícita como `No_aplica`, reconstruirlo, imputarlo o eliminar la observación.
- Corregir las inconsistencias consideradas errores y documentar cada decisión adoptada.

### b. Imputación iterativa con KNN

- Seleccionar una variable numérica con valores faltantes para la cual resulte razonable evaluar una imputación multivariada.
- Seleccionar y justificar las variables que aportarán información al proceso de imputación.
- Construir una matriz numérica escalada y aplicar `IterativeImputer` utilizando `KNeighborsRegressor` como estimador sobre una muestra reproducible.
- Comparar el método con al menos una estrategia de imputación más simple.
- Comparar gráficamente la distribución de los valores observados y los valores imputados.
- Informar la cantidad de iteraciones realizadas y si el algoritmo alcanzó su criterio de parada.
- Explicar por qué el escalado resulta relevante en un método basado en vecinos.
- Evaluar ventajas y limitaciones de los métodos probados y justificar cuál se incorpora al dataset curado.

**Importante:** no es necesario utilizar la misma estrategia de imputación para todas las variables con valores faltantes.

### c. Variables de transacción

Evaluar transformaciones como:

- número de categorías informadas y flag de múltiples categorías;
- edad ordinal o punto medio del rango, explicitando supuestos;
- residencia `4+` transformada sin perder su condición de intervalo abierto;
- variables que resuman el comportamiento observado de clientes y productos;
- interacciones entre variables disponibles.

Toda nueva variable debe estar acompañada por una breve justificación de su significado.

### d. Transformaciones temporales

Construir una tabla diaria auxiliar, ordenada cronológicamente, y utilizarla para explorar transformaciones temporales como:

- día de semana, mes, trimestre y fin de semana;
- rezagos de `Daily_Sales` de 1, 7, 14 y 28 días;
- medias y desvíos móviles de 7 y 28 días;
- rezagos de `Transactions`.
- Analizar valores extremos de `Daily_Sales` y `Transactions` en contexto temporal.

Analizar qué observaciones quedan sin valores al construir estas variables, explicar por qué ocurre y discutir en qué tipos de análisis posteriores podrían resultar útiles.

### e. Codificación y escalado

- Aplicar `OneHotEncoder(handle_unknown="ignore")` sobre un conjunto justificado de variables categóricas.
- Analizar ventajas, limitaciones y aplicabilidad de One Hot Encoding, codificación ordinal y estrategias para variables de alta cardinalidad.
- Justificar qué variables requieren escalado y en qué procedimientos resulta necesario.

### f. Reducción de dimensionalidad con PCA

- Seleccionar y justificar las variables que participarán del análisis.
- Estandarizar la matriz procesada y aplicar PCA.
- Informar varianza explicada individual y acumulada.
- Analizar cuántas componentes serían necesarias para conservar una proporción razonable de la variabilidad.
- Graficar `pca1` contra `pca2`, coloreando por una variable descriptiva relevante.
- Analizar qué variables tienen mayor contribución a las primeras componentes.

## 4. Relaciones y dependencia temporal

- Seleccionar y justificar las variables numéricas a incluir en el análisis de
  correlación.
- Construir matrices de correlación utilizando Pearson y Spearman.
- Comparar ambos resultados e interpretar similitudes y diferencias.
- Analizar la autocorrelación de `Daily_Sales`, prestando especial atención a rezagos
  con interpretación temporal relevante, como 1, 7, 14, 28 y 365 días.
- Identificar variables que presenten información potencialmente redundante y discutir
  si esa redundancia responde a su construcción, granularidad o significado.
- Evitar eliminar variables únicamente por presentar una correlación elevada.
- Señalar asociaciones impuestas por el generador sintético y evitar interpretarlas
  como relaciones causales reales.

## 5. Balance de la curación

Una vez finalizadas las decisiones de limpieza, imputación y transformación:

- Informar cantidad y porcentaje de registros conservados y eliminados.
- Informar cantidad de valores reconstruidos, imputados o modificados por variable.
- Comparar la cantidad de valores faltantes antes y después de la curación.
- Comparar antes y después la distribución de variables relevantes, prestando especial atención a aquellas que hayan sido imputadas o modificadas.
- Verificar nuevamente las principales reglas de consistencia utilizadas durante la auditoría.
- Justificar cualquier pérdida de registros o modificación sustancial de la distribución de una variable.
- Resumir las principales decisiones tomadas y sus posibles consecuencias sobre análisis posteriores.

## 6. Generación del dataset curado

Guardar, sin sobrescribir el archivo de entrada:

`datos/bf_dataset_temporal_curado.csv.gz`

El archivo final debe:

- conservar la granularidad transaccional del dataset original;
- contener los registros considerados válidos luego del proceso de curación;
- incorporar las correcciones e imputaciones justificadas;
- conservar las variables originales relevantes y las nuevas variables de nivel transacción que se hayan decidido incorporar;
- estar ordenado cronológicamente.

Acompañar el archivo con un breve diccionario de datos que permita distinguir variables originales, variables modificadas y variables generadas.

Las tablas auxiliares construidas para análisis temporal, PCA u otras exploraciones no deben guardarse como datasets finales en este trabajo.