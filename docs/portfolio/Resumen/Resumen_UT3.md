# Resumen UT3: Feature Engineering

## Aprendizajes Clave de la Unidad

### 1. Feature Engineering Básico
- **Ratios**: `price_per_sqft`, `sqft_per_bedroom` - capturan relaciones de escala
- **Transformaciones**: `log_price`, `sqrt_sqft` - normalizan distribuciones sesgadas
- **Temporales**: `property_age`, `age_category` - extraen información de fechas
- **Interacciones**: `price_age_interaction` - capturan efectos no lineales

**Aplicación real**: En bienes raíces, `price_per_sqft` es más predictivo que precio absoluto porque normaliza por tamaño.

### 2. Encoding Categórico Avanzado
- **Label Encoding**: Para variables ordinales o baja cardinalidad (≤10)
- **One-Hot Encoding**: Para variables nominales de baja cardinalidad (explosión dimensional)
- **Target Encoding**: Para alta cardinalidad (ej: `native-country` con 42 categorías)
- **Encoding Cíclico**: Sin/cos para variables temporales circulares (hora, día de semana)

**Lección aprendida**: En Adult Income, Label Encoding ganó (0.8632 accuracy) vs. One-Hot (0.8483), pero Target Encoding evitó explosión dimensional (6 features vs. 94).

### 3. Prevención de Data Leakage en Encoding
- **Target Encoding con CV**: Calcular encodings dentro de cada fold de cross-validation
- **ColumnTransformer**: Branching por tipo de variable (numéricas, categóricas baja/alta cardinalidad)

**Caso práctico**: Target Encoding sin CV produjo accuracy optimista (0.85) vs. realista con CV (0.80). La diferencia es crítica.

### 4. PCA y Reducción Dimensional
- **PCA Componentes**: Transforma a espacio de menor dimensión (pierde interpretabilidad)
- **PCA Loadings**: Extrae features originales más importantes (mantiene interpretabilidad)
- **Scree Plot**: Identifica "elbow" para elegir número de componentes

**Aplicación real**: En Ames Housing, 38 componentes (80% varianza) redujeron 53% de features manteniendo R² similar (0.886 vs. 0.888).

### 5. Feature Selection
- **Filter Methods** (F-test, Mutual Information): Rápidos, evalúan features independientemente
- **Wrapper Methods** (Forward/Backward Selection, RFE): Lentos, evalúan subconjuntos
- **Embedded Methods** (RF Importance, Lasso): Balance entre velocidad y rendimiento

**Resultado clave**: Lasso y RF Importance lograron mejor RMSE ($26,090) que el baseline ($26,342) con 53% menos features.

### 6. Temporal Feature Engineering
- **Lag Features**: Valores históricos (`.shift(n)`) - comportamiento pasado
- **Rolling Windows**: Promedios/desviaciones de últimos N eventos
- **Expanding Windows**: Acumulados históricos desde el primer evento
- **RFM Features**: Recency, Frequency, Monetary - framework clásico de marketing
- **Time Windows**: Agregaciones en ventanas fijas (7d, 30d, 90d)

**Impacto medible**: En Online Retail, features temporales mejoraron AUC de 0.6615 a 0.7276 (+10%).

## Conceptos Clave que Debes Saber

### Feature Engineering
1. **Ratios normalizan por escala**: `price_per_sqft` > `price` para comparar propiedades
2. **Transformaciones corrigen sesgo**: Log/sqrt para distribuciones con cola larga
3. **Interacciones capturan no-linealidad**: `price * age` puede ser más informativo que cada una sola

### Encoding
1. **Cardinalidad determina método**: ≤10 (One-Hot), 11-50 (Target), >50 (Target/Binary)
2. **Target Encoding requiere CV**: Sin CV = leakage garantizado
3. **Encoding cíclico para tiempo**: Sin/cos preserva continuidad (23h cerca de 0h)

