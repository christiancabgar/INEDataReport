# Bitácora de Desarrollo: Informe INE

Este documento registra los pasos técnicos, decisiones de diseño y fuentes de datos utilizadas para la creación del reporte en Power BI.

---

## Fase 1: Extracción de Datos (Data Sourcing)

### 1.1. Encuesta de Población Activa (EPA) - Tasas de Paro
* **Tabla seleccionada:** Tasas de paro por distintos grupos de edad, sexo y comunidad autónoma.
* **Ruta de acceso:** `INEbase / Mercado laboral / Actividad, ocupación y paro / Encuesta de población activa / Resultados / Trimestrales / Series desde 2002 / Resultados por comunidades autónomas / Parados`.
* **Rango Temporal:** 2016T1 - 2025T3 (39 trimestres/10 años).
* **Dimensiones extraídas (en orden):** 1. Sexo (Ambos sexos, Hombres, Mujeres).
    2. Comunidades y Ciudades Autónomas (Nacional + 19 regiones).
    3. Edad (Total, Menores de 25 años, 25 y más años).
* **Formato original:** CSV (Delimitado por punto y coma `;`).

### 1.2. Encuesta de Población Activa (EPA) - Tasas de Empleo
* **Tabla seleccionada:** Tasas de empleo por distintos grupos de edad, sexo y comunidad autónoma.
* **Filtros aplicados:**
    * **Sexo:** Ambos sexos, Hombres, Mujeres.
    * **Geografía:** Total Nacional + 19 CC.AA.
    * **Edad:** Total, Menores de 25 años, 25 y más años.
    * **Temporalidad:** 2016T1 - 2025T3 (39 trimestres).
* **Consistencia:** Dimensiones idénticas a la tabla de Paro para permitir cruce de datos.

### 1.3. Encuesta de Población Activa (EPA) - Ocupados por Sector
* **Tabla seleccionada:** Ocupados por sector económico, sexo y comunidad autónoma. Valores absolutos.
* **Rango Temporal:** 2016T1 - 2025T3.
* **Dimensiones:**
    * **Sectores:** Agricultura, Industria, Construcción y Servicios (incluye Total).
    * **Geografía:** Nacional + 19 CC.AA.
    * **Sexo:** Ambos sexos, Hombres, Mujeres.
* **Tipo de dato:** Valores absolutos (miles de personas).

### 1.4. Índice de Precios de Consumo (IPC)
* **Tabla seleccionada:** Índices por comunidades autónomas: general y de grupos ECOICOP (Base 2021).
* **Rango Temporal:** Enero 2011 - Diciembre 2025 (15 años).
* **Dimensiones:** * **Grupos:** General + 12 grupos COICOP.
    * **Geografía:** Nacional + 19 CC.AA.
* **Dato extraído:** Índice base 2021.
* **Nota técnica:** Debido a limitaciones en el servidor del INE para descargas masivas, el set de IPC se dividió en dos archivos (2011-2018 y 2019-2025) para garantizar la integridad de los datos. Se unirán mediante una operación de *Append* en Power Query.

### 1.5. Cifras de Población (Residentes)
* **Tabla seleccionada:** Población residente por fecha, sexo y edad (Series desde 1971).
* **Rango Temporal Real:** 01/01/2003 - 01/01/2022 (20 años).
* **Filtros aplicados:** * **Edad:** Total.
    * **Sexo:** Ambos sexos.
    * **Fecha:** 1 de enero de cada año.

### 1.6. Encuesta de Condiciones de Vida (ECV) - Vivienda
* **Tabla seleccionada:** Hogares por régimen de tenencia de la vivienda y comunidad autónoma.
* **Rango Temporal:** 2014 - 2024.
* **Dimensiones extraídas:** * **Tenencia:** Propiedad (con/sin hipoteca) y Alquiler (precio mercado/inferior).
    * **Geografía:** Nacional + 19 regiones.
* **Objetivo:** Analizar la transición del modelo de propiedad al alquiler y el peso de la carga hipotecaria.

### 1.7. Riesgo de Pobreza y Educación (ECV Nacional)
* **Tabla seleccionada:** Riesgo de pobreza o exclusión social (objetivo Europa 2030) por nivel de formación alcanzado.
* **Variables clave:** Tasa AROPE vs Nivel de estudios (desde primaria hasta superior).
* **Rango Temporal:** 2014 - 2024.
* **Objetivo:** Establecer la correlación nacional entre formación académica y seguridad económica.

---

## Fase 2: ETL y Transformación (Power Query)

### 2.1. Carga y Consolidación Inicial
* **Método:** Conector de Carpeta con extracción individual de binarios para mantener la flexibilidad del esquema.
* **Integración IPC:** Se realizó un *Append* de las series 2011-2018 y 2019-2025 en una única tabla de hechos `Fact_IPC`.
* **Mantenimiento:** Se deshabilitó la carga de consultas auxiliares para optimizar el rendimiento del modelo.

