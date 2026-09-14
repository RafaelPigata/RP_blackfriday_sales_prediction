# TP3 — Black Friday Sales Prediction  
## Modelos supervisados y no supervisados

## Entregable

Desarrollar los puntos en una notebook. Subirla a un repositorio de GitHub o entregar un enlace a Google Colab.

La notebook debe poder ejecutarse de principio a fin y debe:

- utilizar como punto de partida **el dataset indicado en cada `Ejercicio`**;
- registrar las decisiones de transformación y modelado;
- justificar la unidad de observación utilizada en cada problema;
- dejar sólo resultados y visualizaciones relevantes;
- incluir una conclusión debajo de cada análisis;
- fijar las semillas utilizadas cuando corresponda.

En este trabajo se abordarán dos problemas distintos:

1. un problema **supervisado de predicción de ventas**;
2. un problema **no supervisado de segmentación de clientes y productos**.

---

# 1. Problema supervisado: predicción de ventas

El dataset utilizado para este ejercicio será el trabajado y curado en el TP2 (`bf_dataset_temporal.csv`).

El objetivo de esta sección será construir modelos para predecir las **ventas totales diarias**, definidas como la suma de `Purchase` correspondiente a cada fecha.

Se trabajará con un **horizonte de predicción de un día**. Se considerará un escenario de predicción diaria a un paso hacia adelante: para predecir las ventas de un día podrá utilizarse la información observada correspondiente a días anteriores, suponiendo que ésta ya se encuentra disponible al momento de realizar la predicción.

## a. Construcción de la tabla de modelado

A partir del dataset transaccional curado en el TP2, construir una nueva tabla cuya unidad de observación sea el día.

Documentar las transformaciones realizadas y describir la variable objetivo y las variables predictoras utilizadas.

Construir variables que permitan representar información relevante para el problema. Pueden considerarse, entre otras:

* comportamiento histórico de las ventas;
* calendario y estacionalidad;
* cercanía a Black Friday;
* contexto económico;
* actividad comercial.

Justificar la construcción, inclusión o exclusión de las variables utilizadas.

> **Nota:** a los fines del trabajo práctico, puede suponerse que las variables económicas correspondientes a las fechas a predecir se encuentran disponibles como estimaciones.


---

## b. Modelos de referencia

Construir un **baseline estacional** que utilice información de las ventas del año anterior y contemple la variación del nivel de precios.

Ajustar además un modelo clásico de series temporales utilizando **ARIMA o SARIMA**.

Evaluar ambos métodos y utilizarlos como referencia para la comparación con los modelos supervisados posteriores.

---

## c. Partición y estrategia de validación

Reservar los datos correspondientes a **2024 exclusivamente para la evaluación final**.

Utilizar los datos de **2022 y 2023** para entrenamiento, selección de modelos y ajuste de hiperparámetros.

Definir y justificar una estrategia de validación consistente con la naturaleza temporal del problema.

Una vez seleccionada la configuración de cada modelo, entrenar el modelo final utilizando la información disponible de 2022 y 2023 antes de evaluar su desempeño sobre 2024.

---

## d. Modelos supervisados

Entrenar y comparar al menos **dos modelos supervisados pertenecientes a familias diferentes** (pueden usar la cantidad de familias de modelos que quieran), incorporando las covariables construidas previamente.

Para cada modelo:

* describir brevemente su funcionamiento;
* identificar los hiperparámetros relevantes;
* realizar un ajuste razonable de hiperparámetros;
* registrar la configuración seleccionada.

Comparar los resultados obtenidos con los modelos de referencia.

---

## e. Evaluación

Seleccionar al menos **dos métricas de evaluación** apropiadas para el problema y justificar su elección (pueden agregar la cantidad de métricas que quieran).

Analizar el desempeño de los modelos considerando:

* error global;
* diferencias entre valores observados y predichos;
* períodos de mayor error;
* comportamiento de los residuos cuando corresponda;
* comparación con los modelos de referencia.

Incluir visualizaciones relevantes que permitan comparar las ventas observadas con las predicciones.

---

## f. Black Friday

Analizar específicamente el desempeño de los modelos durante el período cercano a **Black Friday**.

