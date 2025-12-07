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
- Notebook de análisis:

  [Abrir en Colab](https://colab.research.google.com/github/MatiasJorda/INGENIERIA-DATOS/blob/main/docs/portfolio/UT1/Notebooks/Practica_4.ipynb) •

  [Ver en GitHub](https://github.com/MatiasJorda/INGENIERIA-DATOS/blob/main/docs/portfolio/UT1/Notebooks/Practica_4.ipynb) •

  [Nbviewer (mirror)](https://nbviewer.org/github/MatiasJorda/INGENIERIA-DATOS/blob/main/docs/portfolio/UT1/Notebooks/Practica_4.ipynb) •

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

* **Estandarización de nombres de columnas**: Se uniformaron a minúsculas para evitar errores durante los merges. Esto garantiza consistencia entre datasets que pueden tener diferentes convenciones de nomenclatura.

* **Extracción de fecha**: Desde `tpep_pickup_datetime` se generó una nueva columna `pickup_date` (tipo `date`) para facilitar el cruce con el calendario. El rango de fechas detectado fue desde 2008-12-31 hasta 2023-02-01.

* **Chequeo de tipos de datos clave**: Se validaron tipos compatibles entre columnas clave:
  - `pulocationid` (int64) ↔ `locationid` (int64) → Compatibles
  - `pickup_date` (date) ↔ `calendar.date` (date) → Compatibles

* **Optimización de memoria**: Para un dataset de 3+ millones de registros, se aplicaron optimizaciones:
  - `pulocationid` y `dolocationid`: Convertidos a `int16` (reducción de memoria).
  - `passenger_count`: Convertido a `int8` (valores típicamente entre 0-6).
  - `locationid` en zones: Convertido a `int16`.
  - **Resultado**: Ahorro de memoria del 8.1% (de 682.6 MB a 627.0 MB).

* **Limpieza de nulos**:
  * En `trips`: Se eliminaron filas con datos críticos ausentes (71,743 registros eliminados, principalmente con nulos en `airport_fee`, `congestion_surcharge`, `store_and_fwd_flag`, `ratecodeid`). Se rellenaron nulos en `passenger_count` con el valor típico de 1.
  * En `zones`: Se imputaron nulos con el valor `"Desconocido"` para mantener integridad referencial.

**Validación final:**
- Total de viajes después de limpieza: **2,995,023**
- Viajes sin pickup location: **0**
- Viajes sin dropoff location: **0**
- Viajes sin passenger_count: **0**

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

* Agrupación de viajes por `borough` utilizando `groupby()`.
* Cálculo de métricas estadísticas:
  - **Conteo**: Número total de viajes por borough.
  - **Distancias**: Media, mediana y desviación estándar de `trip_distance`.
  - **Montos**: Media, mediana y desviación estándar de `total_amount`.
  - **Tarifas**: Media de `fare_amount`.
  - **Propinas**: Media y mediana de `tip_amount`.
  - **Pasajeros**: Media de `passenger_count`.

* Generación de nuevas métricas empresariales:

  * `revenue_per_km`: Ingreso promedio por kilómetro (avg_total / avg_distance). ├Ütil para identificar zonas más rentables.
  * `tip_rate`: Proporción de propinas sobre la tarifa base (avg_tip / avg_fare * 100). Indica satisfacción del cliente.
  * `market_share`: Participación de cada borough en el total de viajes (num_trips / total * 100). Muestra concentración de demanda.

**Principales hallazgos:**

* **Manhattan**: 
  - Concentra el **88.4%** de los viajes (2,648,320 viajes).
  - Recorridos cortos: **2.41 km** promedio (mediana: 1.61 km).
  - Alta tasa de propinas: **19.5%**.
  - Revenue por km: **$9.27**.
  - Tarifa promedio: **$22.35**.

* **Queens**: 
  - **9.5%** del market share (285,126 viajes).
  - Trayectos más largos: **12.32 km** promedio (mediana: 11.27 km).
  - Tarifas elevadas: **$67.35** promedio.
  - Revenue por km: **$5.47**.
  - Tasa de propinas: **15.7%**.

* **EWR y Desconocido**: 
  - Muestran los mayores ingresos por kilómetro (**$65.21** y **$46.24** respectivamente).
  - Probablemente trayectos especiales o de larga distancia (aeropuertos, viajes interurbanos).

* **Bronx y Staten Island**: 
  - Tienen las tasas de propina más bajas (<3%).
  - Menor volumen de viajes pero con patrones distintivos.

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

### 7. Perfiles Automatizados (Opcional)

Si se genera un reporte HTML automatizado usando herramientas como `ydata-profiling` o reportes de Prefect, estos pueden ser visualizados de la siguiente manera:

**Ver reporte HTML generado:**

- Reporte HTML pendiente (agrega enlace cuando exista)

**Alternativa: Abrir desde PowerShell/CMD:**

Si el link no funciona, puedes abrir el reporte ejecutando este comando en PowerShell desde la carpeta del proyecto:

```powershell
# Navegar a la carpeta del reporte
cd "docs\portfolio\UT1\Notebooks\results\reportes"

# Abrir el HTML en el navegador predeterminado
Start-Process "nombre_del_reporte.html"
```

O desde cualquier ubicación usando la ruta completa:

```powershell
Start-Process "docs\portfolio\UT1\Notebooks\results\reportes\nombre_del_reporte.html"
```

**Nota:** Esta sección está preparada para cuando se generen reportes HTML automatizados del análisis de NYC Taxi. Los reportes pueden incluir análisis estadísticos completos, visualizaciones interactivas y alertas sobre la calidad de los datos.

---

## Preguntas Finales

### 1. Qué diferencia hay entre un LEFT JOIN y un INNER JOIN?

**LEFT JOIN**: Mantiene todos los registros de la tabla izquierda (la que se mantiene), mientras que de la tabla derecha se agregan los valores correspondientes. Si no hay coincidencia, las columnas de la tabla derecha se llenan con valores nulos.

**INNER JOIN**: Realiza una intersección de las dos tablas, manteniendo solo los registros que tienen coincidencias en ambas tablas. Si un registro de una tabla no tiene correspondencia en la otra, se elimina del resultado.

### 2. ¿Por qué usamos LEFT JOIN en lugar de INNER JOIN para trips+zones?

Porque al hacer LEFT JOIN nos aseguramos de mantener toda la información de los viajes, agregando las zonas correspondientes a los mismos. Si hiciéramos INNER JOIN, perderíamos la información de los trips que no tienen zona asignada, lo cual podría eliminar datos válidos y sesgar el análisis.

### 3. ¿Qué problemas pueden surgir al hacer joins con datos de fechas?

Los principales problemas incluyen:

- **Diferencias en el tipo de dato**: Por ejemplo, una columna puede ser `string` mientras que la otra es `datetime`.
- **Formatos de fecha distintos**: Por ejemplo, `YYYY-MM-DD` vs `DD/MM/YYYY`.
- **Valores nulos o fechas faltantes**: Pueden impedir el join o generar resultados inesperados.
- **Zonas horarias**: Si las fechas tienen información de zona horaria, pueden causar desajustes.

### 4. ¿Cuál es la ventaja de integrar múltiples fuentes de datos?

Nos permite realizar un análisis más completo y contextualizado. Además, se pueden cruzar variables de diferentes bases para descubrir patrones que no serían visibles en un solo dataset; esto enriquece la información y habilita conclusiones más profundas.

### 5. ¿Qué insights de negocio obtuviste del análisis integrado?

- **Manhattan concentra la mayoría de los viajes** (88.4% del market share), lo que indica una alta demanda en esta zona.
- **Los viajes en Queens son más largos y costosos** en promedio, sugiriendo trayectos interurbanos o hacia aeropuertos.
- **El mejor revenue por kilómetro es de EWR**, probablemente por ser trayectos especiales o de larga distancia.
- **Hay diferencias claras en el revenue por kilómetro y en la tasa de propinas** entre boroughs, lo que puede informar estrategias de pricing diferenciadas.
- **Los días especiales pueden tener impacto** en la distancia y tarifa promedio, aunque en este período no se detectaron eventos especiales.

---

## BONUS: Orquestación con Prefect

### Metodología

Se transformó el pipeline de análisis en un flujo orquestado utilizando **Prefect**, una herramienta de orquestación de workflows en Python. El objetivo fue demostrar cómo convertir un notebook de análisis en un pipeline reproducible, monitoreable y resiliente.

### Implementación

**Tasks creados:**

1. **`@task cargar_datos`**: 
   - Carga datos desde URLs (soporta Parquet y CSV).
   - Incluye reintentos automáticos (`retries=3`) para manejar fallos temporales de red.
   - Logging integrado para monitoreo.

2. **`@task hacer_join_simple`**:
   - Realiza el join entre trips y zones.
   - Normaliza columnas automáticamente.
   - Crea la columna `pickup_date` si no existe.

3. **`@task analisis_rapido`**:
   - Genera estadísticas básicas del dataset integrado.
   - Calcula top boroughs, distancias y tarifas promedio.

**Flow principal:**

El flow `pipeline_taxi_simple()` orquesta el pipeline completo:
- **Paso 1**: Carga datos (trips desde Parquet, zones desde CSV).
- **Paso 2**: Realiza join entre trips y zones.
- **Paso 3**: Ejecuta análisis básico.
- **Paso 4**: Retorna resultados consolidados.

**Resultados del pipeline:**

- Total registros procesados: **3,066,766**
- Distancia promedio: **3.85 millas**
- Tarifa promedio: **$27.02**
- Top 3 Boroughs:
  - Manhattan: 2,715,369 viajes
  - Queens: 286,645 viajes
  - Unknown: 40,116 viajes

### Ventajas de Prefect

**Comparación: Código normal vs. Prefect**

| Aspecto | Código Normal | Con Prefect |
|---------|---------------|-------------|
| Manejo de errores | Si falla, todo se rompe | Reintentos automáticos configurados |
| Monitoreo | Sin visibilidad | Logs integrados y dashboard |
| Escalabilidad | Limitada | Ejecución distribuida posible |
| Reproducibilidad | Manual | Pipeline versionado y documentado |

### Preguntas Bonus

#### 1. ¿Qué ventaja tiene usar `@task` en lugar de una función normal?

Una función normal en Python se ejecuta sin ningún control extra. Cuando usas `@task`:

- **Reintentos automáticos**: Si falla por algo temporal (ej: red caída al bajar datos), Prefect lo reintenta automáticamente.
- **Timeouts**: Podés definir cortes por tiempo (si un paso tarda demasiado, Prefect lo corta).
- **Logs integrados**: Tenés logs integrados en la interfaz de Prefect.
- **Orquestación y monitoreo**: Cada task queda orquestado y monitoreado; podés ver qué falló y reintentar sólo ese paso.

#### 2. ¿Para qué sirve el `@flow` decorator?

El `@flow` define un pipeline completo, que organiza y conecta varios `@task`. Le dice a Prefect: "Esto no es sólo un script de Python, es un flujo de trabajo con dependencias y monitoreo".

Permite:

- **Ejecutar tasks en orden**: Automáticamente respeta las dependencias entre tasks.
- **Pasar resultados**: Los resultados de un task se pasan automáticamente al siguiente.
- **Monitorear el flow run entero**: Podés ver el estado de todo el pipeline en tiempo real.

#### 3. ¿En qué casos reales usarías esto?

**Reportes diarios**: Automatizar que cada mañana se bajen datos de ventas, se limpien y se envíen reportes a un dashboard.

**Análisis automáticos**: Procesar logs de usuarios, detectar anomalías o generar alertas de fraude sin intervención manual.

**Pipelines de ML**: Cargar dataset → limpiar/preprocesar → entrenar modelo → guardar métricas. Todo orquestado y monitoreado.

**ETL en producción**: Pipelines de extracción, transformación y carga de datos que se ejecutan periódicamente con garantías de resiliencia.

---

## Reflexión

Este ejercicio permitió comprender el valor de integrar **múltiples fuentes de datos** y transformar análisis exploratorios en **pipelines reproducibles y escalables**.

### Lecciones Clave

1. **Joins enriquecen el análisis**: Los joins no solo combinan datos, sino que habilitan análisis más completos y contextualizados. El cruce entre trips, zones y calendar permitió descubrir patrones geográficos y temporales que no serían visibles en un solo dataset.

2. **LEFT JOIN vs INNER JOIN**: La elección del tipo de join es crítica. En este caso, usar LEFT JOIN preservó todos los viajes, permitiendo un análisis completo sin perder información valiosa.

3. **Optimización para grandes volúmenes**: Trabajar con 3+ millones de registros requiere técnicas específicas:
   - Optimización de tipos de datos (`int16`, `int8`) para reducir memoria.
   - Muestreo estratégico para visualizaciones.
   - Validación de performance de joins.

4. **Análisis por boroughs revela patrones de mercado**: El análisis mostró que Manhattan concentra el 88.4% de los viajes, mientras que Queens tiene los trayectos más largos. Estas diferencias pueden informar estrategias de negocio diferenciadas.

5. **Días especiales como factor de análisis**: Aunque en este período no se detectaron días especiales, la infraestructura creada permite evaluar el impacto de eventos en el futuro, demostrando la importancia de diseñar análisis extensibles.

6. **Prefect como puente hacia Data Engineering**: El BONUS con Prefect mostró la diferencia entre un simple notebook y un **pipeline orquestado y resiliente**, acercando el trabajo a escenarios reales de **Data Engineering y MLOps**. Las ventajas de reintentos automáticos, logging integrado y monitoreo son fundamentales en producción.

7. **Correlaciones monetarias fuertes**: El análisis de correlaciones reveló que las variables monetarias (`total_amount`, `fare_amount`, `tip_amount`) están fuertemente correlacionadas, mientras que la distancia tiene poca correlación directa, sugiriendo que influyen recargos adicionales, peajes o el tipo de zona.

8. **Validación de joins es esencial**: Verificar que los joins funcionaron correctamente (100% de match rate, 0 nulos en columnas clave) es crucial para garantizar la calidad del análisis posterior.

Con esto se afianza el puente entre análisis exploratorio (EDA) y la construcción de sistemas de datos reproducibles y escalables, preparando el terreno para aplicaciones más complejas en psroducción.

---

