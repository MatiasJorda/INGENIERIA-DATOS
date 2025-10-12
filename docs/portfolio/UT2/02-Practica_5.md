---
title: "UT2 - Actividad 5: Missing Data Detective"
date: 20/08/2025
---

# UT2 - Actividad 5: Missing Data Detective

## Contexto

En esta actividad se trabajó con un dataset relacionado con propiedades residenciales, enfocándonos en identificar, clasificar y tratar datos faltantes (missing data), así como en la detección y análisis de valores atípicos (outliers). Se aplicaron herramientas estadísticas y visuales con el objetivo de mejorar la calidad del dataset antes de su uso para modelado o análisis más avanzados.

## Objetivos

- Detectar datos faltantes y clasificarlos en MCAR, MAR o MNAR.
- Identificar outliers mediante IQR y Z-Score.
- Implementar estrategias de imputación (simple e inteligente).
- Crear visualizaciones para comprender patrones de calidad de datos.
- Reflexionar sobre decisiones éticas en la limpieza de datos.

## Actividades (con tiempos estimados)

| Actividad                          | Tiempo | Resultado esperado                                 |
|-----------------------------------|:------:|----------------------------------------------------|
| Exploración inicial del dataset   |  30m   | Diagnóstico de calidad de datos                    |
| Visualización de missing data     |  45m   | Detección de columnas y filas con datos faltantes |
| Clasificación MCAR/MAR/MNAR       |  30m   | Etiquetado según patrones                         |
| Análisis de outliers (IQR / Z-Score) |  1h   | Identificación de valores atípicos                |
| Imputación (estrategias simples + inteligentes) |  1h | Dataset limpio listo para uso posterior       |
| Documentación y reflexión final   |  30m   | Conclusiones y lecciones aprendidas               |

## Desarrollo

Se realizó un proceso sistemático que incluyó:

- Exploración inicial con `info()`, `describe()` y `dtypes`.
- Evaluación de valores nulos por columna y fila.
- Análisis de memoria por columna.
- Detección de filas duplicadas y columnas redundantes.
- Clasificación de missing data según su mecanismo.
- Detección de outliers usando IQR y Z-Score.
- Visualización de anomalías con boxplots y scatter plots.
- Imputación basada en media, mediana, moda y lógica de negocio.
- Imputación inteligente contextual por grupo (`Neighborhood`, `House Style`).
- Consideraciones éticas sobre imputación y eliminación de datos.

## Evidencias

- Notebook de análisis:

  [Abrir en Colab](https://colab.research.google.com/github/MatiasJorda/INGENIERIA-DATOS/blob/main/docs/portfolio/UT2/Notebooks/Practica_5.ipynb) ·

  [Ver en GitHub](https://github.com/MatiasJorda/INGENIERIA-DATOS/blob/main/docs/portfolio/UT2/Notebooks/Practica_5.ipynb) ·

  [Nbviewer (mirror)](https://nbviewer.org/github/MatiasJorda/INGENIERIA-DATOS/blob/main/docs/portfolio/UT2/Notebooks/Practica_5.ipynb)

---

### Visualización de Missing Data

**Metodología utilizada:**

- Se calculó el porcentaje de valores faltantes por columna.
- Se utilizaron gráficos de barras para destacar las 10 columnas con más `NaN`.
- Se mostró un histograma de valores faltantes por fila.
- Se guardaron los gráficos en carpeta `results/visualizaciones`.

**Visualizaciones generadas:**

- `missing_patterns.png` – Barras + histograma de filas incompletas

**Interpretación:**

Se identificaron columnas con más del 20% de datos faltantes como candidatas a imputación o eliminación. El análisis de filas mostró que la mayoría tiene pocos valores faltantes, permitiendo estrategias de imputación en lugar de eliminación masiva.

---

### Clasificación de Tipos de Missing Data

**Análisis por variable:**

- `Year Built`: patrón MAR → relacionado con `Neighborhood` y `House Style`.
- `Garage Area`: MNAR → faltantes significan ausencia real (coinciden con `Garage Cars == 0`).
- `SalePrice`: MNAR → no completamente aleatorio, se debe imputar cuidadosamente.

**Reflexión ética:**

- Eliminar observaciones con `SalePrice` faltante podría sesgar el análisis.
- Imputar `Garage Area` con cero debe hacerse sólo si se valida que no hay garaje.

---

### Detección de Outliers (IQR y Z-Score)

**Métodos aplicados:**

- **IQR:** para distribuciones sesgadas como `Lot Area` y `SalePrice`.
- **Z-Score:** para distribuciones normales o simétricas como `Garage Area`.

**Resultados:**

- Se creó un DataFrame con resumen por variable (count, %, límites).
- Se generó una visualización combinada con:
  - Boxplot
  - Líneas de corte (límites IQR)
  - Scatter de outliers

**Visualizaciones generadas:**

- `outliers_analysis.png` – 4 boxplots con anotaciones de outliers

**Interpretación:**

- `SalePrice` tiene muchos outliers hacia valores altos → distribución con cola derecha.
- `Year Built` presenta valores extremos en años antiguos → requiere validación histórica.
- En variables sesgadas, los outliers pueden representar propiedades legítimas (mansiones).

---

### Estrategias de Imputación

**Estrategias simples:**

- Media: útil en datos normales → se usó en `Garage Area` en algunos casos.
- Mediana: robusta ante outliers → preferida para `SalePrice`, `Year Built`.
- Moda: usada para variables categóricas (`Garage Type`).

**Estrategia avanzada (smart imputation):**

- `Year Built`: mediana jerárquica por `Neighborhood` + `House Style`.
- `Garage Area`: 0 si no hay `Garage Cars`, luego mediana por barrio.
- `Garage Type`: moda global; se maneja categoría "Unknown".
- Se creó un indicador binario (`GarageArea_was_na`) para auditar imputaciones.

**Resultados:**

- Dataset sin valores nulos en variables clave.
- Reducción del sesgo mediante imputación contextual.

---

## Reflexión

Esta actividad permitió profundizar en la detección y tratamiento de problemas comunes de calidad de datos. La clasificación de los tipos de `missing` fue clave para decidir cómo actuar éticamente sobre los datos: no todos los `NaN` se pueden tratar igual.

El análisis de outliers reveló la necesidad de comprender el contexto de cada variable: un outlier puede ser un error... o una mansión. Por eso, la imputación no debe ser automática, sino contextual.

Finalmente, las estrategias inteligentes basadas en agrupamientos permitieron mantener la coherencia interna del dataset sin recurrir a soluciones simplistas como eliminar filas completas o imputar con la media global. Esta práctica refuerza la importancia de un enfoque reflexivo, reproducible y ético en todo proceso de limpieza de datos.

---
