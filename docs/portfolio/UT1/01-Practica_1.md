---
title: "UT1 - Actividad 1: Carga y EDA del dataset Iris"
date: 04/09/2025
---

# UT1 - Actividad 1: Carga y EDA del dataset Iris

## Contexto
En esta actividad se trabajó con el clásico **dataset de Iris**, utilizando distintas formas de carga: librerías (`seaborn`, `scikit-learn`), Kaggle API, CSV manual y desde URL.  
El objetivo fue practicar diferentes estrategias de acceso a datos, realizar un análisis exploratorio (EDA) básico y documentar los principales hallazgos.

## Objetivos
- Cargar datos desde **múltiples fuentes** (seaborn, sklearn, Kaggle, Colab manual y URL).  
- Aplicar chequeos básicos: `describe`, tipos de datos, valores faltantes.  
- Construir un **diccionario de datos**.  
- Validar rangos y plausibilidad de las variables.  
- Explorar correlaciones y generar visualizaciones clave.  

## Actividades (con tiempos estimados)
| Actividad                | Tiempo | Resultado esperado                              |
|---------------------------|:------:|------------------------------------------------|
| Carga multi-origen        |  30m   | Datasets cargados en DataFrames                |
| Chequeos básicos          |  20m   | Shape, dtypes, valores faltantes, `describe`   |
| Plausibilidad y rangos    |  20m   | Validación min/max y % missing                 |
| Análisis estadístico      |  20m   | Correlaciones, skew/kurtosis                   |
| Visualizaciones           |  30m   | Histogramas, boxplots, heatmap, pairplot       |
| Documentación             |  20m   | Data dictionary e insights                     |

## Desarrollo
1. **Carga de datos**  
   - Se usaron 5 métodos: `seaborn.load_dataset`, `sklearn.datasets.load_iris`, Kaggle API, carga manual de CSV en Colab y URL pública.  
2. **Chequeos básicos**  
   - `df.describe()` y `df.dtypes` para entender la estructura.  
   - Exportación de missingness.  
3. **Plausibilidad**  
   - Se construyó un `range_check.csv` con rangos esperados de variables.  
   - Validación de min/max, %missing y banderas de calidad.  
4. **Análisis estadístico**  
   - Correlaciones.  
   - Skewness y curtosis .  
   - Distribución de especies .  
5. **Visualizaciones**  
   - Histogramas por especie.  
   - Matriz de correlación (`heatmap`).  
   - Pairplot por especie.  
   - Boxplots de variables por especie.  
   - Barplot de missingness.

## Evidencias
- Notebook de análisis: [Open Notebook](Notebooks/Practica_1.ipynb)

## Reflexión
El dataset Iris permitió ejercitar el **flujo completo de un EDA**: desde la carga multi-fuentes hasta el análisis estadístico y visual.  
Se comprobó que las variables de **pétalo** son las más discriminativas entre especies, lo cual explica su uso frecuente en clasificación.  
Además, se reforzó la importancia de validar rangos y valores faltantes, incluso en datasets “limpios”.  
El ejercicio mostró cómo la **documentación clara** (diccionario de datos y reportes) agrega trazabilidad y valor al análisis.  
