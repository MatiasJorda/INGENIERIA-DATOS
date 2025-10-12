---
title: "UT2 - Actividad 7: Detección y Corrección de Sesgo con Fairlearn"
date: 20/08/2025
---

# UT2 - Actividad 7: Detección y Corrección de Sesgo con Fairlearn

## Contexto

En esta actividad se abordó uno de los desafíos más críticos en machine learning moderno: la detección y mitigación de sesgos algorítmicos que pueden perpetuar o amplificar desigualdades sociales existentes. Utilizando la librería **Fairlearn** y datasets con sesgos históricos conocidos (Boston Housing y Titanic), se exploraron técnicas para identificar, cuantificar y corregir sesgos en modelos predictivos, así como las consideraciones éticas fundamentales al trabajar con datos sensibles.

## Objetivos

- Comprender los diferentes tipos de sesgo en machine learning (histórico, sistemático, de representación).
- Aplicar métricas de fairness para detectar disparidades entre grupos protegidos.
- Utilizar Fairlearn para mitigar sesgos mediante técnicas de reducción de restricciones.
- Evaluar trade-offs entre rendimiento predictivo y equidad algorítmica.
- Desarrollar un marco ético para la toma de decisiones sobre uso de variables sensibles.
- Documentar decisiones éticas de manera transparente y justificada.

## Actividades (con tiempos estimados)

| Actividad                          | Tiempo | Resultado esperado                                 |
|-----------------------------------|:------:|----------------------------------------------------|
| Análisis del dataset Boston Housing |  45m   | Identificación de sesgo racial histórico           |
| Reflexión ética sobre variable B   |  30m   | Marco de decisión ética documentado                |
| Análisis del dataset Titanic       |  30m   | Detección de sesgo de género y clase               |
| Aplicación de Fairlearn            |  1h    | Modelo con sesgo mitigado                          |
| Evaluación de trade-offs           |  45m   | Análisis de pérdida de performance vs ganancia de fairness |
| Desarrollo de framework ético      |  1h    | Criterios generalizables para decisiones éticas    |

## Desarrollo

### Parte I: Boston Housing - Sesgo Racial Histórico

**1. Contexto del Dataset**

El dataset Boston Housing, originalmente publicado en 1978, contiene datos sobre precios de viviendas en Boston. Incluye una variable particularmente problemática: **B** (codificada como 1000(Bk - 0.63)²), donde Bk representa la proporción de residentes afroamericanos por zona censal.

**Características clave del dataset:**
- 506 observaciones de vecindarios de Boston
- 13 variables predictoras + precio (MEDV)
- Variable B con correlación significativa con precios: 0.333
- Removido de sklearn en 2020 por razones éticas

**2. Análisis de Sesgo Racial**

Se crearon dos grupos basados en la mediana de la proporción racial:
- **Alta proporción afroamericana**: 253 observaciones
- **Baja proporción afroamericana**: 253 observaciones

**Distribución de precios por grupo:**

| Grupo Racial | Media ($k) | Mediana ($k) | Desv. Est. |
|--------------|------------|--------------|------------|
| Alta prop. afroam | 22.81 | 22.0 | 7.99 |
| Baja prop. afroam | 22.25 | 20.4 | 10.27 |

**Brecha de precios detectada:**
- Diferencia promedio: **-$0.56k** (-2.4%)
- Interpretación: Los precios medios son levemente menores en áreas con baja proporción afroamericana, pero la distribución completa revela patrones más complejos

**3. Reflexión Ética sobre Variable B**

Se desarrolló un análisis crítico estructurado en cuatro dimensiones:

**a) Contexto Histórico**
- Variable diseñada en 1978 en contexto de redlining y segregación racial
- Codifica explícitamente composición racial como predictor de valor inmobiliario
- **Pregunta crítica**: ¿Es ético usar esta variable en 2025?
- **Respuesta consensuada**: **NO** - perpetúa prácticas discriminatorias históricas

**b) Dilema de Utilidad**
- La variable B es estadísticamente predictiva (mejora métricas del modelo)
- Pero codifica y normaliza discriminación racial estructural
- **Pregunta crítica**: ¿Cuándo la utilidad justifica el sesgo?
- **Respuesta**: Muy raramente; solo en contextos académicos para estudiar el sesgo, nunca en producción

**c) Responsabilidad Profesional**
- Sklearn removió el dataset por razones éticas
- Uso educativo para aprender sobre detección de sesgo
- **Pregunta crítica**: ¿Cuál es nuestra responsabilidad como data scientists?
- **Respuesta**: No utilizar datos sesgados fuera de fines académicos; documentar limitaciones éticas; no compartir modelos basados en datos problemáticos

**d) Alternativas Éticas**
Variables alternativas con alta correlación sin sesgo racial explícito:

