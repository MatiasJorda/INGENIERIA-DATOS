# Resumen UT4: Datos Especiales - Geoespacial

## Aprendizajes Clave de la Unidad

### 1. GeoPandas y Datos Geoespaciales
- **GeoDataFrame**: Extensión de DataFrame con columna `geometry` (puntos, líneas, polígonos)
- **CRS (Coordinate Reference System)**: Sistema de coordenadas (EPSG:4326 WGS84, EPSG:3857 Web Mercator)
- **Operaciones espaciales**: `sjoin` (spatial join), `sjoin_nearest` (nearest neighbor), `dissolve` (agregación)

**Aplicación real**: En análisis urbano, GeoPandas permite calcular densidades, distancias y cobertura de servicios de forma precisa.

### 2. Proyecciones Cartográficas
- **EPSG:4326 (WGS84)**: Coordenadas geográficas (lat/lon), estándar web, pero distorsiona áreas
- **EPSG:3857 (Web Mercator)**: Coordenadas proyectadas (metros), preciso para áreas/distancias, estándar web mapping
- **EPSG:5346 (POSGAR 2007)**: Proyección local para Argentina, óptima para análisis nacionales

**Lección crítica**: **Siempre proyectar a CRS métrico antes de calcular áreas o distancias**. WGS84 da resultados incorrectos.

**Caso práctico**: En CABA, calcular área en WGS84 dio ~200 km² (incorrecto) vs. EPSG:3857 dio ~203 km² (correcto, diferencia 1.5%).

### 3. Normalización de Métricas
- **Por área (densidad)**: `métrica / área_km2` - compara concentración espacial
- **Per cápita**: `métrica / población` - compara acceso/uso relativo

**Aplicación real**: En análisis de cobertura SUBTE, San Nicolás tiene 13.6 contactos SUACI per cápita vs. Villa Crespo con 0.5, revelando patrones que valores absolutos ocultan.

### 4. Joins Espaciales
- **`sjoin`**: Encuentra geometrías que se intersectan/contienen (ej: estaciones dentro de barrios)
- **`sjoin_nearest`**: Encuentra la geometría más cercana y calcula distancia (ej: estación más cercana a cada barrio)
- **`overlay`**: Operaciones booleanas (union, intersection, difference) - excluir zonas prohibidas

**Caso práctico**: En CABA, `sjoin_nearest` identificó que Villa Riachuelo está a 6.6 km de la estación más cercana, revelando gaps de cobertura.

### 5. Visualización Geoespacial
- **Coropléticos**: Mapas temáticos con clasificación (quantiles, equal interval) y paletas de colores
- **Basemaps**: Superposición de mapas base (CartoDB) para contexto geográfico
- **Folium**: Mapas interactivos HTML con capas, zoom, pan, información al click

**Aplicación real**: Mapas interactivos con Folium permiten a stakeholders explorar datos sin conocimiento técnico.

### 6. Optimización y Formatos
- **GeoParquet**: Formato eficiente (50-60% más pequeño que GeoJSON, 2-3x más rápido)
- **Simplificación de geometrías**: Reducir puntos con `simplify(tolerance)` - 50-100m para web, 0m para análisis
- **Hexgrid/H3**: Discretización en hexágonos uniformes para comparaciones justas

**Resultado clave**: Simplificación 50m reduce tamaño 50% con pérdida visual mínima, ideal para visualización web.

## Conceptos Clave que Debes Saber

### CRS y Proyecciones
1. **WGS84 (EPSG:4326)**: Para visualización, GPS, intercambio - NO para cálculos de área/distancia
2. **Web Mercator (EPSG:3857)**: Para análisis espacial, cálculos métricos, mapas web
3. **Siempre proyectar antes de calcular**: `gdf.to_crs(epsg=3857)` antes de `.area` o `.distance()`

### Normalización
1. **Densidad (por área)**: Para comparar concentración espacial (hab/km², estaciones/km²)
2. **Per cápita**: Para comparar acceso/uso relativo (contactos/persona, estaciones/persona)
3. **Sin normalización**: Barrios grandes dominan visualmente aunque tengan menor densidad

