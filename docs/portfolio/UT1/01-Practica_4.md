---
title: "UT1 - Actividad 4: EDA Multi-fuentes y Joins"
date: 27/08/2025
---

# UT1 - Actividad 4: EDA Multi-fuentes y Joins

## Contexto
En esta actividad se trabaj├│ con datos abiertos de **NYC Taxi (enero 2023)**, complementados con informaci├│n de **zonas geogr├íficas** y un **calendario de eventos especiales**.  
El objetivo fue integrar m├║ltiples fuentes de datos y realizar un an├ílisis exploratorio que permita extraer patrones metropolitanos y m├®tricas de negocio.  
Adem├ís, se incluy├│ un **BONUS con Prefect** para convertir el pipeline en un flujo orquestado.

## Objetivos
- Integrar datos de **m├║ltiples fuentes** (Parquet, CSV, JSON).
- Aplicar t├®cnicas de **joins en pandas** (left join, inner join).
- Generar an├ílisis agregados con `groupby`.
- Evaluar impacto de **eventos especiales** en los viajes.
- Introducir la orquestaci├│n de pipelines con **Prefect**.

## Actividades (con tiempos estimados)
| Actividad                          | Tiempo | Resultado esperado                                |
|-----------------------------------|:------:|--------------------------------------------------|
| Carga multi-fuentes                |  45m   | Datasets cargados y normalizados                  |
| Limpieza y optimizaci├│n            |  30m   | Tipos optimizados, nulos tratados                 |
| Joins (trips + zones + calendar)   |  45m   | Dataset unificado para an├ílisis                   |
| An├ílisis por borough y eventos     |  45m   | Reportes comparativos (normal vs. d├¡as especiales)|
| Bonus Prefect (pipeline simple)    |  30m   | Flow completo con @task y @flow                   |
| Documentaci├│n                      |  30m   | Reflexiones y respuestas                         |

## Desarrollo
1. **Carga de datos**:
   - Viajes desde Parquet (~3M registros).
   - Zonas desde CSV.
   - Eventos calendario desde JSON.  
2. **Normalizaci├│n**:
   - Columnas a min├║sculas.
   - Creaci├│n de `pickup_date`.
   - Optimizaci├│n de tipos (`int16`, `int8`).  
3. **Joins**:
   - `trips + zones` ÔåÆ asignaci├│n de borough y zona.
   - `trips_with_zones + calendar` ÔåÆ flag `is_special_day`.  
4. **An├ílisis**:
   - M├®tricas por borough: viajes, distancia, tarifas, revenue/km, tip rate.
   - Comparaci├│n d├¡as normales vs. especiales.
   - Identificaci├│n de boroughs dominantes y horas pico.
5. **Bonus Prefect**:
   - Se transformaron funciones clave en `@task`.
   - Se cre├│ un `@flow` principal que orquesta carga ÔåÆ join ÔåÆ an├ílisis.
   - Se destacaron ventajas de Prefect: reintentos autom├íticos, logging, escalabilidad.

