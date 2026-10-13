# TP3, punto 1: avance y estado actual

> Documento de seguimiento, separado del [`TP3_plan_punto1.md`](TP3_plan_punto1.md) (que
> describe qué se piensa hacer en cada ítem). Este archivo registra qué se entendió, qué se
> decidió y qué falta efectivamente escribir en el Colab, para poder retomar sin repetir
> explicaciones ya dadas.

## Estado del Colab

- **Punto a (tabla de modelado diaria)**: completo y corriendo. La tabla `daily` quedó con
  1096 filas (365 + 365 + 366, 2024 es bisiesto) y 42 columnas, incluyendo:
  - `Daily_Sales`, `Transactions` (agregado desde el dataset transaccional curado).
  - Calendario: `Dow`, `Month`, `Quarter`, `Is_Weekend`, codificación cíclica
    (`Dow_Sin/Cos`, `Month_Sin/Cos`).
  - Cercanía a Black Friday: `Days_To_Black_Friday`, `Dias_Abs_BF`, `En_Ventana_BF`
    (verificadas como constantes por día antes de bajarlas de transacción a día).
  - Contexto económico mensual por merge: `CPI_Index`, `Monthly_Inflation_Rate`,
    `Consumer_Confidence_Index`, `Retail_Activity_Index`.
  - Mix de actividad del día: `Prop_Online`/`Prop_Store` (proporciones de canal),
    `Clientes_Unicos`, `Productos_Unicos`, `Prop_Categoria_Top`, `Categorias_Distintas`.
  - Lags (1/7/14/28/365 días) y medias/desvíos móviles (7/28 días) de `Daily_Sales` y
    `Transactions`, todos calculados con `shift(1)` antes de la ventana para no filtrar el
    propio día.
- **Punto b (baselines)**: código pegado (baseline estacional + SARIMA) y cerrado
  conceptualmente, incluyendo por qué son dos predicciones independientes (regla fija vs.
  modelo ajustado) y por qué el baseline estacional falla justo en la ventana de BF (la
  fecha de BF se mueve entre años, así que comparar "mismo día, un año atrás" no alinea el
  evento). Falta el gráfico comparativo final (real vs. ambos baselines, con zoom en la
  ventana de BF) para dar por cerrado del todo el punto, pero ya está el código armado.
- **Punto c (partición train/validación/test)**: **resuelto y corrido en el Colab.**
  Decisión tomada con Rafael y Coti: `TimeSeriesSplit` (ventana expansiva) para la
  validación cruzada dentro de train+validación (2022-2023), descartando el holdout único
  (no sirve para comparar hiperparámetros de forma confiable) y la ventana móvil de tamaño
  fijo (no hace falta: no hay cambio de régimen ni ruido que justifique olvidar historia
  vieja). `n_splits=5` corrido y verificado: folds 3-5 (2023 puro) sin `NaN` en
  `Daily_Sales_Lag365`; folds 1-2 (tocan 2022) sí tienen `NaN` ahí, tal como se esperaba.
  Queda pendiente, recién para el punto d, decidir qué hacer con esos `NaN` según el
  modelo elegido (boosting los tolera nativo, Ridge/ElasticNet no).
