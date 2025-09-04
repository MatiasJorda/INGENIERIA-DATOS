---
title: "UT1 - Actividad 4: EDA Multi-fuentes y Joins"
date: 27/08/2025
---

# UT1 - Actividad 4: EDA Multi-fuentes y Joins

## Contexto
En esta actividad se trabajó con datos abiertos de **NYC Taxi (enero 2023)**, complementados con información de **zonas geográficas** y un **calendario de eventos especiales**.  
El objetivo fue integrar múltiples fuentes de datos y realizar un análisis exploratorio que permita extraer patrones metropolitanos y métricas de negocio.  
Además, se incluyó un **BONUS con Prefect** para convertir el pipeline en un flujo orquestado.

## Objetivos
- Integrar datos de **múltiples fuentes** (Parquet, CSV, JSON).
- Aplicar técnicas de **joins en pandas** (left join, inner join).
- Generar análisis agregados con `groupby`.
- Evaluar impacto de **eventos especiales** en los viajes.
- Introducir la orquestación de pipelines con **Prefect**.

## Actividades (con tiempos estimados)
| Actividad                          | Tiempo | Resultado esperado                                |
|-----------------------------------|:------:|--------------------------------------------------|
| Carga multi-fuentes                |  45m   | Datasets cargados y normalizados                  |
| Limpieza y optimización            |  30m   | Tipos optimizados, nulos tratados                 |
| Joins (trips + zones + calendar)   |  45m   | Dataset unificado para análisis                   |
| Análisis por borough y eventos     |  45m   | Reportes comparativos (normal vs. días especiales)|
| Bonus Prefect (pipeline simple)    |  30m   | Flow completo con @task y @flow                   |
| Documentación                      |  30m   | Reflexiones y respuestas                         |

## Desarrollo
1. **Carga de datos**:
   - Viajes desde Parquet (~3M registros).
   - Zonas desde CSV.
   - Eventos calendario desde JSON.  
2. **Normalización**:
   - Columnas a minúsculas.
   - Creación de `pickup_date`.
   - Optimización de tipos (`int16`, `int8`).  
3. **Joins**:
   - `trips + zones` → asignación de borough y zona.
   - `trips_with_zones + calendar` → flag `is_special_day`.  
4. **Análisis**:
   - Métricas por borough: viajes, distancia, tarifas, revenue/km, tip rate.
   - Comparación días normales vs. especiales.
   - Identificación de boroughs dominantes y horas pico.
5. **Bonus Prefect**:
   - Se transformaron funciones clave en `@task`.
   - Se creó un `@flow` principal que orquesta carga → join → análisis.
   - Se destacaron ventajas de Prefect: reintentos automáticos, logging, escalabilidad.

## Evidencias
- Notebook de análisis: [Open Notebook](Notebooks/Practica_4.ipynb)

## Reflexión
Este ejercicio permitió comprender el valor de integrar **múltiples fuentes**:  
- Los **joins** enriquecen el dataset y habilitan análisis más completos.  
- Se observó cómo los **días especiales** influyen en distancia y tarifas promedio.  
- El análisis por **boroughs** reveló patrones de mercado (zonas más rentables, viajes más largos).  

El **BONUS con Prefect** mostró la diferencia entre un simple notebook y un **pipeline orquestado y resiliente**, acercando el trabajo a escenarios reales de **Data Engineering y MLOps**.  
Con esto se afianza el puente entre análisis exploratorio (EDA) y la construcción de sistemas de datos reproducibles y escalables.

---
