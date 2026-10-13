# Plan de acción — TP3, Punto 1: Problema supervisado (predicción de ventas diarias)

> Documento de planificación. No incluye código ni resultados: solo qué se haría, por qué
> y cómo, en base al dataset curado en el TP2
> (`datos/bf_dataset_temporal_curado.csv.gz`).

## 0. Datos de partida (chequeado sobre el dataset curado)

- 1.650.204 transacciones, exactamente 550.068 por año en **2022, 2023 y 2024**
  (rango `2022-01-01` a `2024-12-31`, tres años calendario completos).
- Granularidad: transaccional (`Transaction_ID`). Hay que **agregar a nivel día**.
- El diccionario (`datos/diccionario_bf_curado.csv`) ya marca variables construidas en el
  TP2 con **riesgo de leakage** si se usan mal:
  - `Producto_Precio_Medio`, `Producto_N_Ventas`, `Cliente_Gasto_Medio`,
    `Cliente_N_Compras`, `Ratio_Precio_vs_Cliente` → calculadas sobre el período completo,
    "recalcular por fold" según la nota del propio diccionario.
  - `Satisfaccion` (y `Sat_Fue_Imputada`) → posterior a la compra, leakage directo si se
    usa para predecir `Purchase`.
  - Estas son variables a nivel *transacción/cliente/producto*; para el problema de este
    punto (ventas **diarias**) en principio no entran directo a la tabla de modelado, pero
    hay que tenerlo presente si se decide agregarlas como insumo de alguna variable diaria
    (ej. "gasto medio por cliente promedio del día").

## a. Construcción de la tabla de modelado (unidad = día)

**Qué haría:**

1. **Agregar el dataset transaccional a nivel `Purchase_Date`**, generando una tabla con
   ~1096 filas (365+365+366 días, 2024 es bisiesto).
   - Variable objetivo: `Daily_Sales = sum(Purchase)` por fecha.
   - Variable auxiliar: `Transactions = count(Transaction_ID)` por fecha (útil como
     predictora y para construir el ticket promedio).
2. **Variables de calendario / estacionalidad** (determinísticas, sin riesgo de leakage
   porque se conocen de antemano):
   - día de la semana, mes, trimestre, `Is_Weekend` (ya existe a nivel transacción, se
     puede derivar a nivel día porque es constante dentro del día).
   - codificación cíclica (seno/coseno) de día de la semana y mes, para no imponer una
     distancia artificial entre diciembre y enero.
3. **Cercanía a Black Friday**: reusar `Days_To_Black_Friday` / `Dias_Abs_BF` /
   `En_Ventana_BF` del TP2, llevados a nivel día (son constantes por fecha dado el año).
   Esto es clave porque el TP2 mostró que la ventana BF concentra picos de venta.
4. **Comportamiento histórico de las ventas (autorregresivo)**:
   - rezagos de `Daily_Sales`: 1, 7, 14, 28 y **365 días** (año anterior, dado que hay 3
     años completos — el TP2 ya sugiere mirar autocorrelación a 365 días).
   - medias y desvíos móviles de 7 y 28 días, **calculados solo con información pasada**
     (`shift(1)` antes de la ventana, para no filtrar el propio día).
   - análogos para `Transactions` (rezagos/medias móviles), como proxy de actividad.
   - **Importante**: documentar qué filas quedan sin valor por falta de historia (los
     primeros ~28 días de 2022 y, para el rezago de 365, todo 2022) y decidir si se
     descartan del entrenamiento o se imputan con una marca explícita.
5. **Contexto económico**: `CPI_Index`, `Monthly_Inflation_Rate`,
   `Consumer_Confidence_Index`, `Retail_Activity_Index`, `Retail_Activity_MoM` — son
   mensuales, constantes dentro del mes; se agregan por *merge* sobre el mes de cada fecha
   (no hay leakage porque el enunciado autoriza suponer que las estimaciones económicas
   del período a predecir están disponibles).
   - `Category_Activity_Index` / `Category_Activity_MoM` están particionadas por
     `Product_Category_1`: para pasarlas a nivel día habría que agregarlas (ej. promedio
     ponderado por ventas del día) o directamente dejarlas fuera de este primer modelo
     (más simple, menos supuestos) y mencionarlo como decisión explícita.
6. **Actividad comercial / mix del día** (variables descriptivas construidas agregando
   transacciones, siempre que no usen `Purchase` del propio día como insumo circular):
   - proporción de ventas por `Sales_Channel` (Online vs Store).
   - cantidad de clientes únicos y productos únicos que compraron ese día.
   - mix de categorías (`Product_Category_1`) más vendidas ese día.
   - **Precaución de leakage**: cualquier variable construida a partir de `Purchase` del
     mismo día que se quiere predecir (ej. ticket promedio del día) no puede usarse como
     predictora — solo como target derivado o para diagnóstico posterior.