- **Punto d (modelos supervisados)**: decisiones tomadas y celdas corridas en el Colab.
  - Modelos elegidos: **Ridge** (lineal regularizado) y **LightGBM** (boosting), en vez de
    ElasticNet/XGBoost/Random Forest/SVR (se evaluaron como alternativas y se descartaron:
    ver razones en la conversación, resumen en la sección de decisiones más abajo).
  - Flujo de 3 corridas para tratar los `NaN` de `Lag365` de forma controlada:
    1. LightGBM con `NaN` nativo (opción 4, sin tocar nada).
    2. LightGBM con flag (`*_Disponible`) + imputación con la mediana de train (opción 5),
       mismo modelo que 1, para aislar el efecto de tratar el `NaN` vs. dejarlo nativo.
    3. Ridge con flag + imputación (opción 5), porque no tolera `NaN`.
  - `GridSearchCV` con `cv=tscv` (el `TimeSeriesSplit` del punto c) y
    `scoring="neg_root_mean_squared_error"` para las tres corridas.
  - `FEATURES` para `X`: se excluyeron de `daily` las columnas de actividad del mismo día
    (`Transactions`, `Clientes_Unicos`, `Productos_Unicos`, `Prop_Online/Store`,
    `Prop_Categoria_Top`, `Categorias_Distintas`; son descriptivas del punto a, no
    predictoras válidas para un esquema de un paso adelante), `CPI_Index_Lag365`
    (intermedia, solo servía para `Baseline_Estacional`) y `Baseline_Estacional`/
    `Baseline_SARIMA` (son las predicciones de referencia del punto b, no insumos).
  - **Estado real al cortar la sesión: las celdas de entrenamiento/CV de las 3 corridas ya
    se corrieron en el Colab.** Falta revisar los resultados (`best_params_`,
    `best_score_` de cada una) y seguir con el punto e (evaluación con RMSE/MAE sobre el
    test 2024, comparando contra los baselines del punto b).
  - **Validación definitiva: `TimeSeriesSplit(n_splits=6)`.** Se probó también con 5 folds
    y en el informe se muestran ambas corridas para exhibir los problemas de la CV: (1) los
    promedios de CV no son comparables entre distintos `n_splits` (más folds diluyen los dos
    folds con Black Friday, que concentran ~88% del error); (2) el ranking Ridge vs.
    LightGBM NaN se dio vuelta entre 5 y 6 folds (y el `alpha` ganador de Ridge saltó de 0,1
    a 10), así que la CV no separa bien a esos dos; (3) el fold con el BF 2022 entrena sin
    haber visto ningún BF (RMSE 25 a 28M, peor que "igual que ayer"); (4) los folds
    tempranos tienen `NaN` de `Lag365` (en 5 folds, el fold 3 entrenaba con solo 2 filas con
    `Lag365` real, lo que inflaba la brecha entre LightGBM NaN e Imp). Resultados con 6
    folds (RMSE CV): LightGBM NaN 6,89M (`lr=0.15, depth=3, n_est=800`), LightGBM Imp 7,56M
    (`lr=0.25, depth=2, n_est=1200`), Ridge 6,56M (`alpha=10`). Se decidió no seguir
    ajustando hiperparámetros ni cambiar `n_splits` después de ver resultados.
- **Punto e (evaluación sobre test 2024): completo y corrido.** Métricas: RMSE, MAE y sMAPE
  (a justificar en un markdown en la notebook: RMSE castiga los picos de BF, MAE da el error
  típico en las mismas unidades, sMAPE da el error relativo y evita la explosión de MAPE
  con valores chicos). Test (RMSE / MAE / sMAPE): LightGBM NaN 5,94 / 2,07 / 11,14; Ridge
  6,85 / 2,07 / 9,44; LightGBM Imp 7,50 / 2,83 / 15,97; SARIMA 8,63 / 2,24 / 8,66; baseline
  estacional 22,39 / 5,92 / 15,06; Lag7 24,15 / 5,76 / 12,00. El ranking coincide con el de
  la CV de 5 folds, no con el de 6. Hallazgos:
  - Casi todo el error sale de pocos días de nov-dic (10 peores días: 89% del error
    cuadrático en LightGBM NaN, 94% Ridge, 95% SARIMA, 73% LightGBM Imp). Concentración alta
    no es malo: significa que en el resto del año el modelo es muy preciso.
  - Todos subestiman el día pico (217M el 2/12). Los dos LightGBM tienen residuos con rachas
    (autocorrelación alta en lags 1 a 10); Ridge y SARIMA quedan dentro de la banda de ruido.
    Posible sesgo lento por crecimiento 2024 (días equivalentes vendieron 10% a 23% más que
    en 2023), que los modelos entrenados con 2022-2023 no alcanzan a reflejar.
  - **Predicciones negativas (se decidió NO recortarlas a 0 y dejarlas mostradas):**
    LightGBM NaN 5 (del 5 al 9/12), LightGBM Imp 6 (del 10 al 14/12), Ridge 0. Diagnóstico con
    `pred_contrib=True`: la variable `Transactions_Desvio28` aporta -20,7M en 2024 contra
    -8,0M en un día sano de 2023. Causa: en 2024 (24 días seguidos, 3 al 26/12) esa variable
    supera su máximo de entrenamiento (4695 el 29/11/2022; 4802 el 5/12/2024), porque el BF
    2024 fue más grande (17.614 transacciones en el día pico contra ~16.200), y el árbol no
    extrapola: cae en la hoja extrema aprendida de pocos días post-pico con ventas
    desplomadas. Sacar esa variable (o todos los desvíos móviles) empeoró la CV (6,89M a
    7,18M y 7,30M), así que se dejó tal cual y se documenta como hallazgo.