### 2.2. Normalización de Series Temporales (Mensuales)
* **Tabla afectada:** `Fact_IPC`.
* **Transformación:** Se aplicó una división de columna por el delimitador "M" para separar el año del mes.
* **Generación de Fechas:** Se creó una columna personalizada mediante la función `#date()` para establecer el primer día de cada mes, convirtiendo el dato de texto en un objeto de tipo Fecha.
* **Validación de Tipos:** Se ajustó la columna `Total` a formato de Número Decimal para permitir operaciones aritméticas sobre los índices de precios.

### 2.3. Normalización de Series Temporales (Trimestrales)
* **Tablas afectadas:** Bloque EPA (`Fact_EPA_Paro`, `Fact_EPA_Empleo`, `Fact_EPA_Sectores`).
* **Transformación:** Segmentación de la columna original mediante el delimitador "T" para aislar el año y el índice del trimestre.
* **Lógica de Fechas:** Implementación de una estructura condicional `if/else` en Power Query para asignar el primer día del primer mes de cada trimestre (T1: Enero, T2: Abril, T3: Julio, T4: Octubre).
* **Resultado:** Estandarización de todas las tablas de mercado laboral bajo un formato de fecha común, permitiendo la correlación temporal con el IPC y la Población.

### 2.4. Depuración de Encuestas y Filtrado Jerárquico
* **Tablas:** `Fact_Vivienda`, `Fact_Pobreza`.
* **Normalización Anual:** Se transformó el año (entero) en un objeto de fecha (1 de enero) mediante la función `#date()` para asegurar la compatibilidad con el eje cronológico del modelo.
* **Integridad del Modelo:** Se filtró el registro "Total Nacional" en la dimensión geográfica para evitar duplicidades en las agregaciones por CCAA.
* **Tipado:** Se validó que la columna `Total` mantenga el formato de Número Decimal para el análisis de porcentajes y tasas.

### 2.5. Resolución de Errores de Tipado
* **Incidencia:** Error de conversión en `Fact_EPA_Sectores` por formato de millares/decimales del INE.
* **Solución:** Uso de "Configuración Regional" para interpretar correctamente la coma decimal y reemplazo de valores nulos (..) por 0.
* **Nota técnica:** Se corrigió error de "Tipo de dato" en Sectores realizando la limpieza de caracteres no numéricos (..) antes de la declaración del tipo Decimal.

### 2.6. Filtrado de Agregaciones Pre-calculadas (Limpieza de Totales)
* **Incidencia:** Presencia de filas "Total", "Ambos sexos" y "Total Nacional" en las tablas de hechos.
* **Acción:** Se han filtrado estas filas en Power Query para evitar la duplicidad de datos (doble conteo) en las visualizaciones.
* **Resultado:** Las tablas ahora solo contienen datos atómicos, permitiendo que las medidas DAX calculen los totales de forma dinámica y correcta.

---

## Fase 3: Modelado de Datos (Arquitectura)

> *Nota: En esta fase integraremos todas las tablas descargadas.*

### 3.1. Arquitectura de Modelo en Estrella
* **Dimensiones:** Creación de `Dim_Calendario` y `Dim_Geografia` mediante DAX para centralizar los filtros temporal y regional.
* **Relaciones:** Establecimiento de vínculos 1:N con dirección de filtro única desde dimensiones hacia tablas de hechos.
* **Atomicidad IPC:** Se ha mantenido el desglose por Grupos ECOICOP eliminando el Índice General para permitir análisis segmentado sin duplicidades.

### 3.2. Resolución de Jerarquías y Granularidad
* **Fact_Poblacion:** Implementación de columna "Comunidad Autónoma" mediante lógica de mapeo de códigos provinciales (01-52) en Power Query.
* **Relaciones:** Vinculación exitosa de todas las tablas de hechos a `Dim_Calendario` y `Dim_Geografia` (con excepción de `Fact_Pobreza`, de carácter nacional).
* **Modelo:** Esquema en estrella validado para el filtrado cruzado dinámico.

---

## Fase 4: Inteligencia de Negocio (DAX)

### 4.1. Infraestructura de Cálculos
* **Organización:** Creación de tabla maestra `_Medidas` para centralizar la lógica de negocio y facilitar el mantenimiento del modelo.
* **Implementación de KPIs:**
    * **Mercado Laboral:** Medida de Tasa de Paro promedio.
    * **Precios:** Cálculo de Variación Interanual del IPC mediante Time Intelligence (`SAMEPERIODLASTYEAR`).
    * **Social:** Medida de Riesgo de Pobreza (AROPE) vinculada a formación.