**Por qué:** el problema pide predecir `Daily_Sales` a horizonte de un día, "suponiendo
que la información de días anteriores ya está disponible". Esto define el diseño: todas
las variables autorregresivas deben construirse con `shift` estricto (nada de ventanas
centradas ni "leakeadas" desde el futuro), mientras que calendario y BF pueden calcularse
sin restricción porque son conocidos de antemano.

**Cómo (a alto nivel):**
- `groupby('Purchase_Date').agg(...)` para la tabla base transaccional → diaria.
- `merge` con una tabla mensual de indicadores económicos por `(año, mes)`.
- Construcción de lags/rolling con `shift()` + `rolling()` ordenando estrictamente por
  fecha antes de aplicar.
- Guardar la tabla resultante en un notebook/paso intermedio documentado (no necesariamente
  como archivo aparte, salvo que se decida reutilizarla en el punto `h`).

## b. Modelos de referencia (baseline)

**Qué haría:**
1. **Baseline estacional**: `Daily_Sales(t) = Daily_Sales(t - 365) * ajuste_por_inflación`,
   usando `CPI_Index` (o `Monthly_Inflation_Rate` acumulada) entre el mes del año anterior
   y el mes actual para escalar el nivel de precios. Esto da un piso "ingenuo pero
   informado" que ya incorpora estacionalidad anual y precios.
2. **ARIMA/SARIMA** sobre la serie univariada `Daily_Sales`, con componente estacional
   semanal (`m=7`) — dado que el TP2 mostró estacionalidad semanal fuerte. Evaluar si
   conviene también un `m=365`/Fourier terms dada la estacionalidad anual, o dejarlo para
   los modelos supervisados y usar SARIMA solo con estacionalidad semanal (más tratable
   computacionalmente).

**Por qué:** el enunciado pide explícitamente un baseline estacional y un modelo clásico
de series de tiempo como referencia. Sirven para verificar que los modelos "más
sofisticados" del punto `d` realmente aporten valor.

**Cómo:** `statsmodels` (`SARIMAX`) ajustado solo sobre 2022–2023 (train), pronosticando
paso a paso sobre 2024 (igual esquema de evaluación que los modelos supervisados, ver
punto `c`).

## c. Partición y estrategia de validación

**Qué haría:**
- **Train/selección de hiperparámetros**: 2022 + 2023.
- **Test final (holdout)**: 2024, usado una sola vez al final.
- Dentro de train, usar **validación cruzada temporal** (`TimeSeriesSplit` o walk-forward
  con ventana expansiva), nunca k-fold aleatorio, para no mezclar pasado y futuro dentro
  de la selección de modelos.
- Como el rezago de 365 días solo tiene historia a partir de 2023, la validación temporal
  dentro de train efectivamente empieza a evaluar "en serio" desde 2023 en adelante — dejar
  esto documentado explícitamente.

**Por qué:** es una serie temporal; cualquier esquema de validación que no respete el
orden cronológico (ej. K-Fold estándar) filtraría información del futuro al pasado y daría
una estimación optimista irreal del error.

## d. Modelos supervisados

**Qué haría:** entrenar al menos dos familias distintas, por ejemplo:
1. **Modelo lineal regularizado** (Ridge o ElasticNet) — interpretable, rápido, buen
   punto de comparación con el baseline estacional.
2. **Modelo de ensamble de árboles con boosting** (LightGBM/XGBoost, ya sugeridos en el
   stack del README) — captura no linealidades e interacciones (ej. entre
   `En_Ventana_BF` y día de la semana) sin necesidad de especificarlas a mano.
- Para cada uno: descripción breve, hiperparámetros relevantes (ej. `alpha` en Ridge;
  `n_estimators`, `max_depth`, `learning_rate` en boosting), búsqueda razonable
  (`GridSearchCV`/`RandomizedSearchCV` con el `TimeSeriesSplit` del punto `c`, no CV
  aleatoria), y registro de la configuración ganadora.

**Por qué:** el enunciado pide comparar familias distintas — lineal vs. no lineal es la
comparación más informativa dado que ya hay estacionalidad/calendario explícita como
features (con lo cual un modelo lineal ya tiene bastante poder) y variables con
interacciones esperables (buen terreno para boosting).

## e. Evaluación

**Qué haría:** reportar al menos:
- **RMSE** (ya sugerida en el TP2/README, penaliza errores grandes — relevante porque el
  evento BF genera picos que un buen modelo debería anticipar).
- **MAE** (más robusta a outliers, da una lectura del error "típico" día a día,
  complementaria al RMSE).
- Opcional: **MAPE o sMAPE** para tener una noción de error relativo, útil al comparar
  magnitudes muy distintas entre temporada baja y BF.
- Análisis de residuos (¿autocorrelación residual? ¿sesgo sistemático en ciertos meses o
  días de semana?), gráfico de observado vs. predicho a lo largo de 2024, y un
  desglose del error por mes/trimestre para identificar períodos problemáticos.