- **Punto f (Black Friday): completo y corrido.** Zonas: ventana BF (`En_Ventana_BF`, 9 días,
  del 26/11 al 4/12/2024, `Days_To_Black_Friday` de -3 a +5), después de la ventana (5 a 31/12,
  27 días) y resto del año (330 días). Se sumó la zona "después" porque ahí se concentran
  los negativos y rachas de LightGBM; mezclada con "resto" contaminaría el desempeño en
  días normales. RMSE relativo (% de la venta media de la zona), ventana / después / resto:
  LightGBM NaN 21,7 / 58,7 / 10,5; LightGBM Imp 26,8 / 75,0 / 15,1; Ridge 31,5 / 27,1 /
  11,9; SARIMA 40,3 / 28,2 / 14,0; baseline estacional 87,2 / 21,4 / 99,0; Lag7 81,8 /
  273,0 / 25,6. Dinámica en la ventana (pico real 216,7M el 2/12, que cae a +3 días de BF):
  LightGBM NaN es el único que ubica el pico en el día correcto y tiene la mayor correlación
  con lo real (0,9), aunque llega al 71% del pico y subestima el total 15%. SARIMA acierta el
  total (+0,2%) pero por cancelación de errores (correlación 0,4, pico un día tarde). Ridge
  pone el pico en el viernes de BF (día 0), 3 días antes: un lineal no puede armar un pico
  corrido a +3. Baseline estacional gana después de la ventana porque no usa lags
  recientes contaminados por el pico, pero es el peor en la ventana y en el resto del año
  (desalineación del BF entre años). Conclusión: LightGBM NaN es el que mejor representa el
  evento, pero después de BF Ridge y SARIMA son más confiables; no hay ganador único.
  Salvedad: el test tiene un solo BF (9 días).
  - **Por qué falla todo después del BF**: cambio de régimen brusco (217M a 72M en un día,
    a 20 a 30M en una semana) mientras los lags y promedios móviles siguen "recordando" el
    pico (el 9/12, `Lag7`=216,7M y `Media7`=65,2M, con ventas reales de 21,2M); además hay
    solo dos BF en el entrenamiento (~54 filas post-pico). Cada modelo se rompe distinto:
    LightGBM por variable fuera de rango, Ridge por la suma lineal de lags altos, SARIMA
    extrapolando la suba. Idea para el informe (no implementada): lag alineado al BF (ventas
    en el mismo día relativo a BF del año anterior) en vez de Lag365 calendario.