Comparar este comportamiento con el desempeño general y discutir qué modelos representan mejor la dinámica observada durante el evento.

---

## g. Interpretación

Analizar qué variables presentan mayor capacidad predictiva en los modelos utilizados.

Relacionar los resultados con los patrones observados durante el análisis exploratorio del TP2.

Distinguir entre **asociación, importancia predictiva y causalidad** al interpretar los resultados.


---

## h. Predicción desde un único origen

Repetir el ejercicio de predicción considerando ahora un escenario diferente: se dispone únicamente de información observada hasta el **31/12/2023** y se desea pronosticar la evolución de las ventas durante **todo el año 2024**.

Al construir las variables necesarias para cada fecha futura, tener en cuenta qué información estaría efectivamente disponible al **31/12/2023** y cómo deben actualizarse aquellas variables que dependen de valores de ventas anteriores.

Comparar los resultados obtenidos con los del esquema de predicción a un paso hacia adelante utilizado previamente.

Analizar:

* cómo cambia el desempeño de los modelos;
* cómo evoluciona el error a medida que aumenta la distancia respecto del origen de predicción;
* qué dificultades aparecen al utilizar variables construidas a partir del comportamiento histórico de las ventas;
* cómo se representa el período cercano a Black Friday bajo este escenario.

Discutir las diferencias entre ambos esquemas de predicción y en qué situaciones podría resultar más apropiado utilizar cada uno.

---

# 2. Problema no supervisado: segmentación

El dataset utilizado para este ejercicio será el trabajado en el TP1 (`bf_dataset.csv`).

En esta sección se estudiará la estructura de los datos desde dos unidades de análisis diferentes: **clientes** y **productos**.

Se realizarán cuatro análisis de clustering:

1. clientes a partir de su comportamiento de compra;
2. clientes incorporando además información sociodemográfica;
3. productos a partir de su comportamiento de venta;
4. productos incorporando además información sobre el perfil de sus compradores.

En los cuatro análisis deberán utilizarse los **mismos métodos de clustering**, de manera que sea posible comparar cómo cambia cada método al modificar las variables utilizadas para representar las observaciones.

---

## a. Segmentación de clientes según comportamiento de compra

Construir una nueva tabla cuya unidad de observación sea el **cliente** (`User_ID`).

Cada fila deberá resumir su comportamiento de compra. Construir y justificar variables que permitan representar aspectos relevantes tales como frecuencia de compra, gasto, características del ticket, diversidad de productos o categorías y otras características que consideren pertinentes.

En esta primera segmentación **no utilizar** las variables:

* `Gender`;
* `Age`;
* `Occupation`;
* `City_Category`;
* `Stay_In_Current_City_Years`;
* `Marital_Status`.

Analizar las variables seleccionadas y realizar las transformaciones necesarias antes de aplicar los métodos de clustering.

Aplicar al menos **dos métodos de clustering pertenecientes a enfoques diferentes** (pueden usar más). Para cada uno:

* seleccionar y justificar los hiperparámetros utilizados;
* obtener la asignación de cada cliente mediante `fit_predict`;
* evaluar la calidad de la solución mediante métricas y criterios gráficos apropiados;
* analizar el tamaño y las características de los clusters obtenidos.

Utilizar una técnica de reducción de dimensionalidad o proyección para representar los clientes en **dos dimensiones** y graficar los clusters encontrados.

La visualización deberá utilizarse como complemento del análisis y no como única evidencia de la existencia de los grupos.

---

## b. Caracterización sociodemográfica de los clusters de clientes

Utilizando los clusters obtenidos en el inciso anterior, analizar las características sociodemográficas de cada grupo a partir de:

* `Gender`;
* `Age`;
* `Occupation`;
* `City_Category`;
* `Stay_In_Current_City_Years`;
* `Marital_Status`.

Estas variables no participaron en la construcción de los clusters del inciso anterior.

Analizar si los grupos encontrados a partir del comportamiento de compra presentan diferencias sociodemográficas relevantes e interpretar los perfiles obtenidos.

---

## c. Segmentación de clientes incorporando información sociodemográfica