**Por qué:** una sola métrica agregada esconde comportamiento diferencial (ej. un modelo
puede tener buen RMSE promedio pero fallar sistemáticamente en BF); por eso se pide mirar
error por período y no solo el número global.

## f. Black Friday

**Qué haría:** aislar explícitamente los días dentro de `En_Ventana_BF == 1` (o un
entorno más acotado alrededor de `Days_To_Black_Friday == 0`) en el set de test 2024,
recalcular las métricas del punto `e` solo sobre esos días, y compararlas contra las
métricas "resto del año". Graficar observado vs. predicho específicamente en esa ventana.

**Por qué:** el evento es el pico más extremo y de mayor interés de negocio; un modelo
puede tener buen desempeño promedio y aun así subestimar sistemáticamente el pico de BF
(underfitting de la cola derecha), que es justamente el escenario más costoso de errar.

## g. Interpretación

**Qué haría:**
- Para el modelo lineal: coeficientes estandarizados.
- Para el modelo de boosting: importancia de variables (`feature_importances_` y,
  si el tiempo lo permite, SHAP) para ver qué variables (lags, BF, calendario, contexto
  económico) más aportan.
- Cruzar estos resultados con las conclusiones del EDA del TP2 (ej. si el TP2 mostró
  fuerte estacionalidad semanal y efecto BF, se esperaría ver esas variables entre las más
  importantes; si no coincide, es un hallazgo a discutir).
- Remarcar explícitamente la diferencia entre **importancia predictiva** (cuánto ayuda una
  variable a predecir dentro de este modelo) y **causalidad** (si esa variable "causa" el
  cambio en ventas) — varias de las variables económicas/de actividad son en realidad
  proxies o co-movimientos del mismo proceso generador sintético, no relaciones causales
  reales, tal como ya advertía el propio TP2.

## h. Predicción desde un único origen (forecast multi-step, no un paso)

**Qué haría:**
1. Reentrenar (o reusar) los modelos usando como límite de información el **31/12/2023**.
2. Generar las 366 fechas de 2024 y construir sus variables **de forma recursiva/
   iterativa**, ya que los rezagos de `Daily_Sales` (1, 7, 14, 28 días) dejan de estar
   disponibles como datos reales a partir de cierto punto:
   - Para lags que caen dentro de 2023 (ej. predecir el 2/1/2024 con lag-1 = 1/1/2024):
     usar el valor real.
   - A medida que avanza 2024, los lags de corto plazo (1, 7, 14, 28 días) empiezan a
     requerir **valores predichos por el propio modelo en pasos anteriores**
     (forecast recursivo) en vez de valores observados.
   - El lag-365 sigue disponible como dato real todo 2024 (apunta a 2023), lo cual lo
     vuelve una variable especialmente valiosa en este escenario.
   - Rolling means/std de 7 y 28 días deben recalcularse usando la mezcla de
     observado+predicho acumulada, actualizándose fecha a fecha.
3. Comparar el error de este esquema (multi-step / recursivo) contra el esquema "un paso
   adelante con datos reales" del punto `a`-`g`, y específicamente cómo crece el error a
   medida que aumenta la distancia al 31/12/2023 (curva de error vs. horizonte).

**Por qué:** este escenario es mucho más realista para un caso de uso de planificación a
largo plazo (presupuesto anual, por ejemplo) donde no se dispone de información real
"del día anterior" sino que hay que proyectar todo el año de una sola vez. El enunciado
pide explícitamente notar que el error se acumula y que la calidad de las variables
autorregresivas se degrada con el horizonte — es esperable que el desempeño empeore
sensiblemente respecto del esquema de un paso, y que la ventana de BF (a ~11 meses de
horizonte) sea de las más difíciles de acertar bajo este esquema.

---

## Riesgos / decisiones a validar antes de programar

- **Qué hacer con las filas sin historia completa** (los primeros ~28 días de 2022, y todo
  2022 para el lag de 365) — ¿se descartan del entrenamiento o se imputan con un valor
  explícito y una bandera, similar a lo hecho con `Sat_Fue_Imputada` en el TP2?
- **Nivel de agregación de `Category_Activity_Index`**: dejarlo fuera del primer modelo
  (más simple) vs. agregarlo ponderado por ventas del día (más fiel, más trabajo).
- **Alcance del SARIMA**: estacionalidad semanal (`m=7`, más simple) vs. incorporar también
  el componente anual (más costoso computacionalmente, a evaluar según tiempo disponible).
- **Cómo tratar 2024 bisiesto (366 días)** al comparar year-over-year con 2022/2023 (365
  días) en el baseline estacional y en los lags de 365 días — definir una convención
  (ej. usar día-de-año, y para el 29/2 tomar el promedio de 28/2–1/3 del año anterior o
  aceptar el desfasaje de un día) y documentarla.
