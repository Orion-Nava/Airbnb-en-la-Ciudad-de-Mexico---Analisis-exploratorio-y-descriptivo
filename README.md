# Airbnb en la Ciudad de México — Análisis Exploratorio y Descriptivo (EDA)

**Objetivo:** realizar un EDA riguroso del dataset oficial de Airbnb para CDMX y, a partir de él, construir un análisis descriptivo que caracterice el mercado desde tres ejes: **precio**, **espacio/geografía** y **comportamiento del anfitrión**.  

Este proyecto **NO** busca optimizar un modelo predictivo para producción; incluye un **modelo explicativo** como herramienta de síntesis/validación interna.

---

## Contenido

1. **EDA**: correcciones, estandarización, datos faltantes, selección de variables, descripción, outliers y asociaciones.
2. **Precios**: distribución, asimetría, outliers explicables, y patrón espacial (alcaldías + mapas agregados por cuadrícula).
3. **Características del inmueble**: capacidad, dormitorios, baños, camas y su relación con el precio.
4. **Análisis temporal**: evolución del mercado (aperturas/listados a lo largo del tiempo).
5. **Amenities / Espacios y servicios**: minería de texto ligera (normalización), nube de palabras y porcentajes de “cumplimiento”.
6. **Ocupación mínima y máxima**: distribución de requisitos de noches.
7. **Calidad operativa del anfitrión**: tiempos de respuesta, response rate, acceptance rate y verificaciones.
8. **Modelo explicativo**: OLS con errores robustos + contraste no lineal con Random Forest.

---

## Dataset

- **Fuente:** dataset oficial de Airbnb (listings).
- **Variables clave:** `price`, `room_type`, `accommodates`, `bathrooms`, `bedrooms`, `beds`, `host_since`, métricas del anfitrión, ubicación (lat/long) y **alcaldía** (por *spatial join* con shapefile).

> Nota: por la asimetría del precio y su sensibilidad a valores extremos, se usan medidas robustas (mediana/IQR) cuando aplica y se analiza explícitamente el rol de los outliers.

---

## Metodología (resumen técnico)

### 1) Limpieza y estandarización
- Corrección del formato de `price` (remoción de símbolo y separadores).
- Eliminación de duplicados.
- Conversión de tipos y normalización de variables categóricas.
- Diagnóstico de faltantes y selección de variables relevantes.

### 2) Outliers: decisión informada (no automática)
- Identificación de precios extremos.
- Distinción entre:
  - **outliers erróneos/inconsistentes** vs
  - **outliers explicables** (propiedades grandes, alta capacidad, muchas habitaciones/baños, etc.).

### 3) Asociación entre variables (según tipo de dato)
- Medidas apropiadas al tipo de variable (no forzar correlación “por costumbre”).
- Separación conceptual: asociación **no** implica causalidad.

### 4) Geoespacial
- *Spatial join* (puntos lat/long → alcaldías).
- Mapas agregados por **cuadrículas de 4 km²** usando **mediana** e **IQR** para reducir sesgos por outliers.
- Densidades y concentración territorial por alcaldía.

### 5) Amenities (texto)
- Normalización de listas de amenities:
  - minúsculas, limpieza de espacios, parsing seguro.
- Nube de palabras + porcentajes de cobertura por amenity.

### 6) Modelo explicativo (síntesis, no producción)
- Transformación: `y = log(price)` con `price > 0`.
- OLS con errores robustos **HC3**, con referencias:
  - `room_type` referencia: **Entire home/apt**
  - `Alcaldía` referencia: **Cuauhtémoc**
- Contraste no lineal: **RandomForestRegressor** para explorar posibles no linealidades/interacciones.

---

## Resultados clave

### Mercado y geografía
- **Concentración territorial fuerte:** Cuauhtémoc concentra ~**47.1%** del total de listados y una densidad aproximada de **335.1 listings/km²**.
- **Patrón espacial claro:** centro/oeste/suroeste tienden a ser más caros; norte/este/sureste concentran opciones más accesibles.
- Zonas consistentemente caras (ejemplos): **Polanco, Santa Fe, Lomas de Chapultepec, Condesa**.

### Distribución del precio
- **Asimetría positiva marcada (cola derecha)**: la media se “infla” por un segmento premium pequeño.

### Evolución temporal
- Los primeros anfitriones aparecen alrededor de **2009**.
- Crecimiento acelerado hasta un pico cercano a **2016**.
- Luego disminución y fluctuaciones (repunte moderado 2022–2023 y caídas posteriores), consistente con expansión → maduración.

### Anfitrión y operación
- Desempeño alto en respuesta:
  - ~**83%** responde en la **primera hora**.
  - >**90%** dentro del **primer día**.
- **Verificación elevada** (~**95%**).
- **Heterogeneidad en acceptance rate**: valores bajos/extremos sugieren listados poco activos o restricciones no observadas.

### Amenities
- **WiFi** es prácticamente universal.
- Le siguen **cocina** y **agua caliente** (con utensilios/insumos básicos).
- ~**>50%** ofrece detectores (humo/CO).
- **< 1/3** ofrece estacionamiento gratis.

---

## Modelo explicativo (interpretación)

### OLS (log-precio)
- Ajuste: **R² ajustado ≈ 0.491** (explica ~50% de la variación en log-precio).
- Efectos por tipo de alojamiento (vs **Entire home/apt**):
  - **Private room** ≈ **36%** más barato.
  - **Shared room** ≈ **73.4%** más barato.
  - **Hotel room**: no difiere significativamente (en este ajuste).

> Interpretación: incluso controlando por ubicación (alcaldía) y características físicas, **el tipo de alojamiento** y la **localización** siguen siendo determinantes del precio.

### Random Forest (contraste no lineal)
- Confirma coherencia global:
  - el tipo de alojamiento domina,
  - luego variables estructurales (capacidad/baños/dormitorios),
  - la ubicación mantiene rol relevante distribuido entre alcaldías.
- Sugiere no linealidades/interacciones que el OLS no captura por completo (esperable).

---

## Reproducibilidad

### Requisitos
- Python
- Librerías principales:
  - `pandas`, `numpy`, `matplotlib`, `seaborn`
  - `geopandas`, `shapely`
  - `folium` (+ plugins)
  - `statsmodels`
  - `scikit-learn`
  - `wordcloud`, `rapidfuzz`
