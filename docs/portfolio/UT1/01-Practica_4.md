---
title: "UT1 - Actividad 4: EDA Multi-fuentes y Joins"
date: 27/08/2025
---

# UT1 - Actividad 4: EDA Multi-fuentes y Joins

## Contexto
En esta actividad se trabajÃ³ con datos abiertos de **NYC Taxi (enero 2023)**, complementados con informaciÃ³n de **zonas geogrÃ¡ficas** y un **calendario de eventos especiales**.  
El objetivo fue integrar mÃºltiples fuentes de datos y realizar un anÃ¡lisis exploratorio que permita extraer patrones metropolitanos y mÃ©tricas de negocio.  
AdemÃ¡s, se incluyÃ³ un **BONUS con Prefect** para convertir el pipeline en un flujo orquestado.

## Objetivos
- Integrar datos de **mÃºltiples fuentes** (Parquet, CSV, JSON).
- Aplicar tÃ©cnicas de **joins en pandas** (left join, inner join).
- Generar anÃ¡lisis agregados con `groupby`.
- Evaluar impacto de **eventos especiales** en los viajes.
- Introducir la orquestaciÃ³n de pipelines con **Prefect**.

## Actividades (con tiempos estimados)
| Actividad                          | Tiempo | Resultado esperado                                |
|-----------------------------------|:------:|--------------------------------------------------|
| Carga multi-fuentes                |  45m   | Datasets cargados y normalizados                  |
| Limpieza y optimizaciÃ³n            |  30m   | Tipos optimizados, nulos tratados                 |
| Joins (trips + zones + calendar)   |  45m   | Dataset unificado para anÃ¡lisis                   |
| AnÃ¡lisis por borough y eventos     |  45m   | Reportes comparativos (normal vs. dÃ­as especiales)|
| Bonus Prefect (pipeline simple)    |  30m   | Flow completo con @task y @flow                   |
| DocumentaciÃ³n                      |  30m   | Reflexiones y respuestas                         |

## Desarrollo
1. **Carga de datos**:
   - Viajes desde Parquet (~3M registros).
   - Zonas desde CSV.
   - Eventos calendario desde JSON.  
2. **NormalizaciÃ³n**:
   - Columnas a minÃºsculas.
   - CreaciÃ³n de `pickup_date`.
   - OptimizaciÃ³n de tipos (`int16`, `int8`).  
3. **Joins**:
   - `trips + zones` â†’ asignaciÃ³n de borough y zona.
   - `trips_with_zones + calendar` â†’ flag `is_special_day`.  
4. **AnÃ¡lisis**:
   - MÃ©tricas por borough: viajes, distancia, tarifas, revenue/km, tip rate.
   - ComparaciÃ³n dÃ­as normales vs. especiales.
   - IdentificaciÃ³n de boroughs dominantes y horas pico.
5. **Bonus Prefect**:
   - Se transformaron funciones clave en `@task`.
   - Se creÃ³ un `@flow` principal que orquesta carga â†’ join â†’ anÃ¡lisis.
   - Se destacaron ventajas de Prefect: reintentos automÃ¡ticos, logging, escalabilidad.