| Variable | Correlación con MEDV | Interpretación |
|----------|----------------------|----------------|
| LSTAT | -0.738 | % población con bajo estatus socioeconómico |
| RM | 0.695 | Número promedio de habitaciones |
| CRIM | -0.388 | Tasa de criminalidad per cápita |
| TAX | -0.469 | Tasa de impuestos a la propiedad |
| PTRATIO | -0.508 | Ratio estudiantes-profesor |

**4. Marco de Decisión Ética para Variables Problemáticas**

**✅ USAR variable B SI:**
- Contexto es puramente académico/educativo
- Se documenta explícitamente su naturaleza problemática
- El objetivo es estudiar/detectar sesgo histórico
- Hay supervisión ética institucional

**❌ NO USAR variable B SI:**
- El modelo se usará en producción
- Afectará decisiones sobre personas reales
- Existe riesgo de perpetuar discriminación
- No hay transparencia sobre su uso

### Parte II: Titanic - Sesgo de Género y Clase

**1. Análisis Exploratorio de Sesgo**

El dataset Titanic presenta dos fuentes de sesgo bien documentadas:

**Sesgo de Género:**
- Tasa de supervivencia mujeres: **74%**
- Tasa de supervivencia hombres: **19%**
- **Gap de género**: **54.8 puntos porcentuales**

**Sesgo de Clase:**
- Tasa de supervivencia 1ra clase: **63%**
- Tasa de supervivencia 3ra clase: **24%**
- **Gap de clase**: **41.3 puntos porcentuales**

**2. Modelo Baseline (Sin Mitigación)**

Se entrenó un RandomForestClassifier como baseline:

```python
Features utilizadas: ['pclass', 'age', 'sibsp', 'parch', 'fare']
Sensitive feature: 'sex'
```

**Métricas del modelo baseline:**
- **Accuracy**: 0.673
- **Demographic Parity Difference**: 0.113

**Interpretación**: El modelo replica el sesgo de género presente en los datos históricos, con una diferencia significativa en tasas de predicción positiva entre géneros.

**3. Aplicación de Fairlearn - ExponentiatedGradient**

Se aplicó la técnica de **Exponentiated Gradient** con restricción de **Demographic Parity**:

```python
ExponentiatedGradient(
    RandomForestClassifier(n_estimators=100),
    constraints=DemographicParity()
)
```

**¿Cómo funciona?**
- Ajusta los pesos de las predicciones para satisfacer restricciones de fairness
- Busca el equilibrio óptimo entre accuracy y paridad demográfica
- Genera un conjunto de modelos (predictor mixture) en lugar de un solo modelo

**Métricas del modelo fair:**
- **Accuracy**: 0.626
- **Demographic Parity Difference**: 0.055

**4. Análisis de Trade-offs**

**Pérdidas y ganancias:**
- **Performance loss**: 6.9% (de 0.673 a 0.626)
- **Fairness gain**: 0.059 (reducción de disparidad de 0.113 a 0.055)

**Evaluación del trade-off:**
- Reducción de ~51% en disparidad demográfica
- Costo en accuracy dentro de rangos aceptables para muchas aplicaciones
- **Recomendación**: ⚠️ Evaluar caso por caso según contexto de uso

**Consideraciones contextuales:**
- Si el modelo afecta decisiones críticas → priorizar fairness
- Si el contexto histórico del sesgo es injusto → corrección justificada
- Si la diferencia de performance es aceptable → usar modelo fair
- Siempre documentar y monitorear ambas métricas

### Parte III: Análisis Extendido - Ames Housing

**1. Sesgo Socioeconómico y Geográfico**

**Brecha por Neighborhood (terciles de precio):**
- MAPE en barrios caros vs barrios baratos: **≈ 12 p.p.**
- Patrón sistemático: sobreestimación en áreas caras, subestimación en áreas baratas
- **Causa**: Variables como location, amenities actúan como proxies de clase socioeconómica

**Brecha temporal (Year Built):**
- MAPE en viviendas modernas (≥2000) vs antiguas (≤1950): **≈ 6 p.p.**
- Peor desempeño en viviendas antiguas
- **Causa**: Materiales, renovaciones y depreciación no capturadas completamente

**2. Implicaciones Éticas en Aplicaciones Reales**

**Riesgos identificados:**
- Perpetuación de desigualdades espaciales y patrimoniales
- Discriminación indirecta por código postal/vecindario
- Amplificación de brechas existentes en valoración inmobiliaria

**Decisión ética propuesta:**
- **Uso en decisiones hipotecarias**: Solo como apoyo, nunca como único criterio
- Requerir auditorías periódicas de sesgo
- Monitoreo continuo de drift en diferentes segmentos
- Recalibración por vecindario/año
- Controles humanos obligatorios en decisiones finales

