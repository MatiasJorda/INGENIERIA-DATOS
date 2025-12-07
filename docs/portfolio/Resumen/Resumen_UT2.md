# Resumen UT2: Calidad & Ética de Datos

## Aprendizajes Clave de la Unidad

### 1. Tipos de Missing Data (MCAR, MAR, MNAR)
- **MCAR (Missing Completely At Random)**: Los valores faltantes son completamente aleatorios, no hay patrón. Ejemplo: un sensor que falla aleatoriamente.
- **MAR (Missing At Random)**: Los faltantes dependen de variables observadas. Ejemplo: `Year Built` faltante relacionado con `Neighborhood`.
- **MNAR (Missing Not At Random)**: Los faltantes dependen del valor faltante mismo. Ejemplo: `Garage Area` faltante porque realmente no hay garaje (valor = 0).

**Aplicación real**: En análisis de viviendas, identificar que `Garage Area` es MNAR permitió imputar con 0 en lugar de la media, preservando la información semántica.

### 2. Estrategias de Imputación
- **Simple**: Media, mediana, moda (rápido pero puede sesgar)
- **Contextual**: Imputar por grupo (ej: mediana por `Neighborhood` + `House Style`)
- **Avanzada**: KNNImputer, IterativeImputer (MICE) - capturan relaciones multivariadas

**Lección aprendida**: La imputación contextual (por grupo) redujo el sesgo vs. imputación global en Ames Housing.

### 3. Detección de Outliers
- **IQR Method**: Para distribuciones sesgadas (Q1 - 1.5×IQR, Q3 + 1.5×IQR)
- **Z-Score**: Para distribuciones normales (|z| > 3)
- **ML-based**: IsolationForest, LOF para outliers multivariados

**Dificultad superada**: En Air Quality dataset, outliers ambientales reales (eventos puntuales) fueron detectados con IsolationForest, no eliminados automáticamente.

### 4. Feature Scaling y Data Leakage
- **Scalers**: StandardScaler (normal), MinMaxScaler (0-1), RobustScaler (outliers), PowerTransformer (sesgo)
- **Data Leakage**: Escalar ANTES del split contamina el test con info del train

**Regla de oro**: **Split primero, transformar después**. Usar Pipeline de sklearn para automatizar.

**Aplicación real**: En Ames Housing, el leakage aumentó R² de 0.196 a 0.185 (optimista pero falso). El pipeline correcto dio 0.117 (realista).

### 5. Sesgo y Fairness
- **Tipos de sesgo**: Histórico (Boston Housing racial), sistemático (Titanic género), representación
- **Métricas**: Demographic Parity, Equal Opportunity, Equalized Odds
- **Mitigación**: Fairlearn (ExponentiatedGradient), reweighting, threshold optimization

**Caso práctico**: En Titanic, el modelo baseline tenía 54.8% gap de género. Con Fairlearn se redujo a 5.5% con pérdida de accuracy del 6.9% (trade-off aceptable).

## Conceptos Clave que Debes Saber

### Missing Data
1. **Siempre clasificar primero** (MCAR/MAR/MNAR) antes de imputar
2. **Crear indicadores binarios** (`_was_na`) para auditar imputaciones
3. **Validar distribución** antes/después de imputar (no debe cambiar drásticamente)

### Outliers
1. **No eliminar automáticamente**: Un outlier puede ser una mansión legítima
2. **Contexto es clave**: Validar con conocimiento del dominio
3. **Tratar antes de escalar**: Outliers distorsionan StandardScaler/MinMaxScaler

### Escalado
1. **PowerTransformer** para datos sesgados (reduce skew de 12.82 a ~0.05)
2. **RobustScaler** cuando no puedes eliminar outliers
3. **Pipeline obligatorio** para prevenir leakage

### Ética
1. **Documentar decisiones**: Por qué se imputó, por qué se eliminó, qué sesgos se detectaron
2. **Evaluar trade-offs**: Performance vs. fairness
3. **No usar variables problemáticas** en producción (ej: variable racial en Boston Housing)

## Reflexiones y Casos Prácticos

### Caso 1: Ames Housing - Imputación Inteligente
**Problema**: `Year Built` faltante en 5% de casos, correlacionado con `Neighborhood`.

**Solución aplicada**: Imputación jerárquica (mediana por `Neighborhood` + `House Style`) en lugar de mediana global.

**Resultado**: Preservó la coherencia interna del dataset. La correlación con `SalePrice` se mantuvo estable (-0.165 vs. -0.141).

**Lección**: La imputación contextual es más costosa computacionalmente pero produce resultados más confiables.

### Caso 2: Air Quality - Outliers Ambientales Reales
**Problema**: Detección de outliers en datos ambientales con algoritmos ML (IsolationForest, LOF).

**Dificultad**: Diferenciar entre errores de sensor y eventos ambientales reales (picos de contaminación).

**Solución**: Visualización con PCA 2D mostró que outliers formaban clusters, sugiriendo eventos reales. Se documentaron pero no se eliminaron automáticamente.

