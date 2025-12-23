# Dashboard Analítico en Power BI – Datos Abiertos

## 1. Objetivo del proyecto

El objetivo de este proyecto es **responder la siguiente pregunta analítica central**:

> ¿Cómo ha evolucionado el volumen de registros a lo largo del tiempo y cómo se distribuye territorialmente y por nivel, considerando variaciones interanuales, crecimiento acumulado y participación relativa?

El dashboard busca **apoyar el análisis exploratorio y comparativo**, permitiendo identificar:
- Tendencias temporales
- Diferencias territoriales
- Cambios estructurales entre niveles
- Patrones de crecimiento y concentración

Este proyecto tiene fines **demostrativos y de evaluación técnica**, y no representa información institucional real.

---

## 2. Fuentes de datos

- **Tipo de datos**: Datos abiertos
- **Formato original**: CSV
- **Características generales**:
  - Registros agregados por periodo
  - Variables temporales (año / periodo)
  - Variables territoriales (región, provincia, comuna)
  - Variables de clasificación por nivel

No se utilizan:
- Datos privados
- Credenciales
- Gateways
- Conexiones institucionales

---

## 3. Proceso de limpieza y transformación de datos

La preparación de datos se realiza en el notebook **`Panel.ipynb`**, utilizando Python.

### 3.1 Etapas principales

1. **Carga de datos**
   - Lectura de archivos CSV
   - Validación de tipos de datos

2. **Limpieza**
   - Normalización de nombres de columnas
   - Manejo de valores nulos
   - Validación de claves temporales y territoriales

3. **Transformación**
   - Construcción de variables de periodo
   - Agregaciones necesarias para análisis
   - Preparación de tablas dimensionales

4. **Exportación**
   - Generación de datasets listos para Power BI
   - Separación lógica entre hechos y dimensiones

El resultado es un conjunto de tablas optimizadas para un **modelo dimensional tipo estrella**.

---

## 4. Modelo analítico (Power BI)

El dashboard utiliza un **modelo en estrella**, compuesto por:

### 4.1 Tabla de hechos
- `fact_registros`
  - Métrica principal: total de registros
  - Claves de tiempo y territorio

### 4.2 Tablas de dimensión
- `dim_tiempo`
  - Año
  - Periodo
- `dim_territorio`
  - Región
  - Provincia
  - Comuna
- `dim_nivel`
  - Nivel global
  - Subcategorías

Las relaciones son:
- 1 a muchos
- Dirección de filtro simple
- Sin ambigüedades ni relaciones bidireccionales innecesarias

---

## 5. Medidas DAX principales

Las métricas fueron construidas siguiendo buenas prácticas de **DAX explícito**, evitando columnas calculadas innecesarias.

### 5.1 Métrica base

```DAX
Total Registros =
SUM ( fact_registros[total] )


Variación YoY =
VAR ValorActual =
    [Total Registros]
VAR ValorPrevio =
    CALCULATE (
        [Total Registros],
        DATEADD ( dim_tiempo[fecha], -1, YEAR )
    )
RETURN
    ValorActual - ValorPrevio


Crecimiento YoY (%) =
DIVIDE (
    [Variación YoY],
    CALCULATE (
        [Total Registros],
        DATEADD ( dim_tiempo[fecha], -1, YEAR )
    )
)


Crecimiento Acumulado =
VAR PrimerPeriodo =
    CALCULATE (
        [Total Registros],
        FIRSTDATE ( dim_tiempo[fecha] )
    )
RETURN
    [Total Registros] - PrimerPeriodo


Participación (%) =
DIVIDE (
    [Total Registros],
    CALCULATE (
        [Total Registros],
        ALL ( dim_territorio )
    )
)
