---
title: "UT5 - Google Cloud Skills: A Tour of Google Cloud Hands-on Labs"
date: 2025
---

# UT5 - Google Cloud Skills: A Tour of Google Cloud Hands-on Labs

## Contexto

En esta actividad se completó el lab introductorio **"A Tour of Google Cloud Hands-on Labs" (GSP282)** de Google Cloud Skills Boost. Este lab fue diseñado como punto de entrada para principiantes en Google Cloud Platform, proporcionando una experiencia práctica con la consola de Google Cloud y sus características fundamentales. El objetivo fue familiarizarse con la plataforma Qwiklabs, entender la estructura de labs de Google Cloud, y realizar tareas básicas de administración en la Cloud Console.

**Fuente**: [Google Cloud Skills Boost - Lab GSP282](https://www.skills.google/focuses/2794?parent=catalog)

**Nivel**: Introductorio  
**Duración**: 45 minutos  
**Costo**: Gratis

## Objetivos

- Familiarizarse con la interfaz de Google Cloud Skills Boost y Qwiklabs
- Acceder a la Cloud Console usando credenciales temporales del lab
- Explorar y entender el concepto de proyectos en Google Cloud
- Gestionar roles y permisos básicos usando Cloud IAM
- Habilitar APIs y servicios en un proyecto de Google Cloud
- Comprender la estructura y componentes comunes de los labs de Google Cloud

## Actividades (con tiempos estimados)

| Actividad                          | Tiempo | Resultado esperado                                    |
|-----------------------------------|:------:|------------------------------------------------------|
| Acceso a Cloud Console             |  10m   | Sesión iniciada en Cloud Console con credenciales temporales |
| Exploración de proyectos           |  10m   | Comprensión del concepto de Project ID y organización de recursos |
| Revisión de roles y permisos (IAM) |  15m   | Entendimiento de roles básicos (Viewer, Editor, Owner) y gestión de acceso |
| Habilitación de APIs               |  10m   | Dialogflow API habilitada, conocimiento de API Library |

## Desarrollo

### 1. Fundamentos de Labs en Google Cloud Skills Boost

#### Componentes comunes de los labs

Antes de comenzar con las tareas prácticas, se aprendió sobre los componentes estándar de todos los labs en la plataforma:

- **Start Lab (botón)**: Crea un ambiente temporal de Google Cloud con todos los servicios y credenciales necesarios habilitados. Inicia un temporizador de cuenta regresiva.

- **Créditos**: El costo de un lab. Generalmente 1 crédito equivale a 1 dólar estadounidense. Algunos labs introductorios (como este) son gratuitos. Los labs más especializados cuestan más porque involucran tareas de computación más pesadas.

- **Tiempo**: Especifica la cantidad de tiempo disponible para completar el lab. Cuando el temporizador llega a 00:00:00, el ambiente temporal y los recursos son eliminados.

- **Score (Puntuación)**: Muchos labs incluyen un sistema de puntuación llamado "activity tracking" que verifica la finalización de pasos específicos en orden. Solo completando todos los pasos se puede recibir crédito de finalización.

#### Plataforma Qwiklabs

Qwiklabs es la plataforma tecnológica donde se ejecutan los labs y cursos de Google Cloud Skills Boost. Permite descubrir rutas de aprendizaje, construir habilidades en la nube, rastrear progreso de actividad y validar conocimiento con badges.

### 2. Acceso a Cloud Console

#### Panel de detalles del lab

Una vez iniciado el lab, el panel **Lab details** contiene información crítica:

- **Open Google Cloud console**: Botón que abre la consola web y centro de desarrollo de Google Cloud.
- **Username y Password**: Credenciales temporales que representan una identidad en Cloud IAM. Estas credenciales solo funcionan durante la duración del lab.
- **Project ID**: Identificador único que vincula recursos y APIs de Google Cloud a un proyecto específico. Los Project IDs son únicos globalmente en Google Cloud.

#### Proceso de inicio de sesión

El proceso implica:
1. Hacer clic en "Open Google Cloud console"
2. Iniciar sesión con las credenciales temporales (formato: `student-xx-xxxxxx@qwiklabs.net`)
3. Aceptar términos y condiciones
4. Acceder a la interfaz principal de Cloud Console

**Importante**: Se debe usar siempre una ventana de navegación privada (Incognito/Private) para evitar conflictos entre cuentas personales y las credenciales temporales del lab.

### 3. Exploración de Proyectos en Google Cloud

#### Concepto de proyecto

Un **Google Cloud Project** es una entidad organizacional para recursos de Google Cloud. Contiene:
- Recursos y servicios (máquinas virtuales, bases de datos, redes)
- Configuraciones y permisos
- Reglas de seguridad y control de acceso

#### Project ID vs Project Name

- **Project Name**: Nombre legible por humanos, puede cambiar
- **Project ID**: Identificador único e inmutable, usado globalmente para identificar el proyecto

**Conceptos erróneos comunes aclarados**:
- Los Project IDs no se pueden cambiar después de la creación
- Un Project ID es único globalmente (no puede haber dos iguales)
- Los proyectos son entidades organizacionales, no servicios individuales

#### Navegación en Cloud Console

Se exploró el menú de navegación para identificar tipos de servicios de Google Cloud:
- **Compute**: Compute Engine, Cloud Functions, App Engine
- **Storage**: Cloud Storage, Cloud SQL, BigQuery
- **Networking**: VPC, Load Balancing, Cloud CDN
- **Security**: IAM, Cloud Identity
- **APIs & Services**: API Library, Service Accounts
- Y muchos más servicios categorizados

### 4. Revisión y Modificación de Roles y Permisos (IAM)

#### Cloud Identity and Access Management (Cloud IAM)

Cloud IAM es el servicio que controla quién tiene qué tipo de acceso a qué recursos. Permite gestionar:
- **Principals** (Usuarios, grupos, service accounts): Entidades que pueden acceder a recursos
- **Roles**: Colecciones de permisos
- **Permissions**: Acciones específicas que pueden realizarse en recursos

#### Roles básicos

Google Cloud ofrece tres roles básicos a nivel de proyecto:

| Rol | Permisos |
|-----|----------|
| **roles/viewer** | Permisos de solo lectura para acciones que no afectan el estado, como ver (pero no modificar) recursos o datos existentes. |
| **roles/editor** | Todos los permisos de viewer, más permisos para acciones que modifican el estado, como cambiar recursos existentes. |
| **roles/owner** | Todos los permisos de editor, más permisos para: gestionar roles y permisos para un proyecto y todos los recursos dentro del proyecto; configurar facturación para un proyecto. |

#### Tarea práctica: Otorgar un rol IAM

Se realizó la siguiente tarea:
1. Acceder a **IAM & Admin > IAM** desde el menú de navegación
2. Hacer clic en **Grant access** (Otorgar acceso)
3. Ingresar un identificador de principal (usuario)
4. Seleccionar el rol **Viewer** desde el menú desplegable
5. Guardar los cambios
6. Verificar que el principal y el rol correspondiente aparezcan listados en la página IAM

**Aprendizaje clave**: Como Editor, puedes crear, modificar y eliminar recursos de Google Cloud, pero no puedes agregar o eliminar miembros de proyectos de Google Cloud directamente (aunque puedes otorgar roles existentes).

### 5. Habilitación de APIs y Servicios

#### API Library de Google Cloud

Google Cloud ofrece más de 200 APIs que cubren áreas desde administración empresarial hasta aprendizaje automático. Todas se integran fácilmente con proyectos y aplicaciones de Google Cloud.

**Características de las APIs**:
- **APIs como Interfaces de Programación de Aplicaciones**: Pueden llamarse directamente o vía librerías de cliente
- **Diseño orientado a recursos**: Las Cloud APIs usan principios de diseño orientado a recursos
- **Monitoreo integrado**: Proporcionan información detallada sobre uso del proyecto, incluyendo niveles de tráfico, tasas de error y latencias

#### Tarea práctica: Habilitar Dialogflow API

Se realizó la siguiente tarea:
1. Acceder a **APIs & Services > Library** desde el menú de navegación
2. Buscar "Dialogflow" en la barra de búsqueda de APIs
3. Seleccionar **Dialogflow API** de los resultados
4. Revisar la descripción: "Permite construir aplicaciones conversacionales ricas (ej., para Google Assistant) sin tener que entender el esquema de aprendizaje automático y procesamiento de lenguaje natural subyacente"
5. Hacer clic en **Enable** (Habilitar)
6. Verificar que la API esté habilitada volviendo a la página principal
7. Explorar la opción "Try this API" para ver documentación y ejemplos

**Observación importante**: Cuando un lab proporciona un nuevo proyecto de Google Cloud para una instancia de lab, habilita automáticamente muchas APIs para que puedas comenzar rápidamente. Cuando creas tus propios proyectos fuera del ambiente de lab, tendrás que habilitar las APIs manualmente.

#### Categorías de APIs

La API Library está organizada en categorías:
- Machine Learning
- Storage
- Compute
- Networking
- Security & Identity
- Big Data
- IoT
- Y muchas más

## Conceptos Clave Aprendidos

### 1. Google Cloud Platform (GCP)

**Definición**: Google Cloud es una suite de servicios en la nube alojados en la infraestructura de Google. Ofrece una amplia variedad de servicios y APIs que pueden integrarse con cualquier aplicación o proyecto de computación en la nube, desde personal hasta nivel empresarial.

**Servicios principales**:
- Computación y almacenamiento
- Análisis de datos
- Aprendizaje automático
- Networking

### 2. Proyectos de Google Cloud

- **Propósito**: Organizar y agrupar recursos relacionados
- **Project ID**: Identificador único e inmutable a nivel global
- **Aislamiento**: Los proyectos aíslan recursos y configuraciones
- **Facturación**: Los proyectos están vinculados a cuentas de facturación

### 3. Cloud IAM (Identity and Access Management)

- **Modelo de seguridad**: Control granular de acceso a recursos
- **Principio de menor privilegio**: Otorgar solo los permisos necesarios
- **Roles básicos vs roles personalizados**: Roles predefinidos vs roles específicos del servicio
- **Permisos heredados**: Los permisos se pueden heredar a nivel de proyecto u organización

### 4. APIs y Servicios

- **APIs como servicios**: Las APIs son interfaces para acceder a funcionalidades de Google Cloud
- **Habilitación explícita**: Las APIs deben habilitarse antes de usarse
- **Monitoreo**: Cada API proporciona métricas de uso, errores y latencia
- **Client libraries**: Librerías de cliente disponibles para múltiples lenguajes de programación

### 5. Cloud Console

- **Interfaz web**: Portal central para acceder y gestionar servicios de Google Cloud
- **Navegación por servicios**: Menú organizado por categorías de servicios
- **Gestión unificada**: Un solo lugar para gestionar todos los recursos de un proyecto

## Aplicaciones Prácticas y Usos Futuros

### 1. Administración de Infraestructura

**Aplicación**: La comprensión de proyectos y IAM es fundamental para:
- Organizar recursos en proyectos separados por ambiente (desarrollo, staging, producción)
- Implementar separación de responsabilidades entre equipos
- Gestionar costos por proyecto
- Aplicar políticas de seguridad específicas por proyecto

**Uso futuro**: Al trabajar con múltiples proyectos de Google Cloud, podrás:
- Crear estructuras organizacionales claras
- Implementar controles de acceso apropiados
- Auditar y monitorear el uso de recursos

### 2. Desarrollo de Aplicaciones

**Aplicación**: La habilitación y uso de APIs es esencial para:
- Integrar servicios de Google Cloud en aplicaciones
- Construir aplicaciones que aprovechen capacidades como Dialogflow, Vision API, Translation API, etc.
- Acceder a datos y funcionalidades a través de interfaces estándar

**Uso futuro**: Cuando desarrolles aplicaciones, podrás:
- Identificar y habilitar las APIs necesarias
- Usar client libraries para integrar funcionalidades fácilmente
- Monitorear el uso de APIs para optimizar costos y rendimiento

### 3. Gestión de Seguridad

**Aplicación**: Cloud IAM es crucial para:
- Implementar controles de acceso basados en roles (RBAC)
- Seguir el principio de menor privilegio
- Auditar quién tiene acceso a qué recursos
- Gestionar acceso programático a través de service accounts

**Uso futuro**: En proyectos de producción, podrás:
- Diseñar estructuras de roles apropiadas
- Implementar políticas de seguridad consistentes
- Realizar auditorías de acceso regularmente
- Gestionar permisos de manera escalable

### 4. Aprendizaje Continuo

**Aplicación**: La plataforma Google Cloud Skills Boost permite:
- Aprender nuevas tecnologías mediante labs prácticos
- Validar conocimiento con badges y certificaciones
- Seguir rutas de aprendizaje estructuradas
- Mantenerse actualizado con nuevas funcionalidades

**Uso futuro**: Puedes:
- Completar labs especializados en áreas de interés (ML, Data Engineering, Security)
- Prepararte para certificaciones de Google Cloud
- Explorar nuevas APIs y servicios a medida que se lanzan
- Construir un portafolio de habilidades verificadas

## Desafíos y Observaciones

### Desafío 1: Navegación en Cloud Console

**Desafío**: La Cloud Console tiene muchas opciones y servicios, lo que puede ser abrumador inicialmente.

**Solución**: El lab proporciona una introducción estructurada que guía a través de las áreas principales. La práctica de navegación ayuda a familiarizarse con la organización de servicios.

**Aprendizaje**: Con el tiempo y la práctica, la navegación se vuelve más intuitiva. Es útil recordar que los servicios están organizados por categorías lógicas.

### Desafío 2: Comprensión de IAM

**Desafío**: Los conceptos de IAM (principals, roles, permisos) pueden ser abstractos al principio.

**Solución**: El lab presenta los conceptos de manera práctica, permitiendo otorgar roles y ver los cambios inmediatamente. La tabla de roles básicos ayuda a entender las diferencias.

**Aprendizaje**: IAM es más fácil de entender mediante práctica. Es útil empezar con roles básicos y luego explorar roles personalizados.

### Desafío 3: APIs como Concepto

**Desafío**: Entender qué significa "habilitar una API" puede ser confuso si no se tiene experiencia previa.

**Solución**: El lab explica que habilitar una API activa el servicio en tu proyecto. La exploración de la API Library muestra la amplia gama de capacidades disponibles.

**Aprendizaje**: Las APIs en Google Cloud son interfaces para acceder a servicios. Habilitar una API es como "encender" ese servicio para tu proyecto.

## Reflexiones Finales

### Fortalezas del Lab

1. **Enfoque introductorio bien estructurado**: El lab es perfecto para principiantes, proporcionando contexto antes de cada tarea
2. **Ambiente temporal seguro**: Las credenciales temporales eliminan el riesgo de cambios accidentales en proyectos personales
3. **Aprendizaje práctico**: La experiencia hands-on es más efectiva que solo leer documentación
4. **Activity tracking**: El sistema de puntuación motiva la finalización completa de todas las tareas

### Áreas de Mejora

1. **Tiempo limitado**: Aunque se da tiempo suficiente, el ambiente temporal puede ser estresante para principiantes
2. **Navegación inicial**: Podría beneficiarse de un tour guiado más interactivo de la Cloud Console

### Comparación con Otros Enfoques de Aprendizaje

- **Vs. Tutoriales en video**: Los labs proporcionan experiencia práctica real, mientras que los videos son más pasivos
- **Vs. Documentación**: Los labs guían paso a paso, mientras que la documentación requiere más auto-dirigimiento
- **Vs. Proyectos propios**: Los labs eliminan la configuración inicial y permiten enfocarse en conceptos específicos

## Decisiones y Próximos Pasos

### Decisiones Tomadas

1. **Uso de navegación privada**: Decisión crítica para evitar conflictos de credenciales
2. **Exploración sistemática**: Revisar cada sección del menú para entender la organización
3. **Lectura completa de descripciones**: Leer descripciones de APIs y servicios para comprender su propósito

### Próximos Pasos Recomendados

1. **Get Started with Cloud Shell and gcloud**: Aprender a usar la línea de comandos para interactuar con Google Cloud
2. **Create a Virtual Machine**: Primeros pasos prácticos con infraestructura
3. **Compute Engine: Qwik Start**: Profundizar en servicios de computación
4. **Explorar API Library**: Revisar APIs disponibles para identificar oportunidades
5. **Estudiar para certificaciones**: Considerar preparación para Associate Cloud Engineer o Professional Cloud Architect

### Integración con Otros Aprendizajes

- **Con DataOps**: La gestión de proyectos y IAM es fundamental para pipelines de datos seguros
- **Con MLOps**: Las APIs de ML requieren habilitación y gestión similar
- **Con DevOps**: IAM y organización de proyectos son esenciales para CI/CD en la nube

---

## Conclusiones

Este lab introductorio proporcionó una base sólida para trabajar con Google Cloud Platform. Los conceptos aprendidos—proyectos, IAM, APIs—son fundamentales para cualquier trabajo posterior en GCP. La experiencia práctica con la Cloud Console eliminó barreras iniciales y proporcionó confianza para explorar servicios más avanzados.

**Principales takeaways**:
- Google Cloud organiza recursos en proyectos únicos e identificables globalmente
- IAM proporciona control granular de acceso siguiendo el principio de menor privilegio
- Las APIs deben habilitarse explícitamente antes de usarse
- La Cloud Console es el punto central para gestionar todos los recursos y servicios
- La práctica hands-on es esencial para entender realmente cómo funcionan estos conceptos

**Valor para Ingeniería de Datos**:
- Organización de proyectos facilita separar ambientes de datos (dev/staging/prod)
- IAM es crucial para pipelines de datos seguros y acceso controlado
- APIs habilitan integración con servicios de Google Cloud (BigQuery, Dataflow, etc.)
- Base sólida para labs más avanzados sobre data engineering y analytics

Este lab representa el primer paso en un viaje de aprendizaje continuo con Google Cloud, estableciendo fundamentos que se aplicarán en proyectos de ingeniería de datos más complejos.

---

**Referencias:**
- [Google Cloud Skills Boost](https://www.skills.google/)
- [Lab GSP282: A Tour of Google Cloud Hands-on Labs](https://www.skills.google/focuses/2794?parent=catalog)
- [Google Cloud Console Documentation](https://cloud.google.com/docs/overview)
- [Cloud IAM Documentation](https://cloud.google.com/iam/docs)
- [APIs Explorer Directory](https://developers.google.com/apis-explorer)