- **Punto g (interpretación): completo.** Coeficientes de Ridge (sobre variables ya
  escaladas con `StandardScaler`) e importancia de LightGBM (`gain` y `split` del
  booster, sobre el modelo NaN nativo). Coinciden entre sí y con el TP2: los dos se
  apoyan fuerte en autorregresivas (`Transactions_Lag1`, `Daily_Sales_Lag/Media/Desvio7`)
  y en cercanía a BF (`En_Ventana_BF`, `Days_To_Black_Friday`, `Dias_Abs_BF`); Ridge
  suma `Is_Weekend`/`Dow` (estacionalidad semanal, ya vista en TP2). Diferencias:
  en LightGBM la importancia está brutalmente concentrada en 2 variables
  (`Transactions_Lag1` y `Dias_Abs_BF` juntas superan por mucho al resto), mientras que
  en Ridge el peso está repartido entre ~15 variables (un árbol puede capturar de un
  solo corte una combinación que un modelo lineal tiene que repartir entre varias
  correlacionadas). LightGBM prefiere la distancia *absoluta* a BF (`Dias_Abs_BF`,
  forma de "V"), Ridge usa la distancia con signo más el flag `En_Ventana_BF`, porque
  una suma lineal no puede replicar bien una V con una sola variable numérica.
  Salvedades documentadas: (1) los signos de coeficientes individuales de Ridge no se
  pueden leer aislados por la multicolinealidad entre lags/rolling de la misma serie
  (ej. `Daily_Sales_Lag7` da negativo pese a la intuición); (2) `Retail_Activity_Index`
  aparece con peso relevante en los dos modelos pero es un proxy temporal (se mueve
  con el paso del tiempo junto con las ventas, no hay causalidad), mismo caso ya
  advertido en el TP2 con las variables económicas. SHAP quedó **pendiente como
  opcional** ("si al final sobra tiempo"), no es necesario para cerrar el punto.