### Joins Espaciales
1. **`sjoin`**: Para "qué está dentro de qué" (estaciones en barrios, barrios en comunas)
2. **`sjoin_nearest`**: Para "qué está más cerca" (hospital más cercano, estación más cercana)
3. **Validar CRS**: Ambos GeoDataFrames deben tener el mismo CRS antes de unir

### Visualización
1. **Coropléticos**: Clasificación por quantiles (5 grupos iguales) vs. equal interval (rangos iguales)
2. **Basemaps**: Mejoran interpretación pero aumentan tiempo de renderizado
3. **Folium**: Ideal para comunicación, pero archivos HTML pueden ser grandes (>5 MB)

## Reflexiones y Casos Prácticos

### Caso 1: CABA - Cálculo de Densidad Poblacional
**Problema**: Calcular densidad poblacional por radio censal requiere áreas precisas en km².

**Dificultad inicial**: Calcular área en WGS84 (EPSG:4326) dio resultados incorrectos porque los grados no son metros.

**Solución aplicada**: Proyección a EPSG:3857 (Web Mercator) antes de calcular área.

**Resultado**: Densidades correctas (top: 110,200 hab/km² en Flores) vs. valores distorsionados en WGS84.

**Aplicación real**: En planificación urbana, densidades precisas son críticas para zonificación y servicios.

### Caso 2: Contactos SUACI - Normalización Per Cápita
**Problema**: San Nicolás tiene 399,369 contactos vs. Villa Crespo con 50,000 - ¿cuál tiene mayor demanda?

**Solución**: Normalización per cápita reveló que San Nicolás tiene 13.6 contactos/persona vs. Villa Crespo con 0.5.

**Interpretación**: San Nicolás es barrio central con alta actividad administrativa/comercial, aunque población residente baja.

**Aplicación real**: En gestión pública, métricas per cápita identifican zonas que requieren mayor capacidad de atención.

### Caso 3: Cobertura SUBTE - Joins Espaciales
**Problema**: Identificar gaps de cobertura de transporte público en CABA.

**Solución aplicada**: 
- `sjoin` para contar estaciones por barrio
- `sjoin_nearest` para calcular distancia mínima desde centroide de cada barrio

**Resultado**: Villa Riachuelo está a 6.6 km de la estación más cercana, revelando necesidad de expansión.

**Aplicación real**: En planificación de transporte, estos análisis guían decisiones de inversión en infraestructura.

### Caso 4: Hexgrid - Comparaciones Justas
**Problema**: Barrios de tamaño muy variable (0.01 km² a 5 km²) distorsionan visualizaciones.

**Solución**: Discretización en hexágonos H3 nivel 9 (~0.5 km² por celda) para comparaciones uniformes.

**Resultado**: Eliminó sesgo por tamaño, permitiendo comparar métricas sin efecto de área variable.

**Aplicación real**: En análisis de crimen o servicios, hexgrid permite identificar hotspots sin sesgo geográfico.

### Caso 5: OSMnx - Distancias por Red Vial
**Problema**: Distancia euclidiana subestima el acceso real (no considera calles, barreras).

**Solución**: Descarga de grafo de calles con OSMnx, cálculo de shortest path usando NetworkX.

**Resultado**: Distancias reales son 500-1,000 m mayores que euclidianas, especialmente en barrios periféricos.

**Aplicación real**: En análisis de accesibilidad, distancias por red vial son esenciales para planificación realista.

### Caso 6: Optimización - GeoParquet vs. GeoJSON
**Problema**: GeoJSON de 800 KB, lectura lenta (0.6s) para visualización web.

**Solución**: Conversión a GeoParquet con simplificación 50m.

**Resultado**: Tamaño reducido 50% (400 KB), lectura 3x más rápida (0.2s), pérdida visual mínima.

**Aplicación real**: En pipelines de producción, GeoParquet es estándar para almacenamiento interno.

## Dificultades Encontradas y Cómo se Superaron

### Dificultad 1: CRS Inconsistente
**Problema**: Intentar `sjoin` con GeoDataFrames en CRS diferentes produce errores o resultados incorrectos.

**Solución**: 
- Siempre verificar CRS: `gdf.crs`
- Proyectar a CRS común antes de operaciones: `gdf.to_crs(epsg=3857)`
- Validar geometrías: `gdf.is_valid`, `gdf.make_valid()` si es necesario

