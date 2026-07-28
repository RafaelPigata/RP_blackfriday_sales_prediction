# Trabajo Práctico N.º 1

## Análisis y visualización de datos: Black Friday

## Input

1. Dataset Black Friday (Kaggle / Analytics Vidhya).

## Entregables

1. Se puede desarrollar cada punto en la misma notebook en la que se escriba el código.
2. Se debe subir el entregable a un repositorio de GitHub o enviar el enlace a un documento de Google Colab.
3. Si bien pueden realizarse diversos análisis y visualizaciones durante el desarrollo, en el entregable debe dejarse únicamente aquello que sea relevante.
4. Luego de cada análisis es importante expresar una conclusión sobre lo observado.

## 1. Familiarización con el dataset

### a. Datos disponibles

Importar y analizar el dataset para determinar con qué datos contamos, tanto en cantidad como en calidad.

- Indagar el tipo de cada dato (categórico, numérico).
- Analizar la naturaleza de las variables (demográficas, geográficas, producto, comportamiento de compra).

### b. Identificación de clientes

- ¿Tenemos forma de identificar de manera única a cada cliente?
- ¿Cuántos clientes únicos tenemos en el dataset?

### c. Información temporal

- ¿Tenemos información temporal en el dataset?
- En caso de contar o no con fechas:
  - ¿qué implicancias tiene para el análisis?;
  - ¿qué limitaciones introduce?

### d. Datos nulos y suficiencia de los datos

- ¿Tenemos datos nulos?
- ¿En qué variables aparecen?
- ¿Qué interpretación de negocio podrían tener?
- ¿Tenemos datos suficientes para pensar en realizar un modelo predictivo?

### e. Variable objetivo

- ¿Tenemos la columna objetivo necesaria para un problema de aprendizaje supervisado?
- ¿Qué representa esta variable desde el punto de vista del negocio?

## 2. Análisis y visualización de los datos

### a. Resumen estadístico

Para los datos numéricos, realizar una breve descripción mediante estadísticas de resumen:

- mínimo;
- máximo;
- media;
- mediana;
- cuantiles.

### b. Distribución de las variables

Visualizar la distribución de las variables mediante gráficos como histogramas y diagramas de caja (*boxplots*). Identificar:

- valores atípicos (*outliers*);
- asimetrías;
- posibles datos faltantes.

### c. Distribución de la variable objetivo

Analizar la distribución de `Purchase`:

- ¿Está sesgada?
- ¿Existen valores extremos?
- ¿Cómo es el comportamiento general del gasto?

### d. Comparación del comportamiento de compra

Comparar el comportamiento de los clientes en términos de su gasto según diferentes variables.

Responder además:

1. ¿Cuál es la distribución de los clientes por género?
2. ¿Qué grupos etarios presentan mayor gasto?
3. ¿Dónde se encuentra el mayor número de clientes?
4. ¿Cuántos productos únicos tiene el dataset?
5. ¿Cuáles son las categorías de productos más frecuentes?
6. ¿Qué combinaciones de categorías están asociadas a un mayor gasto?
7. ¿Cuáles son los productos con mayor monto de compra?
8. ¿Cómo se comportan los clientes con valores faltantes en `Product_Category_2` y `Product_Category_3`?
9. ¿Cómo se comporta el gasto en presencia de datos faltantes?

### e. Relación entre variables

En función de los puntos anteriores, analizar la relación entre las distintas variables y la variable objetivo.

### f. Segmentación del gasto

Segmentar la variable objetivo en grupos, por ejemplo: gasto bajo, medio y alto.

- Analizar las diferencias entre los segmentos.
- Caracterizar cada grupo.

### g. Selección preliminar de variables

¿Qué columnas resultan interesantes para incluir en un futuro modelo predictivo?

Considerar especialmente aquellas que presentan diferencias significativas en el comportamiento del gasto.