**Aplicación real**: En monitoreo ambiental, estos "outliers" pueden ser alertas tempranas de problemas (fugas, incendios).

### Caso 3: Feature Scaling - PowerTransformer vs. StandardScaler
**Problema**: Variables con skew extremo (Lot Area: 12.82, Misc Val: 21.94) distorsionaban modelos lineales.

**Solución**: PowerTransformer (Yeo-Johnson) transformó distribuciones hacia normalidad.

**Resultado**: Reducción de skew a ~0.05, mejora en convergencia de modelos lineales, mejor R² en cross-validation.

**Aplicación real**: En finanzas, datos de ingresos/gastos tienen skew similar. PowerTransformer es estándar en estos casos.

### Caso 4: Data Leakage - El Error Sutil
**Problema**: Escalar antes del split introdujo leakage sutil pero significativo.

**Dificultad**: El error no es obvio - el código funciona, pero las métricas son optimistas.

**Solución**: Implementar Pipeline de sklearn que automáticamente previene leakage.

**Resultado**: R² realista (0.117) vs. optimista (0.185). La diferencia es crítica para confiar en el modelo.

**Aplicación real**: En producción, este tipo de leakage causa modelos que fallan en datos nuevos.

### Caso 5: Fairness - Titanic y Sesgo de Género
**Problema**: Modelo de supervivencia replicaba sesgo histórico (74% mujeres vs. 19% hombres sobrevivieron).

**Solución**: ExponentiatedGradient con restricción de Demographic Parity.

**Trade-off**: Pérdida de accuracy del 6.9% (0.673 → 0.626) pero reducción de disparidad del 51% (0.113 → 0.055).

**Aplicación real**: En sistemas de crédito, este trade-off es aceptable para evitar discriminación.

## Dificultades Encontradas y Cómo se Superaron

### Dificultad 1: Clasificar Missing Data Correctamente
**Problema**: Diferenciar entre MAR y MNAR requiere análisis profundo de correlaciones y contexto.

**Solución**: 
- Análisis de correlaciones entre variables con missing y observadas
- Validación con conocimiento del dominio (ej: Garage Area = 0 si no hay garaje)
- Documentación de razonamiento

### Dificultad 2: Balancear Performance vs. Fairness
**Problema**: Mitigar sesgo reduce accuracy, pero ¿cuánto es aceptable?

**Solución**:
- Establecer umbrales por dominio (salud: <2%, finanzas: <5%)
- Evaluar impacto en negocio, no solo métricas técnicas
- Documentar trade-offs explícitamente

### Dificultad 3: Detectar Data Leakage Sutil
**Problema**: El leakage no siempre es obvio (ej: escalar antes de split).

**Solución**:
- Siempre usar Pipeline
- Validar que train.max() < val.min() en datos temporales
- Revisar feature importance para features sospechosas

## Aspectos a Tener en Cuenta en el Futuro

### Checklist de Calidad de Datos
- [ ] Clasificar missing data (MCAR/MAR/MNAR) antes de imputar
- [ ] Validar outliers con contexto del dominio
- [ ] Usar Pipeline para prevenir leakage
- [ ] Documentar decisiones éticas explícitamente
- [ ] Evaluar fairness en modelos que afectan personas
- [ ] Normalizar métricas (per cápita, por área) para comparaciones justas

### Reglas de Oro
1. **"Si hay duda sobre missing data, imputa contextualmente, no globalmente"**
2. **"Split primero, transformar después - siempre usar Pipeline"**
3. **"Un outlier puede ser información valiosa, no eliminarlo sin validar"**
4. **"La equidad algorítmica no es opcional cuando afecta personas"**

### Aplicaciones en la Vida Real
- **Finanzas**: Detección de fraude (outliers), scoring crediticio (fairness)
- **Salud**: Imputación de datos clínicos faltantes, detección de anomalías
- **Retail**: Limpieza de datos de transacciones, detección de productos anómalos
- **Gobierno**: Análisis de políticas públicas con consideraciones de equidad

## Conceptos Técnicos Esenciales

### Librerías Clave
- **Pandas**: Detección de missing, análisis exploratorio
- **Scikit-learn**: Imputadores (SimpleImputer, KNNImputer, IterativeImputer), scalers, Pipeline
- **Fairlearn**: Mitigación de sesgo (ExponentiatedGradient, GridSearch)
- **IsolationForest, LOF**: Detección de outliers multivariados

### Métricas Importantes
- **Missing %**: Porcentaje de valores faltantes por columna/fila
- **Skewness**: Medida de asimetría (normal: ~0, sesgado: >1 o <-1)
- **Demographic Parity Difference**: Diferencia en tasas de predicción positiva entre grupos
- **Equalized Odds Difference**: Diferencia en TPR y FPR entre grupos

---

**Conclusión**: UT2 enseña que la calidad de datos no es solo técnica, sino también ética. Cada decisión (imputar, eliminar, escalar) tiene implicaciones que deben documentarse y justificarse. La prevención de leakage y la evaluación de fairness son fundamentales para modelos confiables y responsables.

