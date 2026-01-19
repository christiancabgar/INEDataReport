# Análisis y Visualización de Datos del INE con Power BI

## Objetivo del Proyecto
El objetivo de esta práctica es adquirir experiencia avanzada en la extracción, análisis y visualización de datos estadísticos oficiales del **Instituto Nacional de Estadística (INE)**. Se busca transformar datos crudos en un reporte interactivo que permita responder preguntas clave sobre la realidad socioeconómica de España.

---

## Plan de Trabajo

### 1. Extracción y Preparación de Datos (ETL)
- [ ] **Selección de Datos:** Descarga de ficheros (CSV, Excel, etc.) desde [ine.es](https://www.ine.es).
- [ ] **Auditoría de Datos:** Identificación de nulos, duplicados e inconsistencias.
- [ ] **Transformación:** Limpieza avanzada con Power Query (Lenguaje M) o Python.
- [ ] **Modelado:** Creación de un esquema de datos (Star Schema) y definición de relaciones.

### 2. Análisis Exploratorio (EDA)
- [ ] Identificación de tendencias iniciales.
- [ ] Uso de histogramas y gráficos base para entender la distribución.
- [ ] Detección de patrones geográficos y temporales.

### 3. Diseño del Reporte e Interactividad
El dashboard debe incluir, como mínimo, los siguientes elementos:
1. **Gráfico de Líneas:** Evolución temporal.
2. **Gráfico de Barras:** Comparativa de categorías.
3. **Mapa Interactivo:** Distribución geográfica por provincias/comunidades.
4. **Tabla de Métricas:** Resumen de KPIs clave.
5. **Gráfico de Dispersión:** Análisis de correlación entre variables.
6. **Segmentadores (Slicers):** Filtros dinámicos por año, región, sexo, etc.

---

## Preguntas de Negocio / Investigación

### I. Encuesta de Población Activa (EPA)
* Evolución de la tasa de empleo/desempleo (últimos 10 años).
* Variación por sectores (Agricultura, Industria, Servicios, etc.).
* Comparativa regional (Comunidades Autónomas) y su evolución.
* Perfil demográfico del desempleo (edad, sexo, estudios).
* Brecha de género en la tasa de empleo.

### II. Censo de Población y Viviendas
* Evolución poblacional por provincias (últimos 20 años).
* Análisis de densidad de población.
* Tamaño medio del hogar y cambios en el tiempo.
* Régimen de tenencia (Propiedad vs. Alquiler).
* Nivel educativo por regiones.

### III. Índice de Precios de Consumo (IPC)
* Evolución de la inflación en España (últimos 15 años).
* Categorías de productos con mayor incremento de precios.
* Inflación por Comunidades Autónomas.
* Relación entre inflación y crecimiento salarial.
* Análisis de estacionalidad de precios.

### IV. Encuesta de Condiciones de Vida
* Evolución del ingreso medio por hogar.
* Tasa de riesgo de pobreza o exclusión social (AROPE).
* Desigualdad regional en niveles de pobreza.
* Impacto del nivel educativo en la calidad de vida.
* Pobreza según la tipología del hogar.