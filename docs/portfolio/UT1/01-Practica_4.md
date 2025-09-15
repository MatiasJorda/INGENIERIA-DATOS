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
- Notebook de análisis: [Open Notebook](https://nbviewer.org/github/MatiasJorda/INGENIERIA-DATOS/blob/main/docs/portfolio/UT1/Notebooks/Practica_4.ipynb)

### Resultados:

En este proyecto trabajamos con datos de viajes en taxi en la ciudad de Nueva York, complementados con otras fuentes de información. El enfoque principal fue realizar un análisis exploratorio de datos (EDA), aplicando operaciones de combinación (`join`) para integrar datasets y extraer información valiosa.

---

### Carga de datos desde distintas fuentes

**Fuentes utilizadas:**

* **Viajes en taxi (formato Parquet)**: Se importaron más de **3 millones de registros** del dataset oficial de enero de 2023, aprovechando la eficiencia del formato `.parquet` frente al `.csv`.
* **Zonas geográficas (CSV)**: Se añadió el archivo oficial de zonas de NYC, que contiene 265 registros con identificadores de ubicación, boroughs, zonas y zonas de servicio.
* **Calendario de eventos (JSON)**: Se incorporó un calendario local con fechas de eventos, transformando las fechas de formato texto a tipo `datetime`.

Se validó la estructura y calidad de los datos iniciales (filas, columnas, tipos y rangos temporales), asegurando una correcta carga de cada fuente.

---

### Normalización y preparación para combinaciones

**Transformaciones aplicadas:**

* **Estandarización de nombres de columnas**: Se uniformaron a minúsculas para evitar errores durante los merges.
* **Extracción de fecha**: Desde `tpep_pickup_datetime` se generó una nueva columna `pickup_date` para facilitar el cruce con el calendario.
* **Chequeo de tipos de datos clave**: Se validaron tipos compatibles entre columnas clave (`pulocationid ↔ locationid`, `pickup_date ↔ calendar.date`).
* **Optimización de memoria**: Se convirtieron tipos numéricos a formatos más eficientes (`int16`, `int8`) y se imputaron valores faltantes donde fue necesario.
* **Limpieza de nulos**:

  * En `trips`: se eliminaron filas con datos críticos ausentes.
  * En `zones`: se imputaron nulos con el valor `"Desconocido"`.

Al final del proceso, los datasets quedaron **limpios, tipados adecuadamente y listos para ser combinados.**

---

### Primer Join: Trips + Zones

**Método aplicado:**

* Se utilizó un `left join` entre los datasets de viajes y zonas, vinculando `pulocationid` con `locationid`.
* Se conservaron todos los registros originales de viajes, añadiendo los datos geográficos correspondientes.

**Verificación de resultado:**

* Total de filas antes y después del merge: **2,995,023** (sin pérdida de registros).
* Nuevas columnas agregadas: `borough`, `zone`, `service_zone`.
* Nulos en `borough`: **0** → Join exitoso.

**Distribución de pickups por borough:**

| Borough   | Viajes aproximados |
| --------- | ------------------ |
| Manhattan | \~2.65M            |
| Queens    | \~285K             |
| Unknown   | \~39K              |
| Brooklyn  | \~15K              |
| Bronx     | \~3.9K             |

---

### Segundo Join: Trips+Zones + Calendar

**Cómo se realizó:**

* Se combinó el dataset anterior con el calendario de eventos utilizando `pickup_date ↔ date` como clave.
* Se utilizó nuevamente un `left join` para conservar todos los viajes, incluso aquellos sin evento asociado.
* Se creó una nueva columna booleana `is_special_day`, marcando como `True` si el día coincidía con un evento, y `False` en caso contrario.

**Validación final:**

* Se mantuvo el total original de viajes.
* Se garantizó la ausencia de nulos en las variables clave para análisis posteriores.

---

### Análisis agregado por Borough

**Procesamiento realizado:**

* Agrupación de viajes por `borough`.
* Cálculo de métricas: número de viajes, distancias (media, mediana, desviación estándar), montos totales, tarifas, propinas y cantidad promedio de pasajeros.
* Generación de nuevas métricas:

  * `revenue_per_km`: ingreso promedio por kilómetro.
  * `tip_rate`: proporción de propinas sobre la tarifa.
  * `market_share`: participación de cada borough en el total de viajes.

**Principales hallazgos:**

* **Manhattan**: concentra el 88.4% de los viajes con recorridos cortos (2.4 km promedio) y una alta tasa de propinas (19.5%).
* **Queens**: destaca por tener los trayectos más largos (12.3 km) y tarifas elevadas.
* **EWR y Desconocido**: muestran los mayores ingresos por kilómetro, probablemente por ser trayectos especiales o de larga distancia.
* **Bronx y Staten Island**: tienen las tasas de propina más bajas (<3%).

---

###  Días normales vs días especiales

**Qué se hizo:**

* Agrupamiento por `borough` y `is_special_day`.
* Comparación de métricas clave: número de viajes, distancia promedio y tarifa media.

**Resultado:**

* En el mes analizado (enero 2023), **no hubo días catalogados como especiales**.
* Todos los viajes correspondieron a días normales.
* **Promedio general en días normales**:

  * Viajes: 2,995,023
  * Distancia: 3.44 millas
  * Tarifa: \$26.97

---

###  Técnicas aplicadas para trabajar con grandes volúmenes de datos

* **Muestreo estratégico**: Se extrajo una muestra aleatoria de 10.000 viajes (0.3%) para facilitar visualizaciones rápidas.
* **Validación de joins**:

  * Coincidencias con zonas: 100%
  * Zonas utilizadas: 255/265 → **96.2% de cobertura**
* **Análisis por hora del día**:

  * Horas pico: **18:00 (210,761 viajes)**, **17:00** y **15:00**.

---

### Análisis de correlaciones numéricas

**Variables analizadas:**

* `trip_distance`
* `fare_amount`
* `tip_amount`
* `total_amount`

**Principales correlaciones:**

| Variables comparadas           | Correlación   |
| ------------------------------ | ------------- |
| `total_amount` ↔ `fare_amount` | **0.98**      |
| `total_amount` ↔ `tip_amount`  | **0.71**      |
| `fare_amount` ↔ `tip_amount`   | **0.59**      |
| `trip_distance` ↔ otras        | < 0.1 (débil) |

**Interpretación:**

* Existe una fuerte relación entre las variables monetarias, como es esperable.
* La distancia tiene poca correlación directa con el monto total, lo que sugiere que influyen recargos adicionales, peajes o el tipo de zona.

---


## Reflexión
Este ejercicio permitió comprender el valor de integrar **múltiples fuentes**:  
- Los **joins** enriquecen el dataset y habilitan análisis más completos.  
- Se observó cómo los **días especiales** influyen en distancia y tarifas promedio.  
- El análisis por **boroughs** reveló patrones de mercado (zonas más rentables, viajes más largos).  

El **BONUS con Prefect** mostró la diferencia entre un simple notebook y un **pipeline orquestado y resiliente**, acercando el trabajo a escenarios reales de **Data Engineering y MLOps**.  
Con esto se afianza el puente entre análisis exploratorio (EDA) y la construcción de sistemas de datos reproducibles y escalables.

---