- **Punto h (predicción multi-step desde el 31/12/2023): en curso, avanzado.**
  - **Decisión clave, ya tomada y justificada con datos**: el modelo de este punto se
    entrena **sin las columnas de `Transactions_*`** (lags/rolling), porque en el
    esquema recursivo esa serie tampoco se conoce de antemano y no hay modelo para
    pronosticarla. Se descartaron como alternativas (1) congelar esas columnas en su
    último valor real de 2023 y (2) armar un mini-pronóstico aparte solo para
    `Transactions`, por ser más ruidoso o más trabajo del necesario. Se verificó con
    correlación de Pearson que `Daily_Sales` y `Transactions` están casi perfectamente
    correlacionadas (0.992 en nivel; entre 0.987 y 0.998 en cada lag/rolling
    equivalente, ej. Lag1, Lag7, Media7, Desvio28), así que sacar `Transactions_*`
    no debería perder información relevante: `Daily_Sales_*` ya trae casi la misma
    señal.
  - `FEATURES_H` = `FEATURES` sin las columnas de `Transactions_*`. Se reentrenaron
    LightGBM (NaN nativo) y Ridge (flag+imputación) con los **mismos hiperparámetros
    ya elegidos en el punto d** (no se volvió a correr `GridSearchCV`, alcanza con
    reentrenar sobre el nuevo conjunto de columnas).
  - Bucle recursivo día a día implementado y corrido para 2024 completo (366 días),
    con una `historia` de `Daily_Sales` independiente por modelo (cada uno se
    retroalimenta solo con sus propias predicciones, no se mezclan). Las variables
    conocidas de antemano (calendario, BF, contexto económico) se toman directo de
    `test`; las autorregresivas (`Daily_Sales_Lag1/7/14/28/365`, `Media7/28`,
    `Desvio7/28`) se recalculan en cada paso desde la `historia` acumulada.
  - **Resultado global, recursivo vs. un paso** (RMSE / MAE, en millones):
    LightGBM 14.88 / 6.81 (un paso: 5.94 / 2.07, empeora +150% en RMSE); Ridge 8.01 /
    4.79 (un paso: 6.85 / 2.07, empeora solo +17% en RMSE). **Hallazgo central: Ridge
    se degrada mucho menos que LightGBM al pasar a recursivo**, se invierte el
    ranking del punto e (ahí LightGBM ganaba).
  - En el gráfico real vs. recursivo, LightGBM sube antes de tiempo a principios de
    noviembre y, después del pico de BF, se queda pegado en 65-80M durante semanas
    (la realidad ya volvió a 20-30M); Ridge reacciona más despacio pero se recupera
    más rápido después del pico.
  - En la curva de error (móvil de 14 días) contra el horizonte (días desde el
    31/12/2023): las 4 curvas (LightGBM y Ridge, recursivo y un paso) están chatas y
    parejas durante casi todo el año (hasta ~día 320, fines de octubre), y recién se
    disparan todas juntas cerca de Black Friday (día ~330-360). O sea que la
    acumulación de error **no es gradual a lo largo del año**, es un fenómeno
    concentrado en el evento: LightGBM recursivo llega a un pico de ~50M de error
    móvil y **no baja tan rápido después** (se queda arriba de 45M varios días),
    mientras que Ridge recursivo y las curvas de un paso llegan a ~23M y bajan más
    rápido.
  - **Pendiente para cerrar el punto h**: correr el desglose por zona (ventana BF /
    después / resto del año) sobre las predicciones recursivas, para confirmar con
    números si Ridge le gana a LightGBM en las tres zonas o solo después de la
    ventana (la hipótesis a partir del gráfico es que en la subida previa a BF van
    parejos, y la ventaja de Ridge se nota más "después" que "durante").
  - **Explicación de fondo, ya cerrada con Rafael y Coti tras varias vueltas**: hay
    tres fenómenos relacionados pero distintos que no hay que mezclar:
    1. Predicciones "un poco altas" después de BF (día a día y recursivo): las
       variables autorregresivas traen datos reales pero desactualizados (la
       "cicatriz" del pico), y el modelo tiene poca práctica (solo 2 BF en
       entrenamiento) en cómo pesarlas.
    2. Predicciones negativas (solo día a día, LightGBM): `Transactions_Desvio28`
       rompió el máximo de todo el entrenamiento porque el BF 2024 fue más grande
       (17.614 transacciones el día pico vs. ~16.200 en 2022/2023); un árbol no
       extrapola, cae en la rama más parecida que conoce (días post-pico con ventas
       casi nulas) y termina prediciendo por debajo de cero.
    3. Error que se acumula sin corregirse (solo recursivo): el modelo escribe su
       propia "libreta" en vez de leer datos reales, así que un error de un día
       contamina en cadena los días siguientes (efecto teléfono descompuesto),
       cosa que no pasa en el esquema de un paso porque cada mañana llega un dato
       real que corrige cualquier desvío del día anterior.
  - **Ideas a futuro para el informe** (evaluadas conceptualmente, no implementadas a
    propósito, para responder "qué harían distinto en un proyecto real"):
    - Modelar `log(Daily_Sales)` en vez de `Daily_Sales` directo (al deshacer con
      `exp()`, es matemáticamente imposible que dé negativo), o usar un `objective`
      de LightGBM pensado para variables siempre positivas (Poisson/Tweedie), que
      garantiza no-negatividad por construcción en vez de por transformación manual.
      Ninguna de las dos arregla la causa de fondo (el modelo se queda sin
      referencia frente a un valor que rompe el récord de entrenamiento), solo
      evita que el síntoma se vea tan feo.
    - Para atacar la causa de fondo: más años de historia con más Black Fridays
      (con solo 2 ejemplos el modelo prácticamente los memoriza en vez de
      generalizar), o variables relativas en vez de absolutas (ej. desvío de
      transacciones dividido por el promedio del mes, o un percentil histórico, en
      vez de un conteo crudo que va a seguir rompiendo récords a medida que el
      negocio crece con el tiempo, aunque el año sea "normal").
- Punto g y h: falta cerrar la tabla por zona del h (ver arriba) y redactar. Pendiente
  en la notebook: escribir los markdown entre las celdas (Rafael y Coti los arman a
  mano, en pausa por pedido explícito hasta que avisen que quieren retomarlo),
  incluyendo la justificación de métricas del punto e.

