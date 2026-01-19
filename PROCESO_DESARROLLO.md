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

### 2.1. Carga Inicial (EPA)
* **Importación:** Importación realizada mediante el conector de Texto/CSV.
* **Codificación:** Se seleccionó UTF-8 para garantizar la integridad de caracteres especiales (tildes, eñes).
* **Entorno de Trabajo:** Acceso al Editor de Power Query para el proceso de limpieza y tipado.

### 2.2. Ingeniería de Atributos Temporales
* **Descomposición de Periodo:** Se dividió la columna original `Periodo` usando el delimitador "T" para separar el Año del Trimestre.
* **Normalización de Fechas:** Se creó una columna calculada `Fecha` convirtiendo los trimestres en fechas estandarizadas (01/01, 01/04, 01/07, 01/10) para permitir análisis cronológicos.
* **Validación de Métricas:** Se verificó que la columna `Total` (Tasa de paro) se importara como número decimal, permitiendo cálculos de agregación.

### 2.3. Finalización del ETL
* **Renombrado de Campos:** Se cambió `Total` por `Tasa de Paro` para mejorar la legibilidad del modelo de datos.
* **Carga:** Se aplicó "Cerrar y Aplicar" para trasladar los datos limpios al modelo relacional de Power BI.

---

## Fase 3: Modelado de Datos (Arquitectura)

> *Nota: En esta fase integraremos todas las tablas descargadas.*

### 3.1. Creación de Dimensiones Maestras
* **Dim_Calendario:** Tabla generada para unificar las fechas de EPA (trimestral), IPC (mensual) y Censo (anual).
* **Dim_Geografia:** Tabla de referencia para provincias y Comunidades Autónomas para evitar duplicidades de nombres.

### 3.2. Integración de Hechos (Fact Tables)
* Relación entre `Fact_EPA`, `Fact_IPC` y `Fact_Censo` a través de las Dimensiones Maestras.

---

## Fase 4: Visualización y Storytelling

### 4.1. Análisis de Desempleo (EPA)
* **Gráfico de Líneas:** Implementación de la evolución temporal de la Tasa de Paro (Promedio) filtrado por "Total Nacional".
* **Validación:** Se confirma la consistencia de los datos históricos (2016-2025).

---

## Registro de Problemas y Soluciones (Log)