## Evidencias
- Notebook de anÃ¡lisis:

  [Abrir en Colab](https://colab.research.google.com/github/MatiasJorda/INGENIERIA-DATOS/blob/main/docs/portfolio/UT1/Notebooks/Practica_4.ipynb) Â·

  [Ver en GitHub](https://github.com/MatiasJorda/INGENIERIA-DATOS/blob/main/docs/portfolio/UT1/Notebooks/Practica_4.ipynb) Â·

  [Nbviewer (mirror)](https://nbviewer.org/github/MatiasJorda/INGENIERIA-DATOS/blob/main/docs/portfolio/UT1/Notebooks/Practica_4.ipynb) Â·

### Resultados:

En este proyecto trabajamos con datos de viajes en taxi en la ciudad de Nueva York, complementados con otras fuentes de informaciÃ³n. El enfoque principal fue realizar un anÃ¡lisis exploratorio de datos (EDA), aplicando operaciones de combinaciÃ³n (`join`) para integrar datasets y extraer informaciÃ³n valiosa.

---

### Carga de datos desde distintas fuentes

**Fuentes utilizadas:**

* **Viajes en taxi (formato Parquet)**: Se importaron mÃ¡s de **3 millones de registros** del dataset oficial de enero de 2023, aprovechando la eficiencia del formato `.parquet` frente al `.csv`.
* **Zonas geogrÃ¡ficas (CSV)**: Se aÃ±adiÃ³ el archivo oficial de zonas de NYC, que contiene 265 registros con identificadores de ubicaciÃ³n, boroughs, zonas y zonas de servicio.
* **Calendario de eventos (JSON)**: Se incorporÃ³ un calendario local con fechas de eventos, transformando las fechas de formato texto a tipo `datetime`.

Se validÃ³ la estructura y calidad de los datos iniciales (filas, columnas, tipos y rangos temporales), asegurando una correcta carga de cada fuente.

---

### NormalizaciÃ³n y preparaciÃ³n para combinaciones

**Transformaciones aplicadas:**

* **EstandarizaciÃ³n de nombres de columnas**: Se uniformaron a minÃºsculas para evitar errores durante los merges. Esto garantiza consistencia entre datasets que pueden tener diferentes convenciones de nomenclatura.

* **ExtracciÃ³n de fecha**: Desde `tpep_pickup_datetime` se generÃ³ una nueva columna `pickup_date` (tipo `date`) para facilitar el cruce con el calendario. El rango de fechas detectado fue desde 2008-12-31 hasta 2023-02-01.

* **Chequeo de tipos de datos clave**: Se validaron tipos compatibles entre columnas clave:
  - `pulocationid` (int64) â†” `locationid` (int64) â†’ Compatibles
  - `pickup_date` (date) â†” `calendar.date` (date) â†’ Compatibles

* **OptimizaciÃ³n de memoria**: Para un dataset de 3+ millones de registros, se aplicaron optimizaciones:
  - `pulocationid` y `dolocationid`: Convertidos a `int16` (reducciÃ³n de memoria).
  - `passenger_count`: Convertido a `int8` (valores tÃ­picamente entre 0-6).
  - `locationid` en zones: Convertido a `int16`.
  - **Resultado**: Ahorro de memoria del 8.1% (de 682.6 MB a 627.0 MB).

* **Limpieza de nulos**:
  * En `trips`: Se eliminaron filas con datos crÃ­ticos ausentes (71,743 registros eliminados, principalmente con nulos en `airport_fee`, `congestion_surcharge`, `store_and_fwd_flag`, `ratecodeid`). Se rellenaron nulos en `passenger_count` con el valor tÃ­pico de 1.
  * En `zones`: Se imputaron nulos con el valor `"Desconocido"` para mantener integridad referencial.

**ValidaciÃ³n final:**
- Total de viajes despuÃ©s de limpieza: **2,995,023**
- Viajes sin pickup location: **0**
- Viajes sin dropoff location: **0**
- Viajes sin passenger_count: **0**

Al final del proceso, los datasets quedaron **limpios, tipados adecuadamente y listos para ser combinados.**

---

### Primer Join: Trips + Zones

**MÃ©todo aplicado:**

* Se utilizÃ³ un `left join` entre los datasets de viajes y zonas, vinculando `pulocationid` con `locationid`.
* Se conservaron todos los registros originales de viajes, aÃ±adiendo los datos geogrÃ¡ficos correspondientes.

**VerificaciÃ³n de resultado:**

* Total de filas antes y despuÃ©s del merge: **2,995,023** (sin pÃ©rdida de registros).
* Nuevas columnas agregadas: `borough`, `zone`, `service_zone`.
* Nulos en `borough`: **0** â†’ Join exitoso.

**DistribuciÃ³n de pickups por borough:**

| Borough   | Viajes aproximados |
| --------- | ------------------ |
| Manhattan | \~2.65M            |
| Queens    | \~285K             |
| Unknown   | \~39K              |
| Brooklyn  | \~15K              |
| Bronx     | \~3.9K             |

---

### Segundo Join: Trips+Zones + Calendar

**CÃ³mo se realizÃ³:**

* Se combinÃ³ el dataset anterior con el calendario de eventos utilizando `pickup_date â†” date` como clave.
* Se utilizÃ³ nuevamente un `left join` para conservar todos los viajes, incluso aquellos sin evento asociado.
* Se creÃ³ una nueva columna booleana `is_special_day`, marcando como `True` si el dÃ­a coincidÃ­a con un evento, y `False` en caso contrario.

**ValidaciÃ³n final:**

* Se mantuvo el total original de viajes.
* Se garantizÃ³ la ausencia de nulos en las variables clave para anÃ¡lisis posteriores.

---

### AnÃ¡lisis agregado por Borough

**Procesamiento realizado:**

* AgrupaciÃ³n de viajes por `borough` utilizando `groupby()`.
* CÃ¡lculo de mÃ©tricas estadÃ­sticas:
  - **Conteo**: NÃºmero total de viajes por borough.
  - **Distancias**: Media, mediana y desviaciÃ³n estÃ¡ndar de `trip_distance`.
  - **Montos**: Media, mediana y desviaciÃ³n estÃ¡ndar de `total_amount`.
  - **Tarifas**: Media de `fare_amount`.
  - **Propinas**: Media y mediana de `tip_amount`.
  - **Pasajeros**: Media de `passenger_count`.

* GeneraciÃ³n de nuevas mÃ©tricas empresariales:

  * `revenue_per_km`: Ingreso promedio por kilÃ³metro (avg_total / avg_distance). Ãštil para identificar zonas mÃ¡s rentables.
  * `tip_rate`: ProporciÃ³n de propinas sobre la tarifa base (avg_tip / avg_fare * 100). Indica satisfacciÃ³n del cliente.
  * `market_share`: ParticipaciÃ³n de cada borough en el total de viajes (num_trips / total * 100). Muestra concentraciÃ³n de demanda.

**Principales hallazgos:**

* **Manhattan**: 
  - Concentra el **88.4%** de los viajes (2,648,320 viajes).
  - Recorridos cortos: **2.41 km** promedio (mediana: 1.61 km).
  - Alta tasa de propinas: **19.5%**.
  - Revenue por km: **$9.27**.
  - Tarifa promedio: **$22.35**.

* **Queens**: 
  - **9.5%** del market share (285,126 viajes).
  - Trayectos mÃ¡s largos: **12.32 km** promedio (mediana: 11.27 km).
  - Tarifas elevadas: **$67.35** promedio.
  - Revenue por km: **$5.47**.
  - Tasa de propinas: **15.7%**.

* **EWR y Desconocido**: 
  - Muestran los mayores ingresos por kilÃ³metro (**$65.21** y **$46.24** respectivamente).
  - Probablemente trayectos especiales o de larga distancia (aeropuertos, viajes interurbanos).

* **Bronx y Staten Island**: 
  - Tienen las tasas de propina mÃ¡s bajas (<3%).
  - Menor volumen de viajes pero con patrones distintivos.

---

###  DÃ­as normales vs dÃ­as especiales

**QuÃ© se hizo:**

* Agrupamiento por `borough` y `is_special_day`.
* ComparaciÃ³n de mÃ©tricas clave: nÃºmero de viajes, distancia promedio y tarifa media.

**Resultado:**

* En el mes analizado (enero 2023), **no hubo dÃ­as catalogados como especiales**.
* Todos los viajes correspondieron a dÃ­as normales.
* **Promedio general en dÃ­as normales**:

  * Viajes: 2,995,023
  * Distancia: 3.44 millas
  * Tarifa: \$26.97

---

###  TÃ©cnicas aplicadas para trabajar con grandes volÃºmenes de datos

* **Muestreo estratÃ©gico**: Se extrajo una muestra aleatoria de 10.000 viajes (0.3%) para facilitar visualizaciones rÃ¡pidas.
* **ValidaciÃ³n de joins**:

  * Coincidencias con zonas: 100%
  * Zonas utilizadas: 255/265 â†’ **96.2% de cobertura**
* **AnÃ¡lisis por hora del dÃ­a**:

  * Horas pico: **18:00 (210,761 viajes)**, **17:00** y **15:00**.

---

### AnÃ¡lisis de correlaciones numÃ©ricas

**Variables analizadas:**

* `trip_distance`
* `fare_amount`
* `tip_amount`
* `total_amount`

**Principales correlaciones:**

| Variables comparadas           | CorrelaciÃ³n   |
| ------------------------------ | ------------- |
| `total_amount` â†” `fare_amount` | **0.98**      |
| `total_amount` â†” `tip_amount`  | **0.71**      |
| `fare_amount` â†” `tip_amount`   | **0.59**      |
| `trip_distance` â†” otras        | < 0.1 (dÃ©bil) |

**InterpretaciÃ³n:**

* Existe una fuerte relaciÃ³n entre las variables monetarias, como es esperable.
* La distancia tiene poca correlaciÃ³n directa con el monto total, lo que sugiere que influyen recargos adicionales, peajes o el tipo de zona.

---

### 7. Perfiles Automatizados (Opcional)

Si se genera un reporte HTML automatizado usando herramientas como `ydata-profiling` o reportes de Prefect, estos pueden ser visualizados de la siguiente manera:

**ðŸ“„ Ver reporte HTML generado:**

- Reporte HTML pendiente (agrega enlace cuando exista)

**ðŸ’» Alternativa: Abrir desde PowerShell/CMD:**

Si el link no funciona, puedes abrir el reporte ejecutando este comando en PowerShell desde la carpeta del proyecto:

```powershell
# Navegar a la carpeta del reporte
cd "docs\portfolio\UT1\Notebooks\results\reportes"

# Abrir el HTML en el navegador predeterminado
Start-Process "nombre_del_reporte.html"
```

O desde cualquier ubicaciÃ³n usando la ruta completa:

```powershell
Start-Process "docs\portfolio\UT1\Notebooks\results\reportes\nombre_del_reporte.html"
```

**Nota:** Esta secciÃ³n estÃ¡ preparada para cuando se generen reportes HTML automatizados del anÃ¡lisis de NYC Taxi. Los reportes pueden incluir anÃ¡lisis estadÃ­sticos completos, visualizaciones interactivas y alertas sobre la calidad de los datos.

---

## Preguntas Finales

### 1. Â¿QuÃ© diferencia hay entre un LEFT JOIN y un INNER JOIN?

**LEFT JOIN**: Mantiene todos los registros de la tabla izquierda (la que se mantiene), mientras que de la tabla derecha se agregan los valores correspondientes. Si no hay coincidencia, las columnas de la tabla derecha se llenan con valores nulos.

**INNER JOIN**: Realiza una intersecciÃ³n de las dos tablas, manteniendo solo los registros que tienen coincidencias en ambas tablas. Si un registro de una tabla no tiene correspondencia en la otra, se elimina del resultado.

### 2. Â¿Por quÃ© usamos LEFT JOIN en lugar de INNER JOIN para trips+zones?

Porque al hacer LEFT JOIN nos aseguramos de mantener toda la informaciÃ³n de los viajes, agregando las zonas correspondientes a los mismos. Si hiciÃ©ramos INNER JOIN, perderÃ­amos la informaciÃ³n de los trips que no tienen zona asignada, lo cual podrÃ­a eliminar datos vÃ¡lidos y sesgar el anÃ¡lisis.

### 3. Â¿QuÃ© problemas pueden surgir al hacer joins con datos de fechas?

Los principales problemas incluyen:

- **Diferencias en el tipo de dato**: Por ejemplo, una columna puede ser `string` mientras que la otra es `datetime`.
- **Formatos de fecha distintos**: Por ejemplo, `YYYY-MM-DD` vs `DD/MM/YYYY`.
- **Valores nulos o fechas faltantes**: Pueden impedir el join o generar resultados inesperados.
- **Zonas horarias**: Si las fechas tienen informaciÃ³n de zona horaria, pueden causar desajustes.

### 4. Â¿CuÃ¡l es la ventaja de integrar mÃºltiples fuentes de datos?

Nos permite realizar un anÃ¡lisis mÃ¡s completo y contextualizado. AdemÃ¡s, se pueden cruzar variables de diferentes bases para descubrir patrones que no serÃ­an visibles en un solo dataset; esto enriquece la informaciÃ³n y habilita conclusiones mÃ¡s profundas.

### 5. Â¿QuÃ© insights de negocio obtuviste del anÃ¡lisis integrado?

- **Manhattan concentra la mayorÃ­a de los viajes** (88.4% del market share), lo que indica una alta demanda en esta zona.
- **Los viajes en Queens son mÃ¡s largos y costosos** en promedio, sugiriendo trayectos interurbanos o hacia aeropuertos.
- **El mejor revenue por kilÃ³metro es de EWR**, probablemente por ser trayectos especiales o de larga distancia.
- **Hay diferencias claras en el revenue por kilÃ³metro y en la tasa de propinas** entre boroughs, lo que puede informar estrategias de pricing diferenciadas.
- **Los dÃ­as especiales pueden tener impacto** en la distancia y tarifa promedio, aunque en este perÃ­odo no se detectaron eventos especiales.

---

## ðŸš€ BONUS: OrquestaciÃ³n con Prefect

### MetodologÃ­a

Se transformÃ³ el pipeline de anÃ¡lisis en un flujo orquestado utilizando **Prefect**, una herramienta de orquestaciÃ³n de workflows en Python. El objetivo fue demostrar cÃ³mo convertir un notebook de anÃ¡lisis en un pipeline reproducible, monitoreable y resiliente.

### ImplementaciÃ³n

**Tasks creados:**

1. **`@task cargar_datos`**: 
   - Carga datos desde URLs (soporta Parquet y CSV).
   - Incluye reintentos automÃ¡ticos (`retries=3`) para manejar fallos temporales de red.
   - Logging integrado para monitoreo.

2. **`@task hacer_join_simple`**:
   - Realiza el join entre trips y zones.
   - Normaliza columnas automÃ¡ticamente.
   - Crea la columna `pickup_date` si no existe.

3. **`@task analisis_rapido`**:
   - Genera estadÃ­sticas bÃ¡sicas del dataset integrado.
   - Calcula top boroughs, distancias y tarifas promedio.

**Flow principal:**

El flow `pipeline_taxi_simple()` orquesta el pipeline completo:
- **Paso 1**: Carga datos (trips desde Parquet, zones desde CSV).
- **Paso 2**: Realiza join entre trips y zones.
- **Paso 3**: Ejecuta anÃ¡lisis bÃ¡sico.
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

**ComparaciÃ³n: CÃ³digo normal vs. Prefect**

| Aspecto | CÃ³digo Normal | Con Prefect |
|---------|---------------|-------------|
| Manejo de errores | Si falla, todo se rompe | Reintentos automÃ¡ticos configurados |
| Monitoreo | Sin visibilidad | Logs integrados y dashboard |
| Escalabilidad | Limitada | EjecuciÃ³n distribuida posible |
| Reproducibilidad | Manual | Pipeline versionado y documentado |

### Preguntas Bonus

#### 1. Â¿QuÃ© ventaja tiene usar `@task` en lugar de una funciÃ³n normal?

Una funciÃ³n normal en Python se ejecuta sin ningÃºn control extra. Cuando usas `@task`:

- **Reintentos automÃ¡ticos**: Si falla por algo temporal (ej: red caÃ­da al bajar datos), Prefect lo reintenta automÃ¡ticamente.
- **Timeouts**: PodÃ©s definir cortes por tiempo (si un paso tarda demasiado, Prefect lo corta).
- **Logs integrados**: TenÃ©s logs integrados en la interfaz de Prefect.
- **OrquestaciÃ³n y monitoreo**: Cada task queda orquestado y monitoreado; podÃ©s ver quÃ© fallÃ³ y reintentar sÃ³lo ese paso.

#### 2. Â¿Para quÃ© sirve el `@flow` decorator?

El `@flow` define un pipeline completo, que organiza y conecta varios `@task`. Le dice a Prefect: "Esto no es sÃ³lo un script de Python, es un flujo de trabajo con dependencias y monitoreo".

Permite:

- **Ejecutar tasks en orden**: AutomÃ¡ticamente respeta las dependencias entre tasks.
- **Pasar resultados**: Los resultados de un task se pasan automÃ¡ticamente al siguiente.
- **Monitorear el flow run entero**: PodÃ©s ver el estado de todo el pipeline en tiempo real.

#### 3. Â¿En quÃ© casos reales usarÃ­as esto?

**Reportes diarios**: Automatizar que cada maÃ±ana se bajen datos de ventas, se limpien y se envÃ­en reportes a un dashboard.

**AnÃ¡lisis automÃ¡ticos**: Procesar logs de usuarios, detectar anomalÃ­as o generar alertas de fraude sin intervenciÃ³n manual.

**Pipelines de ML**: Cargar dataset â†’ limpiar/preprocesar â†’ entrenar modelo â†’ guardar mÃ©tricas. Todo orquestado y monitoreado.

**ETL en producciÃ³n**: Pipelines de extracciÃ³n, transformaciÃ³n y carga de datos que se ejecutan periÃ³dicamente con garantÃ­as de resiliencia.

---

## ReflexiÃ³n

Este ejercicio permitiÃ³ comprender el valor de integrar **mÃºltiples fuentes de datos** y transformar anÃ¡lisis exploratorios en **pipelines reproducibles y escalables**.

### Lecciones Clave

1. **Joins enriquecen el anÃ¡lisis**: Los joins no solo combinan datos, sino que habilitan anÃ¡lisis mÃ¡s completos y contextualizados. El cruce entre trips, zones y calendar permitiÃ³ descubrir patrones geogrÃ¡ficos y temporales que no serÃ­an visibles en un solo dataset.

2. **LEFT JOIN vs INNER JOIN**: La elecciÃ³n del tipo de join es crÃ­tica. En este caso, usar LEFT JOIN preservÃ³ todos los viajes, permitiendo un anÃ¡lisis completo sin perder informaciÃ³n valiosa.

3. **OptimizaciÃ³n para grandes volÃºmenes**: Trabajar con 3+ millones de registros requiere tÃ©cnicas especÃ­ficas:
   - OptimizaciÃ³n de tipos de datos (`int16`, `int8`) para reducir memoria.
   - Muestreo estratÃ©gico para visualizaciones.
   - ValidaciÃ³n de performance de joins.

4. **AnÃ¡lisis por boroughs revela patrones de mercado**: El anÃ¡lisis mostrÃ³ que Manhattan concentra el 88.4% de los viajes, mientras que Queens tiene los trayectos mÃ¡s largos. Estas diferencias pueden informar estrategias de negocio diferenciadas.

5. **DÃ­as especiales como factor de anÃ¡lisis**: Aunque en este perÃ­odo no se detectaron dÃ­as especiales, la infraestructura creada permite evaluar el impacto de eventos en el futuro, demostrando la importancia de diseÃ±ar anÃ¡lisis extensibles.

6. **Prefect como puente hacia Data Engineering**: El BONUS con Prefect mostrÃ³ la diferencia entre un simple notebook y un **pipeline orquestado y resiliente**, acercando el trabajo a escenarios reales de **Data Engineering y MLOps**. Las ventajas de reintentos automÃ¡ticos, logging integrado y monitoreo son fundamentales en producciÃ³n.

7. **Correlaciones monetarias fuertes**: El anÃ¡lisis de correlaciones revelÃ³ que las variables monetarias (`total_amount`, `fare_amount`, `tip_amount`) estÃ¡n fuertemente correlacionadas, mientras que la distancia tiene poca correlaciÃ³n directa, sugiriendo que influyen recargos adicionales, peajes o el tipo de zona.

8. **ValidaciÃ³n de joins es esencial**: Verificar que los joins funcionaron correctamente (100% de match rate, 0 nulos en columnas clave) es crucial para garantizar la calidad del anÃ¡lisis posterior.

Con esto se afianza el puente entre anÃ¡lisis exploratorio (EDA) y la construcciÃ³n de sistemas de datos reproducibles y escalables, preparando el terreno para aplicaciones mÃ¡s complejas en psroducciÃ³n.

---

