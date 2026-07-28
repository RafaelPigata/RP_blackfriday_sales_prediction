# Descifrando el carrito: patrones de compra en Black Friday

## Descripción

Una empresa de retail, **ABC Private Limited**, busca comprender el comportamiento de compra de sus clientes (específicamente, el monto de compra) frente a distintos productos de diversas categorías.

Para ello, ha compartido un resumen de compras de distintos clientes correspondiente al último mes, enfocado en productos de alto volumen.

El dataset incluye:
- Datos demográficos de los clientes (edad, género, estado civil, tipo de ciudad, años de residencia en la ciudad actual)
- Detalles de los productos (ID del producto y categorías)
- Monto total de compra del último mes

---

## Objetivo

Construir un modelo que permita **predecir el monto de compra de un cliente para distintos productos**, con el fin de generar ofertas personalizadas según el perfil del cliente y el producto.

---

## Diccionario de datos

| Variable | Descripción |
|----------|-------------|
| User_ID | Identificador del usuario |
| Product_ID | Identificador del producto |
| Gender | Género del usuario |
| Age | Edad (en rangos) |
| Occupation | Ocupación (enmascarada) |
| City_Category | Categoría de la ciudad (A, B, C) |
| Stay_In_Current_City_Years | Años de residencia en la ciudad actual |
| Marital_Status | Estado civil |
| Product_Category_1 | Categoría principal del producto (enmascarada) |
| Product_Category_2 | Categoría secundaria del producto (enmascarada) |
| Product_Category_3 | Categoría terciaria del producto (enmascarada) |
| Purchase | Monto de compra (variable objetivo) |

---

## Métrica

Se recomienda utilizar el **RMSE (Root Mean Squared Error)**.

- Es una métrica muy común para problemas de regresión
- Penaliza más fuertemente los errores grandes en comparación con el MAE (Mean Absolute Error)

---

## Stack sugerido

- Python
- pandas / numpy
- scikit-learn
- LightGBM / XGBoost
- matplotlib / seaborn / plotly