## Evidencias

- Notebook de análisis: [Ver Notebook](https://github.com/MatiasJorda/INGENIERIA-DATOS/raw/main/docs/portfolio/UT2/Notebooks/Practica_7.ipynb)


---

## Framework Ético Desarrollado

### Taxonomía de Decisiones según Tipo de Sesgo

**1. Cuándo DETECTAR únicamente (sin corrección automática):**

**Aplica cuando:**
- Sesgo histórico complejo arraigado en injusticia estructural (ej: Boston Housing)
- Alto riesgo de daño social si se perpetúa
- Contexto de investigación o educación
- Proxies inevitables sin capacidad de intervención en fuente de datos

**Acciones recomendadas:**
- Documentar exhaustivamente el sesgo y sus orígenes
- Restringir uso a contextos académicos
- Publicar análisis de limitaciones éticas
- No desplegar en producción

**Ejemplo**: Dataset Boston Housing con variable racial

---

**2. Cuándo DETECTAR + CORREGIR:**

**Aplica cuando:**
- Sesgo sistemático claro y cuantificable (ej: Titanic por género)
- Herramientas técnicas disponibles (Fairlearn, reweighting, post-processing)
- Trade-off performance-fairness es aceptable
- Contexto de producción con supervisión

**Técnicas aplicables:**
- ExponentiatedGradient con constraints
- Grid Search sobre múltiples modelos fair
- Threshold optimization por grupo
- Reweighting de datos de entrenamiento

**Acciones recomendadas:**
- Definir métricas de fairness apropiadas al dominio
- Establecer umbrales de pérdida de performance aceptables
- Implementar monitoreo continuo post-deployment
- Auditorías periódicas de sesgo

**Ejemplo**: Predicción de supervivencia Titanic, clasificación de riesgo crediticio

---

**3. Cuándo RECHAZAR el modelo:**

**Aplica cuando:**
- Alto impacto socioeconómico + sesgo severo no mitigable
- Pérdida de performance inaceptable tras corrección
- Falta de transparencia o explicabilidad
- Datos irreparablemente sesgados o no representativos
- Ausencia de gobernanza o supervisión ética

**Acciones recomendadas:**
- Documentar razones de rechazo
- Buscar fuentes de datos alternativas
- Rediseñar el problema o enfoque
- Considerar soluciones no-ML

**Ejemplo**: Modelos de contratación basados en datos históricos con discriminación severa

---

### Métricas de Fairness Aplicables por Tarea

**Para Clasificación:**
- **Demographic Parity**:
- **Equal Opportunity**:
- **Equalized Odds**: 
- **Calibration**: 

**Para Regresión:**
- **Group Fairness via MAPE/MAE**: Error promedio similar entre grupos
- **Residual Distribution Shift**: Distribución de errores comparable
- **Calibration within groups**: Predicciones no sistemáticamente sesgadas por grupo
- **Quantile Fairness**: Errores similares en diferentes percentiles

---

### Umbrales de Tolerancia por Dominio

| Dominio | Umbral Recomendado | Justificación |
|---------|-------------------|---------------|
| Salud/Medicina | < 2% disparidad | Decisiones críticas de vida/muerte |
| Justicia/Legal | < 3% disparidad | Derechos fundamentales en juego |
| Finanzas/Crédito | < 5% disparidad | Impacto económico significativo |
| Vivienda | < 5% disparidad | Acceso a necesidad básica |
| Marketing/Recomendaciones | < 10% disparidad | Menor impacto individual |
| Investigación | Variable | Según contexto y propósito |

**Nota**: Estos son lineamientos generales; cada caso requiere análisis específico del contexto, stakeholders y regulaciones aplicables.

---

## Reflexión Final

### Lecciones Clave Aprendidas

**1. Sobre Detección de Sesgo:**
- El sesgo puede manifestarse de formas sutiles y complejas
- Variables "inocentes" frecuentemente actúan como proxies de atributos sensibles
- La detección requiere análisis exploratorio profundo y conocimiento del dominio
- No todo sesgo estadístico implica injusticia, pero requiere análisis ético

**2. Sobre Corrección de Sesgo:**
- Fairlearn y herramientas similares son poderosas pero no mágicas
- No corrigen sesgos estructurales en los datos origen
- Siempre existe trade-off entre performance y fairness
- La corrección técnica debe acompañarse de gobernanza y políticas

**3. Sobre Tipos de Sesgo:**
- **Sesgo histórico**: más complejo de corregir, requiere decisiones éticas profundas
- **Sesgo sistemático**: más manejable técnicamente con herramientas apropiadas
- **Sesgo de representación**: requiere mejora de datos, no solo de modelos

**4. Sobre Responsabilidad Profesional:**
- Los data scientists tienen responsabilidad ética sobre impactos de sus modelos
- La transparencia y documentación son obligatorias
- El "no hacer daño" debe priorizarse sobre métricas de performance
- La supervisión humana es indispensable en contextos sensibles

### Dilemas Éticos Críticos

**¿Cuándo es más valioso detectar que corregir automáticamente?**

Detectar es más valioso cuando:
- El sesgo proviene de injusticia histórica que una corrección ocultaría
- Se necesita transparencia sobre cómo se manifiesta el sesgo
- Las aplicaciones son altamente sensibles donde correcciones automáticas pueden generar consecuencias imprevistas
- El objetivo es educativo o de auditoría más que productivo

**¿Cómo balancear transparencia vs utilidad cuando persiste el sesgo?**

El balance se logra mediante:
- **Transparencia**: Documentación exhaustiva del sesgo y sus efectos
- **Utilidad limitada**: Definir contextos de uso apropiados con restricciones claras
- **Explicabilidad**: Permitir auditoría y comprensión de decisiones
- **Supervisión**: Mantener humanos en el loop para decisiones críticas
- **Monitoreo**: Tracking continuo de métricas de fairness en producción

**¿Qué responsabilidades éticas tenemos con sesgos históricos no corregibles?**

Nuestras responsabilidades incluyen:
- **Reconocerlos explícitamente** sin ocultarlos o minimizarlos
- **No reforzarlos** mediante deployment acrítico de modelos
- **Educar** a stakeholders sobre limitaciones y riesgos
- **Abogar** por mejores datos y procesos de recolección
- **Rechazar** proyectos que perpetúen injusticias conocidas

**CRÍTICO: ¿Es mejor un modelo con sesgo conocido y documentado vs uno "corregido" pero impredecible?**

**En la mayoría de los casos, es preferible un modelo con sesgo conocido y documentado porque:**
- Permite tomar decisiones conscientes e informadas
- Facilita auditoría y accountability
- Evita falsa sensación de equidad
- Permite identificar cuándo NO usar el modelo

**Sin embargo, el modelo "corregido" puede ser preferible cuando:**
- Las técnicas de corrección son bien entendidas y validadas
- El contexto permite cierta reducción de performance
- Existe infraestructura para monitoreo continuo
- El sesgo original es sistemático y no histórico-estructural

**En práctica**: La elección depende del dominio, impacto potencial y capacidades de supervisión. La transparencia sobre trade-offs es siempre obligatoria.

---

## Resultados Numéricos Consolidados

### Boston Housing (Regresión + Sesgo Racial)
- **Brecha detectada**: ~10.5% más error (MAPE) en áreas con alta proporción afroamericana
- **Decisión**: Solo uso educativo
- **Lección**: Sesgos históricos profundos no son técnicamente corregibles

### Titanic (Clasificación + Sesgo Sistemático)
- **Brecha original**: 54.8% diferencia en supervivencia por género
- **Post-corrección**: 
  - Performance loss: 6.9%
  - Fairness gain: 51% reducción en disparidad
- **Decisión**: Evaluar caso por caso según contexto

### Ames Housing (Regresión + Sesgo Socioeconómico)
- **Brecha geográfica**: ~12% diferencia en MAPE entre barrios caros y baratos
- **Brecha temporal**: ~6% diferencia por antigüedad de vivienda
- **Decisión**: Uso en hipotecas solo como apoyo con supervisión humana

---

## Herramientas y Técnicas Clave

**Fairlearn:**
- `MetricFrame`: Análisis de métricas desagregadas por grupo
- `demographic_parity_difference`: Métrica de paridad demográfica
- `equalized_odds_difference`: Métrica de oportunidad equalizada
- `ExponentiatedGradient`: Algoritmo de reducción con constraints
- `GridSearch`: Búsqueda exhaustiva sobre modelos fair

**Flujo de trabajo recomendado:**
1. Análisis exploratorio con desagregación por grupos protegidos
2. Entrenamiento de modelo baseline sin mitigación
3. Evaluación de múltiples métricas de fairness
4. Aplicación de técnicas de mitigación (pre-processing, in-processing, post-processing)
5. Evaluación de trade-offs performance-fairness
6. Decisión ética documentada
7. Deployment con monitoreo continuo

---

Esta práctica refuerza que la equidad algorítmica no es opcional ni un "nice-to-have": es un requerimiento ético fundamental en el desarrollo de sistemas de IA responsables. La detección y mitigación de sesgos debe ser parte integral del ciclo de vida de todo proyecto de machine learning, especialmente cuando los modelos afectan decisiones sobre personas y sus oportunidades.

**La pregunta no es si nuestros modelos tienen sesgo, sino qué estamos haciendo al respecto.**