### Feature Selection
1. **Filter**: Rápido para pre-selección (MI captura no-linealidad mejor que F-test)
2. **Wrapper**: Costoso pero captura interacciones (RFE es más rápido que Forward/Backward)
3. **Embedded**: Balance óptimo (Lasso elimina features redundantes automáticamente)

### Temporal Features
1. **Siempre `.shift(1)` antes de aggregations**: Previene leakage
2. **Rolling vs. Expanding**: Rolling (tendencias recientes), Expanding (comportamiento histórico)
3. **TimeSeriesSplit obligatorio**: KFold no respeta orden temporal

## Reflexiones y Casos Prácticos

### Caso 1: Adult Income - Encoding por Cardinalidad
**Problema**: Variables con cardinalidad muy diferente (`sex`: 2, `native-country`: 42).

**Solución aplicada**: ColumnTransformer con branching - One-Hot para baja cardinalidad, Target Encoding para alta.

**Resultado**: Pipeline híbrido (0.8485 accuracy) balanceó performance y dimensionalidad (30 features vs. 94 con One-Hot puro).

**Aplicación real**: En e-commerce, productos tienen miles de categorías. Target Encoding es esencial.

### Caso 2: Ames Housing - Lasso vs. PCA
**Problema**: 81 features, muchas redundantes (ej: `GarageArea` y `GarageCars` correlacionadas).

**Solución**: Lasso eliminó 40 features automáticamente (coeficientes = 0) vs. PCA que mantuvo todas pero transformadas.

**Resultado**: Lasso mejoró RMSE ($26,090) y mantuvo interpretabilidad (coeficientes lineales) vs. PCA ($26,620) que perdió interpretabilidad.

**Lección**: En dominios donde la interpretabilidad importa (bienes raíces, medicina), Feature Selection > PCA.

### Caso 3: Online Retail - Impacto de Features Temporales
**Problema**: Predecir si un usuario comprará de nuevo usando solo features estáticas (cart_size, order_total).

**Solución**: Agregar 30 features temporales (lags, rolling, RFM, time windows).

**Resultado**: AUC mejoró de 0.6615 a 0.7276 (+10%). Las features más importantes fueron `product_diversity_ratio` (10.4%) y `recency_days` (8.3%).

**Aplicación real**: En marketing, RFM features son estándar para segmentación y predicción de churn.

### Caso 4: Temporal Features - Prevención de Leakage
**Problema**: Calcular `rolling_cart_mean_3` sin `.shift(1)` incluye la orden actual (leakage).

**Dificultad**: El error no es obvio - el código funciona, pero el modelo "ve el futuro".

**Solución**: Siempre `.shift(1)` antes de `.rolling()` o `.expanding()`, y usar TimeSeriesSplit.

**Resultado**: AUC realista (0.7276) vs. optimista sin shift (0.85+). La diferencia es crítica.

**Aplicación real**: En producción, este leakage causa modelos que fallan porque "predicen" con información no disponible.

### Caso 5: PCA - Trade-off Varianza vs. Performance
**Problema**: ¿Cuántos componentes usar? 70%, 80%, 90%, 95% de varianza?

**Solución**: Evaluación empírica con cross-validation mostró que 80% varianza (38 componentes) es el "elbow".

**Resultado**: 80% varianza: RMSE $26,715, tiempo 17s. 99% varianza: RMSE $26,822, tiempo 33s. Mejora marginal (<1%) con costo 2x.

**Aplicación real**: En apps móviles, reducir componentes mejora latencia sin pérdida significativa de precisión.

## Dificultades Encontradas y Cómo se Superaron

### Dificultad 1: Target Encoding sin Leakage
**Problema**: Calcular encodings usando todo el dataset contamina el train con info del test.

**Solución**: 
- Usar `category_encoders.TargetEncoder` con `cv` parameter
- O implementar manualmente dentro de TimeSeriesSplit/CrossVal
- Validar que train accuracy no sea mucho mayor que CV accuracy

### Dificultad 2: Elegir entre PCA y Feature Selection
**Problema**: ¿Cuándo usar PCA (pierde interpretabilidad) vs. Feature Selection (mantiene features originales)?

