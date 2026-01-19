# Bitácora de Desarrollo: Informe INE

Este documento registra los pasos técnicos, decisiones de diseño y fuentes de datos utilizadas para la creación del reporte en Power BI.

---

## Fase 1: Extracción de Datos (Data Sourcing)

### 1.1. Encuesta de Población Activa (EPA) - Tasas de Paro
* **Tabla seleccionada:** Tasas de paro por distintos grupos de edad, sexo y comunidad autónoma.
* **Ruta de acceso:** `INEbase / Mercado laboral / Actividad, ocupación y paro / Encuesta de población activa / Resultados / Trimestrales / Series desde 2002 / Resultados por comunidades autónomas / Parados`.
* **Rango Temporal:** 2016T1 - 2025T4 (40 trimestres/10 años).
* **Dimensiones extraídas (en orden):** 1. Sexo (Ambos sexos, Hombres, Mujeres).
    2. Comunidades y Ciudades Autónomas (Nacional + 19 regiones).
    3. Edad (Total, Menores de 25 años, 25 y más años).
* **Formato original:** CSV (Delimitado por punto y coma `;`).

---

## Fase 2: ETL y Transformación (Power Query)

> *Nota: Aquí documentaremos los pasos aplicados en el editor de Power Query.*

---

## Fase 3: Modelado de Datos



---

## Fase 4: Visualización y Storytelling



---

## Registro de Problemas y Soluciones (Log)