## Evidencias
- Notebook de an├ílisis:

  [Abrir en Colab](https://colab.research.google.com/github/MatiasJorda/INGENIERIA-DATOS/blob/main/docs/portfolio/UT1/Notebooks/Practica_4.ipynb) ┬À

  [Ver en GitHub](https://github.com/MatiasJorda/INGENIERIA-DATOS/blob/main/docs/portfolio/UT1/Notebooks/Practica_4.ipynb) ┬À

  [Nbviewer (mirror)](https://nbviewer.org/github/MatiasJorda/INGENIERIA-DATOS/blob/main/docs/portfolio/UT1/Notebooks/Practica_4.ipynb) ┬À

### Resultados:

En este proyecto trabajamos con datos de viajes en taxi en la ciudad de Nueva York, complementados con otras fuentes de informaci├│n. El enfoque principal fue realizar un an├ílisis exploratorio de datos (EDA), aplicando operaciones de combinaci├│n (`join`) para integrar datasets y extraer informaci├│n valiosa.

---

### Carga de datos desde distintas fuentes

**Fuentes utilizadas:**

* **Viajes en taxi (formato Parquet)**: Se importaron m├ís de **3 millones de registros** del dataset oficial de enero de 2023, aprovechando la eficiencia del formato `.parquet` frente al `.csv`.
* **Zonas geogr├íficas (CSV)**: Se a├▒adi├│ el archivo oficial de zonas de NYC, que contiene 265 registros con identificadores de ubicaci├│n, boroughs, zonas y zonas de servicio.
* **Calendario de eventos (JSON)**: Se incorpor├│ un calendario local con fechas de eventos, transformando las fechas de formato texto a tipo `datetime`.

Se valid├│ la estructura y calidad de los datos iniciales (filas, columnas, tipos y rangos temporales), asegurando una correcta carga de cada fuente.

---

### Normalizaci├│n y preparaci├│n para combinaciones

**Transformaciones aplicadas:**

* **Estandarizaci├│n de nombres de columnas**: Se uniformaron a min├║sculas para evitar errores durante los merges. Esto garantiza consistencia entre datasets que pueden tener diferentes convenciones de nomenclatura.

* **Extracci├│n de fecha**: Desde `tpep_pickup_datetime` se gener├│ una nueva columna `pickup_date` (tipo `date`) para facilitar el cruce con el calendario. El rango de fechas detectado fue desde 2008-12-31 hasta 2023-02-01.

* **Chequeo de tipos de datos clave**: Se validaron tipos compatibles entre columnas clave:
  - `pulocationid` (int64) Ôåö `locationid` (int64) ÔåÆ Compatibles
  - `pickup_date` (date) Ôåö `calendar.date` (date) ÔåÆ Compatibles

* **Optimizaci├│n de memoria**: Para un dataset de 3+ millones de registros, se aplicaron optimizaciones:
  - `pulocationid` y `dolocationid`: Convertidos a `int16` (reducci├│n de memoria).
  - `passenger_count`: Convertido a `int8` (valores t├¡picamente entre 0-6).
  - `locationid` en zones: Convertido a `int16`.
  - **Resultado**: Ahorro de memoria del 8.1% (de 682.6 MB a 627.0 MB).

* **Limpieza de nulos**:
  * En `trips`: Se eliminaron filas con datos cr├¡ticos ausentes (71,743 registros eliminados, principalmente con nulos en `airport_fee`, `congestion_surcharge`, `store_and_fwd_flag`, `ratecodeid`). Se rellenaron nulos en `passenger_count` con el valor t├¡pico de 1.
  * En `zones`: Se imputaron nulos con el valor `"Desconocido"` para mantener integridad referencial.

**Validaci├│n final:**
- Total de viajes despu├®s de limpieza: **2,995,023**
- Viajes sin pickup location: **0**
- Viajes sin dropoff location: **0**
- Viajes sin passenger_count: **0**

Al final del proceso, los datasets quedaron **limpios, tipados adecuadamente y listos para ser combinados.**

---

### Primer Join: Trips + Zones

**M├®todo aplicado:**

* Se utiliz├│ un `left join` entre los datasets de viajes y zonas, vinculando `pulocationid` con `locationid`.
* Se conservaron todos los registros originales de viajes, a├▒adiendo los datos geogr├íficos correspondientes.

**Verificaci├│n de resultado:**

* Total de filas antes y despu├®s del merge: **2,995,023** (sin p├®rdida de registros).
* Nuevas columnas agregadas: `borough`, `zone`, `service_zone`.
* Nulos en `borough`: **0** ÔåÆ Join exitoso.

**Distribuci├│n de pickups por borough:**

| Borough   | Viajes aproximados |
| --------- | ------------------ |
| Manhattan | \~2.65M            |
| Queens    | \~285K             |
| Unknown   | \~39K              |
| Brooklyn  | \~15K              |
| Bronx     | \~3.9K             |

---

### Segundo Join: Trips+Zones + Calendar

**C├│mo se realiz├│:**

* Se combin├│ el dataset anterior con el calendario de eventos utilizando `pickup_date Ôåö date` como clave.
* Se utiliz├│ nuevamente un `left join` para conservar todos los viajes, incluso aquellos sin evento asociado.
* Se cre├│ una nueva columna booleana `is_special_day`, marcando como `True` si el d├¡a coincid├¡a con un evento, y `False` en caso contrario.

**Validaci├│n final:**

* Se mantuvo el total original de viajes.
* Se garantiz├│ la ausencia de nulos en las variables clave para an├ílisis posteriores.

---

### An├ílisis agregado por Borough

**Procesamiento realizado:**

* Agrupaci├│n de viajes por `borough` utilizando `groupby()`.
* C├ílculo de m├®tricas estad├¡sticas:
  - **Conteo**: N├║mero total de viajes por borough.
  - **Distancias**: Media, mediana y desviaci├│n est├índar de `trip_distance`.
  - **Montos**: Media, mediana y desviaci├│n est├índar de `total_amount`.
  - **Tarifas**: Media de `fare_amount`.
  - **Propinas**: Media y mediana de `tip_amount`.
  - **Pasajeros**: Media de `passenger_count`.

* Generaci├│n de nuevas m├®tricas empresariales:

  * `revenue_per_km`: Ingreso promedio por kil├│metro (avg_total / avg_distance). ├Ütil para identificar zonas m├ís rentables.
  * `tip_rate`: Proporci├│n de propinas sobre la tarifa base (avg_tip / avg_fare * 100). Indica satisfacci├│n del cliente.
  * `market_share`: Participaci├│n de cada borough en el total de viajes (num_trips / total * 100). Muestra concentraci├│n de demanda.

**Principales hallazgos:**

* **Manhattan**: 
  - Concentra el **88.4%** de los viajes (2,648,320 viajes).
  - Recorridos cortos: **2.41 km** promedio (mediana: 1.61 km).
  - Alta tasa de propinas: **19.5%**.
  - Revenue por km: **$9.27**.
  - Tarifa promedio: **$22.35**.

* **Queens**: 
  - **9.5%** del market share (285,126 viajes).
  - Trayectos m├ís largos: **12.32 km** promedio (mediana: 11.27 km).
  - Tarifas elevadas: **$67.35** promedio.
  - Revenue por km: **$5.47**.
  - Tasa de propinas: **15.7%**.

* **EWR y Desconocido**: 
  - Muestran los mayores ingresos por kil├│metro (**$65.21** y **$46.24** respectivamente).
  - Probablemente trayectos especiales o de larga distancia (aeropuertos, viajes interurbanos).

* **Bronx y Staten Island**: 
  - Tienen las tasas de propina m├ís bajas (<3%).
  - Menor volumen de viajes pero con patrones distintivos.

---

###  D├¡as normales vs d├¡as especiales

**Qu├® se hizo:**

* Agrupamiento por `borough` y `is_special_day`.
* Comparaci├│n de m├®tricas clave: n├║mero de viajes, distancia promedio y tarifa media.

**Resultado:**

* En el mes analizado (enero 2023), **no hubo d├¡as catalogados como especiales**.
* Todos los viajes correspondieron a d├¡as normales.
* **Promedio general en d├¡as normales**:

  * Viajes: 2,995,023
  * Distancia: 3.44 millas
  * Tarifa: \$26.97

---

###  T├®cnicas aplicadas para trabajar con grandes vol├║menes de datos

* **Muestreo estrat├®gico**: Se extrajo una muestra aleatoria de 10.000 viajes (0.3%) para facilitar visualizaciones r├ípidas.
* **Validaci├│n de joins**:

  * Coincidencias con zonas: 100%
  * Zonas utilizadas: 255/265 ÔåÆ **96.2% de cobertura**
* **An├ílisis por hora del d├¡a**:

  * Horas pico: **18:00 (210,761 viajes)**, **17:00** y **15:00**.

---

### An├ílisis de correlaciones num├®ricas

**Variables analizadas:**

* `trip_distance`
* `fare_amount`
* `tip_amount`
* `total_amount`

**Principales correlaciones:**

| Variables comparadas           | Correlaci├│n   |
| ------------------------------ | ------------- |
| `total_amount` Ôåö `fare_amount` | **0.98**      |
| `total_amount` Ôåö `tip_amount`  | **0.71**      |
| `fare_amount` Ôåö `tip_amount`   | **0.59**      |
| `trip_distance` Ôåö otras        | < 0.1 (d├®bil) |

**Interpretaci├│n:**

* Existe una fuerte relaci├│n entre las variables monetarias, como es esperable.
* La distancia tiene poca correlaci├│n directa con el monto total, lo que sugiere que influyen recargos adicionales, peajes o el tipo de zona.

---

### 7. Perfiles Automatizados (Opcional)

Si se genera un reporte HTML automatizado usando herramientas como `ydata-profiling` o reportes de Prefect, estos pueden ser visualizados de la siguiente manera:

**­ƒôä Ver reporte HTML generado:**

- Reporte HTML pendiente (agrega enlace cuando exista)

**­ƒÆ╗ Alternativa: Abrir desde PowerShell/CMD:**

Si el link no funciona, puedes abrir el reporte ejecutando este comando en PowerShell desde la carpeta del proyecto:

```powershell
# Navegar a la carpeta del reporte
cd "docs\portfolio\UT1\Notebooks\results\reportes"

# Abrir el HTML en el navegador predeterminado
Start-Process "nombre_del_reporte.html"
```

O desde cualquier ubicaci├│n usando la ruta completa:

```powershell
Start-Process "docs\portfolio\UT1\Notebooks\results\reportes\nombre_del_reporte.html"
```

**Nota:** Esta secci├│n est├í preparada para cuando se generen reportes HTML automatizados del an├ílisis de NYC Taxi. Los reportes pueden incluir an├ílisis estad├¡sticos completos, visualizaciones interactivas y alertas sobre la calidad de los datos.

---

## Preguntas Finales

### 1. ┬┐Qu├® diferencia hay entre un LEFT JOIN y un INNER JOIN?

**LEFT JOIN**: Mantiene todos los registros de la tabla izquierda (la que se mantiene), mientras que de la tabla derecha se agregan los valores correspondientes. Si no hay coincidencia, las columnas de la tabla derecha se llenan con valores nulos.

**INNER JOIN**: Realiza una intersecci├│n de las dos tablas, manteniendo solo los registros que tienen coincidencias en ambas tablas. Si un registro de una tabla no tiene correspondencia en la otra, se elimina del resultado.

### 2. ┬┐Por qu├® usamos LEFT JOIN en lugar de INNER JOIN para trips+zones?

Porque al hacer LEFT JOIN nos aseguramos de mantener toda la informaci├│n de los viajes, agregando las zonas correspondientes a los mismos. Si hici├®ramos INNER JOIN, perder├¡amos la informaci├│n de los trips que no tienen zona asignada, lo cual podr├¡a eliminar datos v├ílidos y sesgar el an├ílisis.

### 3. ┬┐Qu├® problemas pueden surgir al hacer joins con datos de fechas?

Los principales problemas incluyen:

- **Diferencias en el tipo de dato**: Por ejemplo, una columna puede ser `string` mientras que la otra es `datetime`.
- **Formatos de fecha distintos**: Por ejemplo, `YYYY-MM-DD` vs `DD/MM/YYYY`.
- **Valores nulos o fechas faltantes**: Pueden impedir el join o generar resultados inesperados.
- **Zonas horarias**: Si las fechas tienen informaci├│n de zona horaria, pueden causar desajustes.

### 4. ┬┐Cu├íl es la ventaja de integrar m├║ltiples fuentes de datos?

Nos permite realizar un an├ílisis m├ís completo y contextualizado. Adem├ís, se pueden cruzar variables de diferentes bases para descubrir patrones que no ser├¡an visibles en un solo dataset; esto enriquece la informaci├│n y habilita conclusiones m├ís profundas.

### 5. ┬┐Qu├® insights de negocio obtuviste del an├ílisis integrado?

- **Manhattan concentra la mayor├¡a de los viajes** (88.4% del market share), lo que indica una alta demanda en esta zona.
- **Los viajes en Queens son m├ís largos y costosos** en promedio, sugiriendo trayectos interurbanos o hacia aeropuertos.
- **El mejor revenue por kil├│metro es de EWR**, probablemente por ser trayectos especiales o de larga distancia.
- **Hay diferencias claras en el revenue por kil├│metro y en la tasa de propinas** entre boroughs, lo que puede informar estrategias de pricing diferenciadas.
- **Los d├¡as especiales pueden tener impacto** en la distancia y tarifa promedio, aunque en este per├¡odo no se detectaron eventos especiales.

---

## ­ƒÜÇ BONUS: Orquestaci├│n con Prefect

### Metodolog├¡a

Se transform├│ el pipeline de an├ílisis en un flujo orquestado utilizando **Prefect**, una herramienta de orquestaci├│n de workflows en Python. El objetivo fue demostrar c├│mo convertir un notebook de an├ílisis en un pipeline reproducible, monitoreable y resiliente.

### Implementaci├│n

**Tasks creados:**

1. **`@task cargar_datos`**: 
   - Carga datos desde URLs (soporta Parquet y CSV).
   - Incluye reintentos autom├íticos (`retries=3`) para manejar fallos temporales de red.
   - Logging integrado para monitoreo.

2. **`@task hacer_join_simple`**:
   - Realiza el join entre trips y zones.
   - Normaliza columnas autom├íticamente.
   - Crea la columna `pickup_date` si no existe.

3. **`@task analisis_rapido`**:
   - Genera estad├¡sticas b├ísicas del dataset integrado.
   - Calcula top boroughs, distancias y tarifas promedio.

**Flow principal:**

El flow `pipeline_taxi_simple()` orquesta el pipeline completo:
- **Paso 1**: Carga datos (trips desde Parquet, zones desde CSV).
- **Paso 2**: Realiza join entre trips y zones.
- **Paso 3**: Ejecuta an├ílisis b├ísico.
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

**Comparaci├│n: C├│digo normal vs. Prefect**

| Aspecto | C├│digo Normal | Con Prefect |
|---------|---------------|-------------|
| Manejo de errores | Si falla, todo se rompe | Reintentos autom├íticos configurados |
| Monitoreo | Sin visibilidad | Logs integrados y dashboard |
| Escalabilidad | Limitada | Ejecuci├│n distribuida posible |
| Reproducibilidad | Manual | Pipeline versionado y documentado |

### Preguntas Bonus

#### 1. ┬┐Qu├® ventaja tiene usar `@task` en lugar de una funci├│n normal?

Una funci├│n normal en Python se ejecuta sin ning├║n control extra. Cuando usas `@task`:

- **Reintentos autom├íticos**: Si falla por algo temporal (ej: red ca├¡da al bajar datos), Prefect lo reintenta autom├íticamente.
- **Timeouts**: Pod├®s definir cortes por tiempo (si un paso tarda demasiado, Prefect lo corta).
- **Logs integrados**: Ten├®s logs integrados en la interfaz de Prefect.
- **Orquestaci├│n y monitoreo**: Cada task queda orquestado y monitoreado; pod├®s ver qu├® fall├│ y reintentar s├│lo ese paso.

#### 2. ┬┐Para qu├® sirve el `@flow` decorator?

El `@flow` define un pipeline completo, que organiza y conecta varios `@task`. Le dice a Prefect: "Esto no es s├│lo un script de Python, es un flujo de trabajo con dependencias y monitoreo".

Permite:

- **Ejecutar tasks en orden**: Autom├íticamente respeta las dependencias entre tasks.
- **Pasar resultados**: Los resultados de un task se pasan autom├íticamente al siguiente.
- **Monitorear el flow run entero**: Pod├®s ver el estado de todo el pipeline en tiempo real.

#### 3. ┬┐En qu├® casos reales usar├¡as esto?

**Reportes diarios**: Automatizar que cada ma├▒ana se bajen datos de ventas, se limpien y se env├¡en reportes a un dashboard.

**An├ílisis autom├íticos**: Procesar logs de usuarios, detectar anomal├¡as o generar alertas de fraude sin intervenci├│n manual.

**Pipelines de ML**: Cargar dataset ÔåÆ limpiar/preprocesar ÔåÆ entrenar modelo ÔåÆ guardar m├®tricas. Todo orquestado y monitoreado.

**ETL en producci├│n**: Pipelines de extracci├│n, transformaci├│n y carga de datos que se ejecutan peri├│dicamente con garant├¡as de resiliencia.

---

## Reflexi├│n

Este ejercicio permiti├│ comprender el valor de integrar **m├║ltiples fuentes de datos** y transformar an├ílisis exploratorios en **pipelines reproducibles y escalables**.

### Lecciones Clave

1. **Joins enriquecen el an├ílisis**: Los joins no solo combinan datos, sino que habilitan an├ílisis m├ís completos y contextualizados. El cruce entre trips, zones y calendar permiti├│ descubrir patrones geogr├íficos y temporales que no ser├¡an visibles en un solo dataset.

2. **LEFT JOIN vs INNER JOIN**: La elecci├│n del tipo de join es cr├¡tica. En este caso, usar LEFT JOIN preserv├│ todos los viajes, permitiendo un an├ílisis completo sin perder informaci├│n valiosa.

3. **Optimizaci├│n para grandes vol├║menes**: Trabajar con 3+ millones de registros requiere t├®cnicas espec├¡ficas:
   - Optimizaci├│n de tipos de datos (`int16`, `int8`) para reducir memoria.
   - Muestreo estrat├®gico para visualizaciones.
   - Validaci├│n de performance de joins.

4. **An├ílisis por boroughs revela patrones de mercado**: El an├ílisis mostr├│ que Manhattan concentra el 88.4% de los viajes, mientras que Queens tiene los trayectos m├ís largos. Estas diferencias pueden informar estrategias de negocio diferenciadas.

5. **D├¡as especiales como factor de an├ílisis**: Aunque en este per├¡odo no se detectaron d├¡as especiales, la infraestructura creada permite evaluar el impacto de eventos en el futuro, demostrando la importancia de dise├▒ar an├ílisis extensibles.

6. **Prefect como puente hacia Data Engineering**: El BONUS con Prefect mostr├│ la diferencia entre un simple notebook y un **pipeline orquestado y resiliente**, acercando el trabajo a escenarios reales de **Data Engineering y MLOps**. Las ventajas de reintentos autom├íticos, logging integrado y monitoreo son fundamentales en producci├│n.

7. **Correlaciones monetarias fuertes**: El an├ílisis de correlaciones revel├│ que las variables monetarias (`total_amount`, `fare_amount`, `tip_amount`) est├ín fuertemente correlacionadas, mientras que la distancia tiene poca correlaci├│n directa, sugiriendo que influyen recargos adicionales, peajes o el tipo de zona.

8. **Validaci├│n de joins es esencial**: Verificar que los joins funcionaron correctamente (100% de match rate, 0 nulos en columnas clave) es crucial para garantizar la calidad del an├ílisis posterior.

Con esto se afianza el puente entre an├ílisis exploratorio (EDA) y la construcci├│n de sistemas de datos reproducibles y escalables, preparando el terreno para aplicaciones m├ís complejas en psroducci├│n.

---