Construir una segunda representación de los clientes incorporando ahora, además de las variables de comportamiento utilizadas previamente, las variables sociodemográficas.

Analizar qué tratamiento requieren estas variables antes de incorporarlas al clustering y justificar las decisiones tomadas.

Aplicar **los mismos métodos de clustering utilizados en el inciso a**.

Para cada método:

* realizar nuevamente el ajuste y obtener los clusters mediante `fit_predict`;
* seleccionar y justificar sus hiperparámetros;
* evaluar la calidad de la solución obtenida;
* representar los clusters en dos dimensiones;
* interpretar las características de cada grupo.

Comparar los resultados con los obtenidos en el inciso a.

Analizar cuánto cambia la segmentación al incorporar información sociodemográfica y discutir qué variables parecen tener mayor influencia sobre la estructura encontrada.

---

# 3. Segmentación de productos

## a. Segmentación de productos según comportamiento de venta

Construir una nueva tabla cuya unidad de observación sea el **producto** (`Product_ID`).

Cada fila deberá resumir su comportamiento dentro del conjunto de transacciones.

Construir y justificar variables que permitan caracterizar aspectos relevantes del producto, tales como nivel de ventas, cantidad de compradores, características del monto de compra y otras medidas que consideren pertinentes.

En esta primera segmentación no utilizar información sociodemográfica de los clientes que compraron cada producto.

Analizar las variables construidas y realizar las transformaciones necesarias.

Aplicar **los mismos métodos de clustering utilizados en la segmentación de clientes**.

Para cada método:

* obtener la asignación de cada producto mediante `fit_predict`;
* seleccionar y justificar los hiperparámetros;
* evaluar la calidad de los clusters;
* analizar el tamaño y las características de cada grupo.

Utilizar una técnica de reducción de dimensionalidad o proyección para representar los productos en **dos dimensiones** y visualizar los clusters obtenidos.

Interpretar cada cluster a partir de las variables originales utilizadas para representar los productos.

---

## b. Caracterización de los clusters de productos

Caracterizar los clusters obtenidos considerando información que no haya participado inicialmente en su construcción.

Analizar, entre otros aspectos:

* las categorías de producto;
* las características sociodemográficas de los clientes que compran los productos pertenecientes a cada cluster.

Evaluar si los grupos encontrados a partir del comportamiento de venta presentan diferencias en términos del tipo de producto o del perfil de sus compradores.

---

## c. Segmentación de productos incorporando el perfil de sus compradores

Construir una segunda representación de cada producto incorporando, además de las variables de comportamiento comercial utilizadas previamente, información que resuma las características de los clientes que lo compran.

Definir cómo representar a nivel producto variables tales como:

* `Gender`;
* `Age`;
* `Occupation`;
* `City_Category`;
* `Stay_In_Current_City_Years`;
* `Marital_Status`.

Justificar las transformaciones y formas de agregación utilizadas.

Aplicar nuevamente **los mismos métodos de clustering utilizados en los análisis anteriores**.

Para cada método:

* realizar el ajuste y obtener los clusters mediante `fit_predict`;
* seleccionar y justificar sus hiperparámetros;
* evaluar la calidad de la solución;
* representar los clusters en dos dimensiones;
* interpretar las características de cada grupo.

Comparar esta segmentación con la obtenida utilizando únicamente el comportamiento de venta.

Discutir cómo cambia la estructura de los grupos al incorporar información sobre el perfil de los compradores.

---

# 4. Comparación de las segmentaciones

Comparar los resultados obtenidos en los cuatro análisis.

Discutir:

* las diferencias entre los métodos de clustering utilizados;
* el efecto de incorporar variables sociodemográficas en la segmentación de clientes;
* el efecto de incorporar el perfil de los compradores en la segmentación de productos;
* si los distintos métodos identifican estructuras similares o diferentes;
* si las diferencias observadas se deben principalmente al método utilizado o a la representación de las observaciones;
* qué segmentaciones resultan más interpretables.

La comparación entre representaciones deberá realizarse utilizando **el mismo método de clustering**. Por ejemplo, la solución obtenida mediante un determinado método para clientes sin variables sociodemográficas deberá compararse con la solución del mismo método luego de incorporar dichas variables.