---

## Fase 5: Diseño de Interfaz y Narrativa de Datos

### 5.1. Validación de Requerimientos Gráficos
* **Cumplimiento:** Se han integrado los 6 elementos obligatorios (Líneas, Barras, Mapa, Tabla KPI, Dispersión y Slicers).
* **Matriz de Respuestas:** Se ha mapeado cada gráfico con las preguntas de investigación (I-IV) para asegurar la cobertura total del análisis.

### 5.2. Estándares de Visualización
* **Interactividad:** Sincronización de segmentadores entre páginas para mantener el contexto de análisis (Año/Región).
* **Ejes:** Implementación de ejes Y secundarios para permitir la comparativa directa entre tasas porcentuales e índices de precios.

### 5.3. Implementación Página 1: Análisis Macro
* **Gráficos:** Se han configurado el gráfico de líneas (Evolución) y el de dispersión (Correlación) con ejes de reproducción temporal.
* **Interactividad:** Los segmentadores permiten filtrar el análisis regional para comparar el comportamiento de la inflación frente al desempleo local.
* **Validación:** El gráfico de dispersión confirma visualmente la relación entre las variaciones de precios y el dinamismo del mercado laboral por CCAA.

### 5.4. Ajustes de Precisión Visual
* **Ejes:** Implementación de eje Y secundario en gráficos de líneas para permitir la comparativa de escalas dispares (Tasas de paro vs Variaciones IPC).
* **Geo-referenciación:** Limpieza de prefijos numéricos en nombres de CCAA y asignación de categoría "Estado o Provincia" para habilitar el mapeo de Bing.
* **DAX:** Refinamiento de medidas de variación para omitir periodos sin datos y evitar distorsiones al final de las series temporales.

### 5.4. Refinamiento de Cálculos Temporales
* **Medida:** `Variacion IPC (%)`.
* **Corrección:** Inclusión de lógica condicional `IF(ISBLANK(...))` para evitar el desplome de la serie en periodos futuros sin datos.
* **Visualización:** Implementación de Eje Y secundario para sincronizar la lectura de la Tasa de Paro (escala 15-30) con la Inflación (escala -1 a 5).

### 5.5. Finalización de la Capa Geográfica y Visual
* **Mapa:** Implementación de formato condicional por degradado en `Tasa de Paro (%)` para análisis de intensidad regional.

### 5.6. Optimización del Análisis de Dispersión
* **Incidencia:** El gráfico de dispersión presentaba un "efecto aplanado" al agregar grandes intervalos temporales debido a la convergencia de promedios históricos.
* **Solución:** Implementación de "Eje de reproducción" utilizando la dimensión temporal (`Año`). Esto permite visualizar la evolución dinámica de la correlación Paro-IPC sin perder la granularidad de los datos.

### 5.7. Implementación Página 2: Análisis Social y Laboral
* **Sectores y Género:** Uso de barras apiladas para cruzar la ocupación sectorial con la variable sexo, permitiendo identificar segregación laboral.
* **Métricas de Bienestar:** Tabla resumen que integra datos de población (Censo) y tasas AROPE (Condiciones de Vida).
* **Demografía del Paro:** Segmentación por tramos de edad para responder a la vulnerabilidad de colectivos específicos.

### 5.8. Auditoría de Datos en Tabla de Métricas
* **Corrección de Agregación:** Se detectó suma errónea de población histórica. Se sustituyó por la medida `Población Actual` para reflejar solo el último censo disponible.
* **Contexto Nacional:** Se identificó que la Tasa AROPE no tiene desglose regional en el dataset original, por lo que se decide extraerla de la tabla regional para evitar lecturas erróneas.
* **Enriquecimiento Visual:** Implementación de iconos de estado y barras de datos para facilitar la jerarquización de CCAA por volumen poblacional.

### 5.9. Implementación Página 3: Calidad de Vida y Vivienda
* **Vivienda:** Visualización del régimen de tenencia para contrastar la cultura de propiedad vs. alquiler.
* **Educación y Bienestar:** Gráfico de barras que valida la correlación inversa entre nivel de estudios y riesgo de exclusión social (AROPE).

### 5.10. Implementación Página 3:
* **Mapa Interactivo:** Visualización

### 5.11. Optimización de la Navegación (UX)
* **Interactividad:** Implementación de un objeto visual "Navegador de páginas" para facilitar el tránsito entre los tres bloques de análisis.
* **Sincronización:** Se ha verificado que la navegación mantiene los filtros de segmentación (Año/CCAA) activos entre páginas gracias a la sincronización de slicers.
* **Estado Final:** El reporte cumple con los estándares de un dashboard corporativo, permitiendo una exploración no lineal de los datos.