**Solución**:
- **PCA**: Cuando la interpretabilidad no importa (imágenes, genómica) o hay alta correlación
- **Feature Selection**: Cuando la interpretabilidad es crítica (bienes raíces, medicina, finanzas)
- **Híbrido**: PCA Loadings para guiar Feature Selection

### Dificultad 3: Features Temporales con Pandas
**Problema**: Operaciones temporales complejas (rolling, expanding, time windows) pueden ser lentas o propensas a errores.

**Solución**:
- Usar `.groupby()` antes de `.shift()` para asegurar independencia por usuario
- Validar ordenamiento temporal antes de calcular features
- Usar funciones vectorizadas cuando sea posible

### Dificultad 4: Balancear Dimensionalidad vs. Performance
**Problema**: Más features no siempre es mejor - puede causar overfitting o explosión dimensional.

**Solución**:
- Empezar con Filter Methods (MI, F-test) para pre-selección rápida
- Usar Embedded Methods (Lasso, RF) para refinamiento
- Wrapper Methods solo si el dataset es pequeño y el tiempo lo permite

## Aspectos a Tener en Cuenta en el Futuro

### Checklist de Feature Engineering
- [ ] Crear ratios y transformaciones para normalizar distribuciones
- [ ] Analizar cardinalidad antes de elegir método de encoding
- [ ] Usar CV en Target Encoding para prevenir leakage
- [ ] Evaluar interpretabilidad vs. performance (PCA vs. Feature Selection)
- [ ] Implementar features temporales con `.shift(1)` y TimeSeriesSplit
- [ ] Validar que train accuracy ≈ CV accuracy (sin overfitting)

### Reglas de Oro
1. **"La cardinalidad determina el encoding: ≤10 One-Hot, >10 Target Encoding"**
2. **"Feature Selection > PCA cuando la interpretabilidad importa"**
3. **"Siempre `.shift(1)` antes de aggregations temporales"**
4. **"TimeSeriesSplit obligatorio para datos temporales"**

### Aplicaciones en la Vida Real
- **E-commerce**: RFM features para segmentación de clientes, product diversity para recomendaciones
- **Finanzas**: Features temporales de transacciones para detección de fraude
- **Bienes raíces**: Ratios (price_per_sqft) y transformaciones (log) para modelos de pricing
- **Marketing**: Encoding cíclico para patrones estacionales, lags para predicción de demanda

## Conceptos Técnicos Esenciales

### Librerías Clave
- **Pandas**: `.groupby()`, `.shift()`, `.rolling()`, `.expanding()` para features temporales
- **Scikit-learn**: `ColumnTransformer`, `Pipeline`, `PCA`, `SelectKBest`, `RFE`
- **Category Encoders**: `TargetEncoder`, `BinaryEncoder`, `LeaveOneOutEncoder`
- **NumPy**: `np.sin()`, `np.cos()` para encoding cíclico

### Métricas Importantes
- **Mutual Information**: Captura dependencias no lineales (mejor que F-test para relaciones no lineales)
- **Varianza Explicada (PCA)**: % de varianza capturada por componentes (80% es buen balance)
- **Feature Importance (RF)**: Importancia basada en reducción de impureza
- **Lasso Coefficients**: Coeficientes regularizados (features con |coef| ≈ 0 son eliminadas)

### Técnicas Avanzadas
- **Encoding Cíclico**: `sin(2π * x / period)`, `cos(2π * x / period)` para variables circulares
- **Time Windows**: Agregaciones en ventanas fijas (7d, 30d, 90d) vs. ventanas móviles (últimos N eventos)
- **RFM Framework**: Recency (días desde última compra), Frequency (total órdenes), Monetary (gasto promedio/total)

---

**Conclusión**: UT3 enseña que el feature engineering es donde se gana o pierde en machine learning. Las decisiones (encoding, selección, temporales) tienen impacto directo en performance e interpretabilidad. La prevención de leakage y la elección correcta de técnicas según el contexto son fundamentales para modelos exitosos.

