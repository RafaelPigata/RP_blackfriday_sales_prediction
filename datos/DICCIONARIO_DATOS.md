# Diccionario de datos

## Dataset: Black Friday Sales Prediction `bf_dataset_temporal.csv.gz`

El dataset contiene información sintética de transacciones comerciales realizadas entre el **1 de enero de 2022 y el 31 de diciembre de 2024**.

La unidad de observación corresponde a una **transacción individual** asociada a un cliente, un producto y una fecha determinada.

---

## Variables

| Variable | Descripción |
|---|---|
| `Transaction_ID` | Identificador de la transacción |
| `User_ID` | Identificador del usuario |
| `Product_ID` | Identificador del producto |
| `Purchase_Date` | Fecha en la que se realizó la transacción |
| `Transaction_Hour` | Hora en la que se realizó la transacción, expresada entre 0 y 23 |
| `Campaign_Year` | Año correspondiente a la campaña comercial |
| `Days_To_Black_Friday` | Distancia en días entre la fecha de la transacción y el Black Friday correspondiente a ese año. Valores negativos indican días anteriores al evento, `0` corresponde al Black Friday y valores positivos indican días posteriores |
| `Is_Weekend` | Indicador de fin de semana. `1` corresponde a sábado o domingo y `0` al resto de los días |
| `Sales_Channel` | Canal de venta (`Online` o `Store`) |
| `CPI_Index` | Índice de precios al consumidor |
| `Monthly_Inflation_Rate` | Tasa de inflación mensual |
| `Consumer_Confidence_Index` | Índice de confianza del consumidor |
| `Retail_Activity_Index` | Índice de actividad del comercio minorista |
| `Retail_Activity_MoM` | Variación de la actividad del comercio minorista respecto del mes anterior |
| `Category_Activity_Index` | Índice de actividad comercial de la categoría del producto |
| `Category_Activity_MoM` | Variación de la actividad comercial de la categoría respecto del mes anterior |
| `Gender` | Género del usuario (`F` o `M`) |
| `Age` | Edad (en rangos) |
| `Occupation` | Ocupación (enmascarada) |
| `City_Category` | Categoría de la ciudad (`A`, `B`, `C`) |
| `Stay_In_Current_City_Years` | Años de residencia en la ciudad actual. `4+` representa cuatro años o más |
| `Marital_Status` | Estado civil |
| `Product_Category_1` | Categoría principal del producto (enmascarada) |
| `Product_Category_2` | Categoría secundaria del producto (enmascarada) |
| `Product_Category_3` | Categoría terciaria del producto (enmascarada) |
| `Transaction_Satisfaction_Score` | Puntaje de satisfacción asociado a la transacción, en una escala de 0 a 10 |
| `Purchase` | Monto de compra |