### Dificultad 2: Geometrías Inválidas
**Problema**: Polígonos con auto-intersecciones o anillos inválidos fallan en operaciones espaciales.

**Solución**:
- Validar: `gdf[~gdf.is_valid]` identifica problemas
- Corregir: `gdf.geometry = gdf.geometry.make_valid()` (puede cambiar geometría ligeramente)

### Dificultad 3: Performance con Datasets Grandes
**Problema**: Operaciones espaciales (sjoin, overlay) son computacionalmente costosas con miles de polígonos.

**Solución**:
- Simplificación de geometrías (50-100m para visualización)
- Indexación espacial (R-tree automático en GeoPandas)
- Usar GeoParquet para almacenamiento (más rápido que GeoJSON)

### Dificultad 4: Normalización Adecuada
**Problema**: Valores absolutos engañan - barrios grandes dominan visualmente aunque tengan menor densidad.

**Solución**:
- Calcular densidad (por área) para concentración espacial
- Calcular per cápita para acceso/uso relativo
- Justificar elección según pregunta de negocio

## Aspectos a Tener en Cuenta en el Futuro

### Checklist de Análisis Geoespacial
- [ ] Verificar y proyectar CRS antes de cálculos de área/distancia
- [ ] Validar geometrías antes de operaciones espaciales
- [ ] Normalizar métricas (densidad o per cápita) según contexto
- [ ] Usar `sjoin` para intersecciones, `sjoin_nearest` para distancias
- [ ] Simplificar geometrías para visualización web (50-100m)
- [ ] Usar GeoParquet para almacenamiento interno, GeoJSON para web

### Reglas de Oro
1. **"Siempre proyectar a CRS métrico antes de calcular áreas o distancias"**
2. **"Normalizar antes de comparar: densidad para concentración, per cápita para acceso"**
3. **"Validar CRS y geometrías antes de joins espaciales"**
4. **"GeoParquet para análisis, GeoJSON para web, Shapefile para intercambio"**

### Aplicaciones en la Vida Real
- **Planificación Urbana**: Análisis de densidad, cobertura de servicios, accesibilidad
- **Logística**: Optimización de rutas, análisis de cobertura de entregas
- **Salud Pública**: Análisis de distribución de hospitales, accesibilidad a servicios
- **Retail**: Análisis de ubicación de tiendas, cobertura de mercado
- **Gobierno**: Análisis de políticas públicas, distribución de recursos

## Conceptos Técnicos Esenciales

### Librerías Clave
- **GeoPandas**: Manipulación de datos geoespaciales (extensión de pandas)
- **Folium**: Mapas interactivos HTML (basado en Leaflet.js)
- **Contextily**: Basemaps para visualización (CartoDB, OpenStreetMap)
- **OSMnx**: Descarga y análisis de grafos de calles desde OpenStreetMap
- **H3**: Sistema de discretización hexagonal (Uber)

### Operaciones Espaciales
- **`dissolve()`**: Agregar polígonos por atributo (ej: radios → barrios)
- **`sjoin()`**: Spatial join - encontrar intersecciones
- **`sjoin_nearest()`**: Nearest neighbor - encontrar más cercano y distancia
- **`overlay()`**: Operaciones booleanas (union, intersection, difference)
- **`centroid`**: Centroide de polígonos (útil para distancias)

### Formatos de Almacenamiento
- **GeoJSON**: Estándar web, legible, pero grande y lento
- **Shapefile**: Compatible con software legacy, múltiples archivos (.shp, .shx, .dbf)
- **GeoParquet**: Eficiente, comprimido, rápido - ideal para análisis

### Técnicas Avanzadas
- **Simplificación**: `geometry.simplify(tolerance)` - reduce puntos manteniendo forma
- **Hexgrid/H3**: Discretización en hexágonos uniformes para comparaciones justas
- **OSMnx + NetworkX**: Cálculo de distancias reales por red vial (shortest path)

---

**Conclusión**: UT4 enseña que el análisis geoespacial requiere entender cartografía (CRS, proyecciones) además de programación. La normalización adecuada y la elección correcta de operaciones espaciales son fundamentales para análisis precisos. GeoPandas democratiza el análisis espacial, pero requiere disciplina en proyecciones y validación de geometrías.