## Qué se entendió en el camino (para no reexplicar de cero)

- **Horizonte de predicción de un día vs. multi-step (punto h)**: en el esquema de un día,
  el modelo se entrena una sola vez con 2022-2023 y después, para cada día de 2024, usa el
  dato **real** del día anterior como insumo (lag). No hay encadenamiento: cada predicción
  es independiente y se apoya en historia verdadera, no en predicciones propias. Eso es
  distinto al esquema del punto h, donde el corte de información queda fijo al 31/12/2023 y
  las predicciones de un día se reinyectan como si fueran el dato real del siguiente
  (ahí sí hay encadenamiento y el error se acumula con el horizonte).
- **Lags**: una columna que trae el valor de la propia variable de N días/filas atrás
  (`.shift(N)`). Le da al modelo una noción de "ayer", "la semana pasada", "hace un año",
  que de otra forma no existiría (sin lags, cada fila se trata como independiente del
  resto). Por qué esos lags puntuales (1, 7, 14, 28, 365): cada uno captura un patrón
  distinto (inercia de corto plazo, estacionalidad semanal, estacionalidad anual).
- **Medias/desvíos móviles (rolling)**: suavizan el ruido de un lag puntual promediando
  (o midiendo la dispersión de) los últimos N días. Siempre calculados con `shift(1)`
  antes de `.rolling()`, para que la ventana no incluya el propio día que se quiere
  predecir (si no, sería leakage directo).
- **Baseline estacional**: repetir el valor de `Daily_Sales` de hace 365 días
  (`Daily_Sales_Lag365`), ajustado por la variación de precios entre ambos momentos
  (`CPI_Index` actual vs. `CPI_Index` de hace 365 días, también calculado con
  `.shift(365)`). Es una regla ingenua pero informada por estacionalidad e inflación, sirve
  como piso de comparación para los modelos más sofisticados del punto d.
- **ARIMA/SARIMA**: modelos estadísticos clásicos para series univariadas (solo miran
  `Daily_Sales`, sin usar el resto de las columnas). AR = autoregresivo (combinación lineal
  de valores pasados), I = diferenciación para quitar tendencia y lograr estacionariedad,
  MA = corrección por errores pasados correlacionados. SARIMA agrega un bloque estacional
  `(P,D,Q,m)` que repite esa misma lógica pero a la distancia `m` (se usa `m=7` por la
  estacionalidad semanal ya vista en el TP2; `m=365` queda descartado por costo
  computacional y porque con 3 años de datos hay pocos ciclos anuales completos para
  estimarlo bien, esa estacionalidad la cubren los lags explícitos del punto d).
- **Por qué hacen falta baselines**: sirven de vara de comparación. Si un modelo complejo
  (punto d) no le gana claramente a una regla simple o a un modelo clásico, no está
  aportando valor real pese a ser más caro de entrenar y explicar.

## Decisiones tomadas

- **Mix de categoría de producto**: se agregó `Prop_Categoria_Top` (proporción de
  transacciones del día que corresponde a la categoría más vendida, sea cual sea) y
  `Categorias_Distintas` (cantidad de categorías con al menos una venta ese día), en vez de
  abrir una columna por cada una de las ~20 categorías.
- **Filas sin historia completa** (primeros ~28 días de 2022 para los rolling, y todo 2022
  para `Lag365`): **decisión pospuesta a propósito hasta el punto d.** Se relevaron tres
  opciones (descartar filas, imputar con bandera tipo `Sat_Fue_Imputada` del TP2, o dejar
  el `NaN` nativo si el modelo lo tolera como el boosting) pero no hace falta resolverlo
  todavía porque no afecta ni al punto a en sí ni al punto b (el baseline estacional y el
  SARIMA no requieren esas filas: el baseline solo necesita que `Lag365` exista para los
  días de test/2024, que siempre existe porque 2023 está completo; el SARIMA se ajusta
  sobre la serie cruda de `Daily_Sales`, sin `NaN` artificiales).

