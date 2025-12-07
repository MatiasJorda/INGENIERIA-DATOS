---
title: "UT5 - Actividad 15: Pipelines ETL, DataOps y Orquestación con Prefect"
date: 2025
---

# UT5 - Actividad 15: Pipelines ETL, DataOps y Orquestación con Prefect

## Contexto

En esta actividad se trabajó con **Prefect**, una herramienta moderna de orquestación de pipelines de datos, diseñando e implementando un mini pipeline ETL completo. El objetivo fue investigar la documentación oficial para comprender los conceptos fundamentales (Tasks, Flows, DAGs implícitos, caching) y explorar funcionalidades avanzadas como retries, logging estructurado, validación de datos, y deployments. Se implementó un pipeline ETL para procesar datos de ventas de un e-commerce, aplicando principios de DataOps como observabilidad, reproducibilidad y CI/CD para datos.

**Fuente**: [Tarea 15 - Prefect](https://juanfkurucz.com/ucu-id/ut5/15-etl-dataops-prefect/)

## Objetivos

- Comprender los conceptos fundamentales de Prefect: Tasks, Flows, estados, y caching
- Investigar y aplicar funcionalidades avanzadas: retries, logging estructurado, validación
- Diseñar e implementar un pipeline ETL completo con Prefect
- Explorar deployments y scheduling para ejecución programada
- Conectar Prefect con principios de DataOps: observabilidad, reproducibilidad, CI/CD
- Comparar Prefect con alternativas como Apache Airflow y Dagster

## Actividades (con tiempos estimados)

| Actividad                          | Tiempo | Resultado esperado                                    |
|-----------------------------------|:------:|------------------------------------------------------|
| Investigación: Conceptos fundamentales |  15m   | Respuestas documentadas sobre Tasks, Flows, Results y Caching |
| Diseño conceptual del escenario   |  5m    | Escenario definido: e-commerce con roles identificados         |
| Implementación del pipeline base  |  20m   | Pipeline ETL funcionando: extract, transform, load            |
| Investigación: Funcionalidades avanzadas |  15m   | Retries, caching, logging, concurrencia investigados         |
| Investigación: Deployments y Scheduling |  10m   | Conceptos de deployment, work pools, scheduling comprendidos  |
| Extensión DataOps                 |  15m   | Validación con logging estructurado implementada             |
| Reflexión y conexión con DataOps  |  5m    | Reflexiones sobre observabilidad, reproducibilidad, CI/CD     |

## Desarrollo

### 1. Investigación: Conceptos Fundamentales de Prefect

#### 1.1 Tasks en Prefect

**¿Qué es una Task?** Una Task en Prefect es una unidad de trabajo individual que representa una operación específica dentro de un pipeline. Según la [documentación oficial](https://docs.prefect.io/latest/concepts/tasks/), una task es "a discrete piece of work that can be tracked and retried independently". Las tasks son funciones Python decoradas con `@task` que encapsulan lógica de trabajo específica (extraer datos, transformar, cargar).

**Evaluación diferida ("lazy evaluation")**: Las tasks no se ejecutan inmediatamente cuando se definen o llaman. Prefect construye primero una representación del grafo de dependencias. La ejecución real ocurre cuando se ejecuta el flow completo, momento en el cual Prefect resuelve las dependencias y ejecuta las tasks en el orden correcto. Esto permite optimización de ejecución y manejo de dependencias complejas.

**Estados de Tasks**: Prefect rastrea el estado de cada task durante su ciclo de vida:
- **PENDING**: Esperando ser ejecutada
- **RUNNING**: Actualmente en ejecución
- **COMPLETED**: Ejecutada exitosamente
- **FAILED**: Falló durante la ejecución
- **RETRYING**: Siendo reintentada después de un fallo
- **CANCELLED**: Cancelada antes de completarse
- **CRASHED**: Falló de manera inesperada

**Parámetros importantes del decorador `@task`**:
- `retries`: Número de reintentos si falla
- `retry_delay_seconds`: Tiempo de espera entre reintentos
- `timeout_seconds`: Tiempo máximo de ejecución
- `cache_key_fn`: Función personalizada para generar clave de caché
- `cache_expiration`: Duración de validez del caché
- `tags`: Etiquetas para organización y filtrado
- `log_prints`: Captura automática de prints como logs

#### 1.2 Flows en Prefect

**Diferencias entre Flow y Task**: Un Flow es un contenedor que orquesta y coordina la ejecución de múltiples Tasks. Mientras que una Task representa una unidad de trabajo individual, un Flow representa el flujo de trabajo completo que conecta múltiples Tasks. Necesitamos ambos porque las Tasks dividen el trabajo en componentes reutilizables, mientras que los Flows coordinan las Tasks y manejan dependencias automáticamente.

**Subflows**: Un subflow es un Flow que se puede llamar desde dentro de otro Flow. Es útil para modularidad, reutilización, organización lógica, debugging y testing. Por ejemplo, podrías tener un subflow `extract_and_validate()` que se use en múltiples pipelines ETL.

**DAGs implícitos**: Prefect maneja las dependencias entre tasks de manera automática e implícita. Cuando llamas a una task dentro de un flow y pasas el resultado de otra task como parámetro, Prefect detecta automáticamente esa dependencia y construye un DAG. Las tasks se ejecutan en el orden correcto según sus dependencias, no necesariamente en el orden del código.

#### 1.3 Results y Caching

**Result persistence**: La capacidad de Prefect de almacenar y recuperar resultados de tasks. Es importante para resiliencia (no re-ejecutar todo si falla a mitad de camino), debugging (inspeccionar resultados intermedios), reproducibilidad, eficiencia y auditoría.

**Caching**: Permite que las tasks eviten la re-ejecución si ya se ejecutaron previamente con los mismos parámetros. Prefect genera automáticamente una clave de caché basándose en los parámetros de entrada. Se configura con `cache_expiration` (timedelta) o `cache_key_fn` (función personalizada).

**cache_key_fn**: Función personalizada para generar la clave de caché. Útil cuando quieres cachear basándote solo en algunos parámetros (no todos), incluir factores externos, o invalidar el caché con lógica de negocio compleja.

### 2. Diseño Conceptual

**Escenario elegido**: Ventas de un e-commerce con datos de transacciones diarias

**Arquitectura del escenario**:
- **Business data owner**: Equipo de ventas/negocios que genera las transacciones diarias
- **Data engineers**: Equipo que construye y mantiene el pipeline ETL
- **Data consumers**: Analistas, científicos de datos y dashboards que consumen los datos procesados

**Tipo de pipeline**: Batch

**Justificación**: Las ventas se procesan diariamente en lotes. No necesitamos procesamiento en tiempo real para este caso de uso inicial. El procesamiento batch es más eficiente para análisis agregados y permite procesar grandes volúmenes de manera controlada, facilitando la validación y el manejo de errores.

### 3. Implementación del Pipeline Base

#### 3.1 Setup

Se configuró el entorno con Prefect y pandas, importando las librerías necesarias:
- `from prefect import flow, task`
- `pandas`, `numpy`, `datetime`

#### 3.2 Tasks Implementadas

**Task 1: Extract (`extract_data`)**
- Decorador: `@task(tags=["extract", "data-source"], log_prints=True)`
- Función: Simula la extracción de datos de ventas (100 registros con fechas, productos, cantidades, precios y regiones)
- Parámetros: Tags para organización, `log_prints` para capturar outputs

**Task 2: Transform (`transform_data`)**
- Decorador: `@task(tags=["transform", "data-processing"], log_prints=True)`
- Función: Calcula totales, categoriza por ticket size, agrega mes y día de la semana
- Transformaciones:
  - Cálculo de `total = cantidad * precio_unitario`
  - Categorización por ticket size (Bajo, Medio, Alto, Muy Alto)
  - Extracción de mes y día de la semana para análisis

**Task 3: Load (`load_data`)**
- Decorador: `@task(tags=["load", "data-output"], log_prints=True, retries=2, retry_delay_seconds=3)`
- Función: Guarda los datos transformados en CSV
- Características: Incluye retries para manejar posibles fallos en la escritura

#### 3.3 Flow Principal

**Flow: `etl_flow()`**
- Decorador: `@flow(name="ETL Pipeline Ventas", log_prints=True)`
- Orquestación: 
  1. Extrae datos con `extract_data()`
  2. Transforma datos con `transform_data(df_raw)` - depende de extract
  3. Carga datos con `load_data(df_transformed)` - depende de transform

Prefect detecta automáticamente las dependencias y construye el DAG: `extract_data → transform_data → load_data`

### 4. Investigación: Funcionalidades Avanzadas

#### 4.1 Retries y manejo de errores

Prefect permite configurar reintentos automáticos con `@task(retries=N, retry_delay_seconds=X)`. Útil para operaciones que pueden fallar temporalmente (APIs, conexiones de red, I/O). Ya implementado en `load_data()` con 2 reintentos y 3 segundos de delay.

#### 4.2 Caching de resultados

El caching evita re-ejecución de tasks costosas cuando los parámetros no cambian. Se configura con `cache_expiration` (timedelta) o `cache_key_fn` (función personalizada). Útil para extracciones de APIs o procesamiento pesado.

#### 4.3 Logging personalizado

Se puede usar `get_run_logger()` para logging estructurado o `log_prints=True` para capturar prints automáticamente. Ya implementado en todas las tasks. Ventajas: logs centralizados, niveles de log, integración con UI de Prefect.

#### 4.4 Concurrencia y paralelismo

Prefect soporta ejecución concurrente con `ConcurrentTaskRunner()` como parámetro del flow. Las tasks independientes pueden ejecutarse en paralelo, reduciendo el tiempo total. Ejemplo: procesar múltiples regiones en paralelo usando `.submit()`.

### 5. Investigación: Deployments y Scheduling

#### 5.1 Conceptos de Deployment

**Deployment**: Configuración que permite ejecutar un Flow de manera programada o bajo demanda. Conecta un Flow con configuración de ejecución (scheduling, work pool, parámetros). Difiere de un Flow: el Flow es el código, el Deployment es la "instalación" en Prefect Cloud/Server.

**Work Pool**: Grupo de workers que pueden ejecutar flows. Permite organizar dónde y cómo se ejecutan los flows (local, servidor, cloud). Abstracción para manejar infraestructura de ejecución.

**Worker**: Proceso que ejecuta flows desde un Work Pool. Los Workers se conectan a un Work Pool y "toman" trabajos para ejecutarlos. Relación: Deployment → Work Pool → Worker → Ejecución del Flow.

#### 5.2 Scheduling

**Tipos de schedules**:
- **CronSchedule**: Sintaxis cron para horarios recurrentes (ej: `"0 6 * * *"` - todos los días a las 6 AM)
- **IntervalSchedule**: Intervalos fijos de tiempo (ej: `timedelta(hours=1)`)
- **RRuleSchedule**: Reglas RFC 5545 para horarios complejos (útil para horarios de negocio, excluir fines de semana)

**Ejemplo cron**: `"0 6 * * *"` = todos los días a las 6 AM (formato: minuto hora día_mes mes día_semana)

**RRuleSchedule**: Más flexible que cron para casos de negocio complejos, permite definir horarios como "cada lunes y miércoles a las 9 AM, excepto feriados".

### 6. Extensión DataOps: Validación con Logging Estructurado

Se implementó la **Opción A - Validación con logging estructurado**, fundamental para calidad de datos en pipelines ETL.

**Task de validación: `validate_data()`**
- Decorador: `@task(retries=1, retry_delay_seconds=2, tags=["validation", "data-quality"], log_prints=True)`
- Funcionalidad: Valida calidad de datos usando `get_run_logger()` para logging estructurado

**Validaciones implementadas**:
1. **DataFrame no vacío**: Verifica que haya registros para procesar
2. **Valores nulos**: Detecta y registra como warning (no crítico, pero informativo)
3. **Columnas requeridas**: Verifica que existan las columnas necesarias (`fecha`, `producto`, `cantidad`, `precio_unitario`, `total`)
4. **Tipos de datos**: Valida que los tipos sean correctos (ej: `total` debe ser numérico)
5. **Valores negativos**: Detecta cantidades negativas como warning

**Niveles de log utilizados**:
- `logger.info()`: Para información general y validaciones exitosas
- `logger.warning()`: Para problemas no críticos (nulos, valores negativos)
- `logger.error()`: Para errores críticos que detienen el pipeline

**Flow actualizado**: `etl_flow_with_validation()` incluye la validación entre extract y transform, asegurando que solo se procesen datos de calidad.

### 7. Reflexión y Conexión con DataOps

#### 7.1 Conceptos de Prefect y DataOps

**Observabilidad**: Prefect implementa observabilidad mediante:
- Logging estructurado automático
- Estados de ejecución claros para cada task
- UI centralizada para monitoreo en tiempo real
- Métricas de tiempo de ejecución, tasa de éxito, frecuencias de fallo
- Notificaciones y alertas configurables

Esto permite visibilidad completa sobre los pipelines, facilitando debugging y detección temprana de problemas.

**Reproducibilidad**: El caching ayuda a la reproducibilidad porque:
- Asegura resultados consistentes con las mismas entradas
- Mantiene historial de ejecuciones cacheadas
- Permite puntos de recuperación si un pipeline falla
- Facilita validación comparando resultados actuales vs cacheados

Fundamental en DataOps para garantizar pipelines que produzcan resultados consistentes.

**CI/CD para datos**: Los Deployments conectan con CI/CD porque:
- Permiten versionado y despliegue de diferentes versiones de flows
- Schedules automatizan ejecución similar a pipelines de CI/CD
- Soporte para diferentes ambientes (desarrollo, staging, producción)
- Rollback rápido cambiando versión activa
- Integración con sistemas de CI/CD (GitHub Actions, GitLab CI)

Esto permite aplicar prácticas de DevOps a pipelines de datos, facilitando despliegues seguros y controlados.

#### 7.2 Comparación con Alternativas

**Prefect vs Apache Airflow**:

1. **Filosofía de diseño**: Airflow usa DAGs explícitos definidos con operadores, mientras que Prefect usa DAGs implícitos donde las dependencias se detectan automáticamente. Prefect es más "Python puro" y menos verboso.

2. **Curva de aprendizaje**: Prefect tiene curva más suave - solo decoras funciones con `@task` y `@flow`. Airflow requiere entender DAGs, Operators, XComs, etc., siendo más complejo para empezar.

3. **Orientación**: Prefect está diseñado más moderno y orientado a Python nativo, mientras que Airflow fue diseñado originalmente como sistema de workflow basado en DAGs.

**Prefect vs Dagster**:

- **Enfoque**: Dagster está más orientado a "assets" (datos y modelos) como primitivos, mientras que Prefect está más orientado a "tasks" y "flows"
- **Integración**: Dagster tiene integración más profunda con bibliotecas de datos (pandas, pyspark)
- **Software-defined assets**: Dagster permite definir pipelines basándose en qué datos necesitas, no solo en qué tareas ejecutar
- **Ecosistema**: Prefect tiene ecosistema más amplio y comunidad más grande, mientras que Dagster está más enfocado en equipos grandes

Ambas son alternativas modernas a Airflow, pero Dagster es más "data-centric" mientras que Prefect es más "workflow-centric".

## Evidencias

- Notebook principal:

  [Abrir en Colab](https://colab.research.google.com/github/MatiasJorda/INGENIERIA-DATOS/blob/main/docs/portfolio/UT5/Notebooks/quince.ipynb) ·
  
  [Ver en GitHub](https://github.com/MatiasJorda/INGENIERIA-DATOS/blob/main/docs/portfolio/UT5/Notebooks/quince.ipynb) ·
  
  [Nbviewer (mirror)](https://nbviewer.org/github/MatiasJorda/INGENIERIA-DATOS/blob/main/docs/portfolio/UT5/Notebooks/quince.ipynb)
  
### Resultados

En este proyecto trabajamos con Prefect para diseñar e implementar un pipeline ETL completo. El enfoque principal fue:

1. **Investigación profunda** de la documentación oficial para entender conceptos fundamentales
2. **Implementación práctica** de un pipeline ETL para ventas de e-commerce
3. **Extensión DataOps** con validación de calidad de datos usando logging estructurado
4. **Reflexión** sobre cómo Prefect habilita principios de DataOps

---

## Aprendizajes Clave

### Conceptos Fundamentales

- **Tasks y Flows**: Entendimos la diferencia entre unidades de trabajo (tasks) y orquestación (flows), y cómo Prefect construye DAGs automáticamente
- **Estados y observabilidad**: Cada task tiene estados rastreables que permiten monitorear el pipeline en tiempo real
- **Caching**: Permite optimizar ejecuciones y garantizar reproducibilidad

### Funcionalidades Avanzadas

- **Retries**: Mecanismo automático para manejar fallos temporales
- **Logging estructurado**: Fundamental para debugging y monitoreo en producción
- **Validación de datos**: Crítico para asegurar calidad antes de procesar

### DataOps

- **Observabilidad**: Prefect proporciona visibilidad completa sobre pipelines
- **Reproducibilidad**: El caching y result persistence garantizan resultados consistentes
- **CI/CD**: Los deployments permiten aplicar prácticas de DevOps a pipelines de datos

### Comparación con Alternativas

- Prefect tiene curva de aprendizaje más suave que Airflow
- Dagster tiene enfoque más orientado a assets, Prefect más workflow-centric
- Cada herramienta tiene su lugar según las necesidades del equipo

## Desafíos y Soluciones

### Desafío 1: Entender DAGs Implícitos

**Desafío**: Al principio era confuso entender cómo Prefect detecta dependencias automáticamente sin definirlas explícitamente.

**Solución**: Después de investigar y experimentar, quedó claro que Prefect detecta dependencias cuando pasas el resultado de una task como parámetro de otra. Esto simplifica mucho el código comparado con Airflow.

### Desafío 2: Configurar Validación con Logging

**Desafío**: Implementar validación efectiva que registre información útil sin saturar los logs.

**Solución**: Usamos diferentes niveles de log (info, warning, error) y estructuramos las validaciones para que los logs sean informativos pero concisos. Esto permite monitorear calidad de datos sin ruido excesivo.

### Desafío 3: Entender Deployments vs Flows

**Desafío**: Diferenciar entre un Flow (código) y un Deployment (configuración de ejecución).

**Solución**: Investigamos la documentación y entendimos que el Flow es el código Python, mientras que el Deployment es la "instalación" del Flow en Prefect con configuración específica (scheduling, work pool, etc.).

## Decisiones y Próximos Pasos

### Decisiones Tomadas

1. **Escenario elegido**: E-commerce con ventas diarias - permite demostrar pipeline ETL realista pero simple
2. **Tipo batch**: Apropiado para el caso de uso y más fácil de implementar inicialmente
3. **Extensión DataOps**: Opción A (Validación) - más fundamental para calidad de datos que caching o concurrencia para este ejemplo
4. **Logging estructurado**: Implementado en todas las tasks para observabilidad completa

### Próximos Pasos

1. **Explorar Prefect Cloud/Server**: Configurar servidor local o usar Cloud para monitoreo visual
2. **Implementar más validaciones**: Expandir validación de datos con Great Expectations o Pydantic
3. **Agregar concurrencia**: Implementar procesamiento paralelo por región para mejorar performance
4. **Deployment real**: Desplegar el pipeline con scheduling real en Prefect Cloud/Server
5. **Integración con CI/CD**: Conectar con GitHub Actions para despliegue automático
6. **Result persistence avanzado**: Configurar backend de resultados (S3, GCS) para pipelines en producción
7. **Monitoreo y alertas**: Configurar notificaciones cuando los pipelines fallen

---

_"La mejor forma de aprender una herramienta es leer su documentación oficial. Los tutoriales te dan el 'qué', la documentación te da el 'por qué' y el 'cómo'."_

**Referencias:**
- [Documentación oficial de Prefect](https://docs.prefect.io/)
- [Prefect Concepts Overview](https://docs.prefect.io/latest/concepts/)
- [Prefect Tasks Documentation](https://docs.prefect.io/latest/concepts/tasks/)
- [Prefect Flows Documentation](https://docs.prefect.io/latest/concepts/flows/)
- [Prefect Caching Documentation](https://docs.prefect.io/latest/concepts/tasks/#caching)
- [Prefect Deployments Documentation](https://docs.prefect.io/latest/concepts/deployments/)

