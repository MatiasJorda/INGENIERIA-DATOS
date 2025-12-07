---
title: "UT5 - Google Cloud Skills: Creating a Data Transformation Pipeline with Cloud Dataprep"
date: 2025
---

# UT5 - Google Cloud Skills: Creating a Data Transformation Pipeline with Cloud Dataprep

## Contexto

En esta actividad se completó el lab intermedio **"Creating a Data Transformation Pipeline with Cloud Dataprep" (GSP430)** de Google Cloud Skills Boost. Este lab proporcionó experiencia práctica con Cloud Dataprep (anteriormente conocido como Trifacta), un servicio inteligente de datos para explorar, limpiar y preparar visualmente datos estructurados y no estructurados para análisis. El objetivo fue construir un pipeline de transformación de datos utilizando la interfaz visual de Dataprep, conectando datos de BigQuery como fuente y destino, y aprendiendo técnicas de limpieza y enriquecimiento de datos para análisis de e-commerce.

**Fuente**: [Google Cloud Skills Boost - Lab GSP430](https://www.skills.google/focuses/4415?catalog_rank=%7B%22rank%22%3A6%2C%22num_filters%22%3A1%2C%22has_search%22%3Atrue%7D&parent=catalog&search_id=60910456)

**Nivel**: Intermedio  
**Duración**: 1 hora 15 minutos  
**Costo**: 5 créditos  
**Desarrollado con**: Alteryx Designer Cloud (Trifacta)

## Objetivos

- Familiarizarse con la interfaz de usuario de Cloud Dataprep
- Conectar datasets de BigQuery a Cloud Dataprep como fuente de datos
- Explorar calidad y estructura de datos utilizando las herramientas visuales de Dataprep
- Construir un pipeline de transformación de datos con operaciones de limpieza
- Enriquecer datos mediante cálculos y transformaciones personalizadas
- Ejecutar trabajos de transformación y exportar resultados a BigQuery
- Entender el flujo completo de datos desde BigQuery → Dataprep → BigQuery

## Actividades (con tiempos estimados)

| Actividad                          | Tiempo | Resultado esperado                                    |
|-----------------------------------|:------:|------------------------------------------------------|
| Configuración inicial de Dataprep |  10m   | Cloud Dataprep habilitado y accesible en Cloud Console |
| Creación de dataset en BigQuery   |  10m   | Dataset `ecommerce` creado con tabla raw de sesiones |
| Conexión BigQuery a Dataprep      |  10m   | Flow creado y conectado a fuente de datos BigQuery   |
| Exploración de campos de datos    |  15m   | Comprensión de estructura, tipos de datos y calidad  |
| Limpieza de datos                 |  15m   | Pipeline con transformaciones de limpieza aplicadas  |
| Enriquecimiento de datos          |  15m   | Columnas calculadas y transformaciones complejas     |
| Ejecución de jobs a BigQuery      |  10m   | Tabla final `revenue_reporting` creada en BigQuery   |

## Desarrollo

### 1. Configuración Inicial de Cloud Dataprep

#### Habilitación del servicio

Antes de acceder a Dataprep, fue necesario crear la identidad del servicio. Se ejecutó en Cloud Shell:

```bash
gcloud beta services identity create --service=dataprep.googleapis.com
```

Este comando crea la identidad del servicio necesaria para que Dataprep acceda a los recursos de Google Cloud.

#### Acceso a Dataprep

El proceso de acceso incluyó varios pasos:

1. **Navegación**: Desde Cloud Console, menú de navegación → **View All Products** → **Analytics** → **Alteryx Designer Cloud**

2. **Aceptación de términos**: 
   - Aceptar términos de servicio de Google Dataprep
   - Aceptar compartir información de cuenta con Alteryx (el patrocinador del lab)

3. **Permisos**: Otorgar acceso a Alteryx para el proyecto

4. **Autenticación**: Iniciar sesión con credenciales de Qwiklabs y aceptar términos de Alteryx

5. **Configuración de almacenamiento**: Usar ubicación predeterminada para el bucket de almacenamiento

**Nota importante**: Este lab requiere **Google Chrome** como navegador. Otros navegadores actualmente no son compatibles con Dataprep.

### 2. Creación de Dataset en BigQuery

#### Contexto de BigQuery

Aunque el lab se enfoca en Dataprep, BigQuery desempeña dos roles cruciales:
- **Fuente de datos**: Datos de entrada para el pipeline de Dataprep
- **Destino de datos**: Almacenamiento final de los datos transformados

#### Creación del dataset

1. Acceso a BigQuery: Navegación → **BigQuery** en Cloud Console

2. Creación del dataset:
   - **Dataset ID**: `ecommerce`
   - Configuraciones por defecto para el resto de opciones
   - Ubicación: Por defecto (multi-región US)

3. Carga de datos iniciales: Se ejecutó una consulta SQL para copiar un subconjunto del dataset público de e-commerce:

```sql
#standardSQL
CREATE OR REPLACE TABLE ecommerce.all_sessions_raw_dataprep
OPTIONS(
  description="Raw data from analyst team to ingest into Cloud Dataprep"
) AS
SELECT *
FROM `data-to-insights.ecommerce.all_sessions_raw`
WHERE date = '20170801';  -- limiting to one day of data, 56k rows
```

**Resultado**: Se creó la tabla `all_sessions_raw_dataprep` con aproximadamente **56,000 registros** de sesiones de un solo día (1 de agosto de 2017) del Google Merchandise Store, obtenidos de Google Analytics.

#### Estructura del dataset

El dataset contiene información de sesiones de Google Analytics del Google Merchandise Store, incluyendo:
- Información del visitante (fullVisitorId, visitId)
- Datos de sesión (visitStartTime, visitNumber)
- Información geográfica (country, city)
- Métricas de tráfico (pageviews, timeOnSite)
- Información de transacciones (eCommerceAction_type, totalTransactionRevenue)
- Detalles de productos y páginas visitadas

### 3. Conexión de BigQuery a Cloud Dataprep

#### Creación de un Flow

En Dataprep, un **Flow** es un contenedor que organiza todas las transformaciones de datos de un proyecto. Representa el pipeline completo de transformación.

**Pasos para crear el flow**:

1. Hacer clic en **Create a new flow** en la esquina superior derecha
2. Configurar el flow:
   - **Flow Name**: `Ecommerce Analytics Pipeline`
   - **Flow Description**: `Revenue reporting table`
3. Si aparece un popup explicativo, seleccionar **Don't show me any helpers**

#### Conexión a BigQuery

1. Hacer clic en el **Add Icon** dentro del cuadro de Dataset
2. Seleccionar **BigQuery** como fuente de datos
3. Seleccionar el proyecto y el dataset `ecommerce`
4. Seleccionar la tabla `all_sessions_raw_dataprep`
5. Hacer clic en **Add**

**Resultado**: El dataset de BigQuery se conecta al flow de Dataprep, y Dataprep comienza a analizar automáticamente la estructura y calidad de los datos.

### 4. Exploración de Campos de Datos con la UI

#### Interfaz de exploración de Dataprep

La interfaz de Dataprep proporciona visualizaciones poderosas para explorar datos:

**Panel de esquema (izquierda)**:
- Lista todas las columnas del dataset
- Muestra tipos de datos detectados automáticamente
- Permite expandir columnas para ver detalles

**Vista de datos (centro)**:
- Muestra una muestra de los datos
- Resalta problemas de calidad de datos
- Permite ver distribuciones de valores

**Panel de sugerencias (derecha)**:
- Proporciona recomendaciones automáticas basadas en problemas detectados
- Sugiere transformaciones comunes
- Agrupa sugerencias por tipo (limpieza, tipo de datos, etc.)

#### Herramientas de exploración

**1. Histogramas de calidad**:
- Muestra distribuciones de valores por columna
- Identifica valores nulos, duplicados, y outliers
- Permite hacer clic en barras para filtrar filas

**2. Visualización de tipos**:
- Dataprep detecta automáticamente tipos de datos
- Resalta columnas con tipos inconsistentes
- Sugiere conversiones de tipo apropiadas

**3. Métricas de calidad**:
- Porcentaje de valores válidos por columna
- Conteo de valores únicos
- Detección de patrones y formatos

#### Hallazgos durante la exploración

Durante la exploración se identificaron varios aspectos del dataset:

- **Columnas numéricas**: totalTransactionRevenue, pageviews, timeOnSite
- **Columnas categóricas**: country, city, productSKU
- **Columnas de identificación**: fullVisitorId, visitId
- **Columnas de fecha/hora**: date, visitStartTime
- **Valores nulos**: Presentes en varias columnas, especialmente en campos de transacción
- **Tipos de acción de e-commerce**: Valores numéricos (0-8) que representan diferentes acciones

### 5. Limpieza de Datos

#### Concepto de Recipe en Dataprep

En Dataprep, un **Recipe** es la secuencia de transformaciones que se aplican a los datos. Cada transformación se agrega como un paso en el recipe, y Dataprep muestra una vista previa en tiempo real de cómo cada paso afecta los datos.

#### Transformaciones de limpieza aplicadas

**1. Filtrado de tipos de hit**

El dataset contenía diferentes tipos de hits (PAGE, EVENT, TRANSACTION, etc.). Para el análisis de revenue, solo se necesitaban los hits de tipo PAGE (visualizaciones de página).

**Metodología**:
- Revisar el histograma de la columna `type`
- Hacer clic en la barra correspondiente a "PAGE" para seleccionar esas filas
- En el panel de sugerencias, seleccionar **Keep rows** bajo la sugerencia apropiada
- Aplicar el filtro

**Resultado**: El dataset se filtra para incluir solo visualizaciones de páginas, eliminando eventos, transacciones directas y otros tipos de hits que no son relevantes para el análisis de páginas.

**2. Eliminación de columnas innecesarias**

Se identificaron columnas que no aportaban valor al análisis de revenue:
- Columnas con solo valores nulos
- Columnas de metadatos internos no relevantes
- Columnas duplicadas o redundantes

**Metodología**:
- Seleccionar columnas en el panel de esquema
- Usar la opción **Remove column** del menú de columna
- Verificar la vista previa antes de confirmar

### 6. Enriquecimiento de Datos

#### Creación de identificador único de sesión

**Problema identificado**: El dataset tenía dos columnas separadas (`fullVisitorId` y `visitId`), pero ninguna combinación única para identificar una sesión completa. Según la documentación del esquema, `visitId` solo es único por usuario, no globalmente.

**Solución**: Crear una columna combinada concatenando `fullVisitorId` y `visitId` con un separador.

**Metodología**:
1. Seleccionar el icono **Merge columns** en la barra de herramientas
2. Seleccionar columnas: `fullVisitorId` y `visitId`
3. Especificar separador: `-` (guion)
4. Nombre de nueva columna: `unique_session_id`
5. Aplicar transformación

**Resultado**: Se crea la columna `unique_session_id` con valores como `"123456789_98765432-1234567890"`, proporcionando un identificador único para cada sesión de usuario.

#### Creación de etiquetas para tipos de acción de e-commerce

**Problema identificado**: La columna `eCommerceAction_type` contenía valores numéricos (0-8) que representaban diferentes acciones de e-commerce, pero estos números no eran intuitivos para los usuarios finales del análisis.

**Mapeo de valores**:
- 0: 'Unknown'
- 1: 'Click through of product lists'
- 2: 'Product detail views'
- 3: 'Add product(s) to cart'
- 4: 'Remove product(s) from cart'
- 5: 'Check out'
- 6: 'Completed purchase'
- 7: 'Refund of purchase'
- 8: 'Checkout options'

**Metodología - Case Statement**:
1. Seleccionar **Conditions** en la barra de herramientas → **Case on single column**
2. Columna a evaluar: `eCommerceAction_type`
3. Agregar 9 casos (uno para cada valor 0-8)
4. Para cada caso, especificar:
   - **Comparison**: El valor numérico (0, 1, 2, etc.)
   - **New value**: La etiqueta descriptiva correspondiente
5. Nombre de nueva columna: `eCommerceAction_label`
6. Aplicar transformación

**Resultado**: Se crea la columna `eCommerceAction_label` con valores legibles como "Add product(s) to cart" o "Completed purchase", facilitando el análisis y la interpretación de los datos.

#### Ajuste de valores de revenue

**Problema identificado**: Según la documentación del esquema, la columna `totalTransactionRevenue` contiene valores multiplicados por 10^6. Por ejemplo, un valor de 2.40 dólares se almacena como 2400000.

**Solución**: Dividir los valores por 1,000,000 para obtener los valores originales en dólares.

**Metodología - Custom Formula**:
1. Abrir el menú de la columna `totalTransactionRevenue`
2. Seleccionar **Calculate** → **Custom formula**
3. Fórmula: `DIVIDE(totalTransactionRevenue,1000000)`
4. Nombre de nueva columna: `totalTransactionRevenue1`
5. Aplicar transformación
6. Cambiar el tipo de datos a **Decimal** para mejor precisión

**Resultado**: Los valores de revenue se normalizan a dólares reales (ej: 2400000 → 2.40), facilitando el análisis y la visualización.

### 7. Ejecución de Jobs de Cloud Dataprep a BigQuery

#### Configuración del job

**Ambiente de ejecución**: 
- Seleccionar **Dataflow + BigQuery** como ambiente de ejecución
- Esto utiliza Google Cloud Dataflow (Apache Beam) para ejecutar las transformaciones de manera escalable

**Acciones de publicación**:
1. En la página **Run Job**, hacer clic en **Edit** junto a **Create-CSV**
2. Seleccionar **BigQuery** del menú izquierdo
3. Seleccionar el dataset `ecommerce`
4. Seleccionar **Create a New Table**
5. Nombre de tabla: `revenue_reporting`
6. Opción importante: **Drop the Table every run** (permite re-ejecutar el job sin errores de tabla existente)
7. Hacer clic en **Update**
8. Hacer clic en **RUN**

#### Proceso de ejecución

Una vez iniciado el job:

1. **Validación**: Dataprep valida el recipe completo
2. **Compilación**: Las transformaciones se compilan a código Apache Beam
3. **Ejecución en Dataflow**: El job se ejecuta en Cloud Dataflow como un pipeline distribuido
4. **Escritura a BigQuery**: Los resultados transformados se escriben a la tabla especificada

**Tiempo estimado**: Dependiendo del tamaño de los datos, el job puede tardar varios minutos. Dataprep proporciona una barra de progreso y logs de ejecución.

**Nota de troubleshooting**: Si el job falla, se recomienda esperar un minuto, presionar el botón atrás del navegador, y volver a ejecutar el job con las mismas configuraciones.

#### Verificación de resultados

Después de la ejecución exitosa:

1. Volver a BigQuery en Cloud Console
2. Refrescar la página del dataset `ecommerce`
3. Verificar que la tabla `revenue_reporting` existe
4. Explorar los datos transformados para confirmar que todas las transformaciones se aplicaron correctamente

**Resultado final**: Una tabla limpia y enriquecida en BigQuery lista para análisis, con:
- Identificadores únicos de sesión
- Etiquetas legibles para acciones de e-commerce
- Valores de revenue normalizados
- Solo datos relevantes para análisis de páginas

## Conceptos Clave Aprendidos

### 1. Cloud Dataprep (Alteryx Designer Cloud)

**Definición**: Cloud Dataprep es un servicio inteligente de datos para explorar, limpiar y preparar visualmente datos estructurados y no estructurados para análisis. Está basado en la tecnología de Trifacta (ahora parte de Alteryx).

**Características principales**:
- **Interfaz visual intuitiva**: No requiere programación para la mayoría de transformaciones
- **Detección automática de problemas**: Identifica automáticamente problemas de calidad de datos
- **Sugerencias inteligentes**: Recomienda transformaciones basadas en el análisis de datos
- **Vista previa en tiempo real**: Muestra cómo cada transformación afecta los datos antes de ejecutar
- **Integración con Google Cloud**: Conecta nativamente con BigQuery, Cloud Storage, y otros servicios

### 2. Flows y Recipes

- **Flow**: Contenedor que organiza todas las transformaciones de un proyecto. Representa el pipeline completo de datos.
- **Recipe**: Secuencia de transformaciones aplicadas a los datos. Cada transformación es un paso en el recipe.
- **Dataset**: Fuente de datos conectada al flow. Puede venir de BigQuery, Cloud Storage, o archivos locales.

### 3. Tipos de Transformaciones

**Limpieza de datos**:
- Filtrar filas basándose en condiciones
- Eliminar columnas innecesarias
- Manejar valores nulos
- Convertir tipos de datos

**Enriquecimiento de datos**:
- Concatenar columnas
- Crear columnas calculadas
- Aplicar funciones y fórmulas personalizadas
- Crear case statements para mapeo de valores

**Transformaciones avanzadas**:
- Agregaciones (GROUP BY)
- Joins entre datasets
- Pivots y unpivots
- Particionamiento de datos

### 4. Integración BigQuery ↔ Dataprep

**BigQuery como fuente**:
- Conexión directa desde Dataprep
- Lectura eficiente de grandes volúmenes de datos
- Soporte para queries SQL como fuente (no implementado en este lab pero disponible)

**BigQuery como destino**:
- Escritura directa desde Dataprep
- Opciones de escritura: crear nueva tabla, sobrescribir, o agregar
- Integración con particionamiento y clustering de BigQuery

### 5. Ejecución con Cloud Dataflow

**Proceso**:
- Dataprep compila las transformaciones a código Apache Beam
- Se ejecuta en Cloud Dataflow como pipeline distribuido
- Escala automáticamente según el volumen de datos
- Proporciona monitoreo y logging de la ejecución

**Ventajas**:
- Escalabilidad para grandes volúmenes de datos
- Procesamiento distribuido y paralelo
- Manejo automático de errores y reintentos
- Integración nativa con otros servicios de Google Cloud

### 6. Exploración Visual de Datos

**Herramientas de exploración**:
- Histogramas por columna
- Detección automática de tipos de datos
- Métricas de calidad de datos
- Visualización de distribuciones
- Identificación de outliers y valores anómalos

**Ventajas de la exploración visual**:
- Identificación rápida de problemas de calidad
- Comprensión intuitiva de la estructura de datos
- Sugerencias contextuales basadas en lo que se ve
- Facilita la toma de decisiones sobre transformaciones necesarias

## Aplicaciones Prácticas y Usos Futuros

### 1. Preparación de Datos para Machine Learning

**Aplicación**: Cloud Dataprep es ideal para preparar datos antes de entrenar modelos de ML:
- Limpiar y normalizar features
- Manejar valores faltantes
- Crear features derivadas
- Balancear datasets

**Uso futuro**: Al trabajar con modelos de ML en BigQuery ML o Vertex AI, podrás usar Dataprep para:
- Preparar datasets de entrenamiento
- Crear pipelines de feature engineering reutilizables
- Validar calidad de datos antes del entrenamiento

### 2. ETL para Data Warehousing

**Aplicación**: Dataprep puede servir como herramienta de ETL para alimentar data warehouses:
- Transformar datos de múltiples fuentes
- Aplicar reglas de negocio
- Crear vistas consolidadas
- Mantener historial y versionado de transformaciones

**Uso futuro**: Podrás:
- Construir pipelines ETL complejos sin escribir código
- Mantener documentación visual de transformaciones
- Colaborar con analistas de negocio en la definición de transformaciones

### 3. Limpieza de Datos de Google Analytics

**Aplicación**: Como se demostró en el lab, Dataprep es excelente para procesar datos de Google Analytics:
- Normalizar métricas
- Crear identificadores únicos
- Enriquecer con mapeos de valores
- Filtrar datos relevantes

**Uso futuro**: Podrás:
- Automatizar la preparación de datos de Analytics
- Crear reportes consistentes y reutilizables
- Combinar datos de Analytics con otras fuentes

### 4. Data Quality y Profiling

**Aplicación**: Las herramientas de exploración de Dataprep son valiosas para:
- Perfilar datasets desconocidos
- Identificar problemas de calidad antes del análisis
- Documentar características de datasets
- Validar datos después de importaciones

**Uso futuro**: Podrás:
- Realizar auditorías rápidas de calidad de datos
- Establecer reglas de validación visuales
- Crear dashboards de calidad de datos

### 5. Colaboración entre Equipos

**Aplicación**: La interfaz visual de Dataprep facilita:
- Colaboración entre data engineers y analistas de negocio
- Documentación visual de transformaciones
- Revisión de cambios en transformaciones
- Compartir conocimiento sobre datos

**Uso futuro**: Podrás:
- Incluir a analistas de negocio en el proceso de transformación
- Reducir la barrera entre "requisitos de negocio" e "implementación técnica"
- Crear flujos de trabajo colaborativos para preparación de datos

## Desafíos y Observaciones

### Desafío 1: Complejidad de la Interfaz Inicial

**Desafío**: La interfaz de Dataprep tiene muchas opciones y puede ser abrumadora al principio, especialmente el panel de sugerencias y las múltiples formas de realizar transformaciones.

**Solución**: El lab guía paso a paso, pero es útil explorar la interfaz sistemáticamente. Las sugerencias automáticas ayudan, pero entender los conceptos básicos primero facilita el uso.

**Aprendizaje**: La curva de aprendizaje es más suave si se entienden los conceptos de Flow → Recipe → Transformaciones. Una vez dominado el flujo básico, las funcionalidades avanzadas son más accesibles.

### Desafío 2: Limitación del Navegador

**Desafío**: Dataprep solo funciona en Google Chrome actualmente, lo cual puede ser una limitación si se prefiere otro navegador.

**Solución**: Asegurarse de usar Chrome antes de comenzar. Si surge un problema, verificar que se está usando la versión más reciente de Chrome.

**Aprendizaje**: Esta es una limitación técnica actual de la plataforma. Es importante estar al tanto de estos requisitos antes de comenzar proyectos con Dataprep.

### Desafío 3: Comprensión de Transformaciones Compuestas

**Desafío**: Transformaciones como case statements con múltiples condiciones pueden ser complejas de configurar, especialmente con muchos casos.

**Solución**: El lab proporciona una tabla clara de mapeos. En proyectos reales, es útil preparar esta documentación antes de implementar. La vista previa en tiempo real ayuda a verificar que los mapeos son correctos.

**Aprendizaje**: Las transformaciones complejas requieren planificación previa. Documentar los mapeos y reglas antes de implementarlos en Dataprep ahorra tiempo y reduce errores.

### Desafío 4: Tiempo de Ejecución de Jobs

**Desafío**: Los jobs pueden tardar varios minutos en ejecutarse, especialmente con grandes volúmenes de datos. Puede ser difícil saber si el job está progresando correctamente.

**Solución**: Dataprep proporciona barras de progreso y logs. Para este lab con 56k filas, el tiempo fue razonable. En producción, es importante considerar el tiempo de ejecución al diseñar pipelines.

**Aprendizaje**: Planificar tiempo suficiente para ejecución de jobs, especialmente durante desarrollo cuando se iteran sobre transformaciones. Usar muestras de datos más pequeñas durante desarrollo puede acelerar el ciclo de iteración.

### Desafío 5: Debugging de Transformaciones

**Desafío**: Cuando una transformación no produce el resultado esperado, puede ser difícil identificar qué salió mal, especialmente en recipes con muchos pasos.

**Solución**: La vista previa en tiempo real de Dataprep es muy útil. Revisar cada paso del recipe y verificar la vista previa después de cada transformación ayuda a identificar problemas tempranamente.

**Aprendizaje**: Desarrollar recipes incrementalmente, verificando cada transformación antes de agregar la siguiente. Esto facilita el debugging y mantiene el recipe más organizado.

## Reflexiones Finales

### Fortalezas del Lab

1. **Enfoque práctico completo**: El lab cubre todo el ciclo de vida de un pipeline de transformación, desde conexión hasta ejecución final
2. **Datos realistas**: Usar datos reales de Google Analytics hace el lab más relevante y aplicable
3. **Exploración guiada**: La sección de exploración enseña habilidades valiosas de análisis de calidad de datos
4. **Integración end-to-end**: Ver cómo BigQuery y Dataprep trabajan juntos proporciona contexto valioso

### Áreas de Mejora

1. **Tiempo limitado**: Con 75 minutos, puede ser difícil explorar todas las funcionalidades de Dataprep en profundidad
2. **Datos de muestra pequeños**: 56k filas es útil para aprendizaje, pero no refleja completamente cómo Dataprep maneja datasets muy grandes
3. **Casos de uso limitados**: El lab se enfoca en un caso de uso específico; más ejemplos de diferentes tipos de transformaciones serían útiles

### Comparación con Otros Enfoques

- **Vs. ETL programático (Python/SQL)**: Dataprep es más accesible para usuarios no técnicos pero puede ser menos flexible para lógica compleja
- **Vs. SQL directo**: Dataprep proporciona exploración visual y sugerencias que SQL no ofrece, pero SQL puede ser más rápido para transformaciones simples
- **Vs. Dataflow directo**: Dataprep simplifica la creación de pipelines Dataflow pero puede ocultar detalles de implementación

### Valor para Ingeniería de Datos

Este lab es especialmente valioso porque:

- **Demuestra herramientas no-code/low-code**: Importante en equipos mixtos
- **Muestra integración nativa de Google Cloud**: Cómo servicios trabajan juntos
- **Enseña exploración de datos**: Habilidad fundamental para data engineers
- **Introduce conceptos de data quality**: Crítico para pipelines robustos

## Decisiones y Próximos Pasos

### Decisiones Tomadas Durante el Lab

1. **Usar filtro de tipo PAGE**: Decisión para enfocar el análisis en visualizaciones de página relevantes
2. **Crear unique_session_id**: Decisión para resolver el problema de identificación única
3. **Aplicar case statement**: Decisión para mejorar la legibilidad de los datos para usuarios finales
4. **Normalizar revenue**: Decisión crítica para análisis preciso de ingresos
5. **Drop table on every run**: Decisión para facilitar re-ejecuciones durante desarrollo

### Próximos Pasos Recomendados

1. **Explorar transformaciones avanzadas**: 
   - Joins entre múltiples datasets
   - Agregaciones y agrupaciones
   - Pivots y transformaciones de estructura

2. **Trabajar con datasets más grandes**:
   - Probar con millones de filas
   - Observar cómo Dataflow escala
   - Optimizar transformaciones para performance

3. **Integrar con otros servicios**:
   - Cloud Storage como fuente/destino
   - Cloud Functions para validaciones personalizadas
   - Vertex AI para modelos de ML

4. **Automatizar pipelines**:
   - Programar ejecuciones regulares
   - Integrar con Cloud Scheduler
   - Implementar alertas y monitoreo

5. **Explorar otros labs relacionados**:
   - ETL Processing on Google Cloud Using Dataflow and BigQuery
   - Predict Visitor Purchases with a Classification Model in BigQuery ML
   - Working with Cloud Dataprep on Google Cloud (lab introductorio recomendado)

### Integración con Otros Aprendizajes

- **Con BigQuery**: Dataprep complementa BigQuery para preparación de datos antes de análisis
- **Con Dataflow**: Entender cómo Dataprep genera código Dataflow ayuda a comprender pipelines distribuidos
- **Con Data Quality**: Las técnicas de exploración son aplicables a validación de datos en general
- **Con DataOps**: Dataprep puede ser parte de pipelines CI/CD para datos

## Conclusiones

Este lab proporcionó una introducción sólida y práctica a Cloud Dataprep, demostrando cómo las herramientas visuales pueden simplificar la preparación de datos sin sacrificar funcionalidad. La experiencia end-to-end desde BigQuery hasta Dataprep y de vuelta a BigQuery ilustra cómo los servicios de Google Cloud se integran para crear soluciones completas de ingeniería de datos.

**Principales takeaways**:

- **Preparación visual de datos**: Las herramientas visuales pueden ser tan poderosas como el código para muchos casos de uso
- **Exploración antes de transformación**: Entender los datos primero es crucial para transformaciones efectivas
- **Integración nativa**: Los servicios de Google Cloud están diseñados para trabajar juntos sin fricción
- **Calidad de datos**: Las herramientas de exploración ayudan a identificar y resolver problemas de calidad tempranamente
- **Colaboración**: Las interfaces visuales facilitan la colaboración entre equipos técnicos y de negocio

**Valor para Ingeniería de Datos**:

- **Herramienta complementaria**: Dataprep no reemplaza programación pero es valiosa para ciertos casos de uso
- **Rápida prototipación**: Permite iterar rápidamente en transformaciones de datos
- **Documentación visual**: Los recipes sirven como documentación ejecutable de transformaciones
- **Onramp para no-programadores**: Permite que analistas de negocio participen directamente en preparación de datos
- **Preparación para ML**: Excelente para preparar datos antes de entrenar modelos

Este lab representa una pieza importante del ecosistema de herramientas de ingeniería de datos en Google Cloud, proporcionando una alternativa accesible a programación tradicional mientras mantiene la potencia y escalabilidad de servicios como Dataflow.

---

**Referencias:**
- [Google Cloud Skills Boost](https://www.skills.google/)
- [Lab GSP430: Creating a Data Transformation Pipeline with Cloud Dataprep](https://www.skills.google/focuses/4415?catalog_rank=%7B%22rank%22%3A6%2C%22num_filters%22%3A1%2C%22has_search%22%3Atrue%7D&parent=catalog&search_id=60910456)
- [Cloud Dataprep Documentation](https://cloud.google.com/dataprep/docs)
- [Alteryx Designer Cloud](https://www.alteryx.com/products/alteryx-designer-cloud)
- [BigQuery Documentation](https://cloud.google.com/bigquery/docs)
- [Cloud Dataflow Documentation](https://cloud.google.com/dataflow/docs)