## Código pendiente de pegar en el Colab (punto b)

Retomar por acá. Ya está explicado el porqué de cada línea en la conversación, así que las
celdas quedan documentadas brevemente:

```python
# Celda: baseline estacional (ajustado por inflación)
daily["CPI_Index_Lag365"] = daily["CPI_Index"].shift(365)

daily["Baseline_Estacional"] = (
    daily["Daily_Sales_Lag365"] * daily["CPI_Index"] / daily["CPI_Index_Lag365"]
)

es_2024 = daily["Purchase_Date"].dt.year == 2024
daily.loc[es_2024, "Baseline_Estacional"].isna().sum()  # se espera 0
```

```python
# Celda: ajuste del SARIMA sobre train (2022-2023)
from statsmodels.tsa.statespace.sarimax import SARIMAX

train = daily[daily["Purchase_Date"].dt.year.isin([2022, 2023])].set_index("Purchase_Date")["Daily_Sales"]
test = daily[daily["Purchase_Date"].dt.year == 2024].set_index("Purchase_Date")["Daily_Sales"]

# order=(p,d,q) no estacional, seasonal_order=(P,D,Q,m) con m=7 por la estacionalidad
# semanal. Punto de partida, se puede ajustar si el error da mal más adelante.
modelo_sarima = SARIMAX(
    train,
    order=(1, 1, 1),
    seasonal_order=(1, 1, 1, 7),
    enforce_stationarity=False,
    enforce_invertibility=False,
)
resultado_sarima = modelo_sarima.fit(disp=False)
resultado_sarima.summary()
```

```python
# Celda: predicciones un paso adelante sobre 2024
# append() incorpora los valores reales de test sin reentrenar (refit=False);
# get_prediction con dynamic=False predice cada día usando la historia real hasta
# el día anterior, siguiendo el mismo esquema de "un paso adelante" del resto del punto 1.
resultado_extendido = resultado_sarima.append(test, refit=False)

pred_sarima = resultado_extendido.get_prediction(
    start=len(train), end=len(train) + len(test) - 1, dynamic=False
).predicted_mean

daily.loc[es_2024, "Baseline_SARIMA"] = pred_sarima.values
daily.loc[es_2024, ["Purchase_Date", "Daily_Sales", "Baseline_Estacional", "Baseline_SARIMA"]].head()
```

## Carga de datos (Colab)

El repo (`RafaelPigata/RP_blackfriday_sales_prediction`) es **privado**, así que no se
puede leer directo desde GitHub raw. Se evaluaron alternativas (GitHub raw simple, Drive
compartido con `gdown`) pero se optó, a propósito y por ahora, por una solución más burda:
una ruta de Drive hardcodeada por persona, con la del compañero comentada al lado, y cada
quien descomenta la suya antes de correr. No hay `try/except` ni fallback automático. Si
en algún momento el repo pasa a público o el flujo molesta, se puede volver a las
alternativas evaluadas (quedan documentadas en la conversación, no repetidas acá).

## Duda del CPI_Index resuelta

Se cerró con un ejemplo numérico concreto sobre los propios datos: `.shift(365)` sobre
`CPI_Index` (variable mensual "plana" a nivel día) normalmente trae el CPI del mismo mes
del año anterior, porque el valor no cambia dentro del mes. La única distorsión aparece en
el borde de un año bisiesto: el 29/2/2024 (día que no existió en 2023) hace que el lag
"salte" un día, trayendo el 1/3/2023 en vez de un 29/2 inexistente, y ese corrimiento de un
día queda arrastrado el resto de 2024. Como es un desfasaje de un solo día sobre una
variable que casi no cambia mes a mes, se documenta como riesgo conocido (ya estaba
anotado en el plan como pendiente de convención por el bisiesto) pero no bloquea seguir con
el punto b.
