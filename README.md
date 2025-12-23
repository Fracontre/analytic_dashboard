# Dashboard Analítico en Power BI – Datos Abiertos

## 1. Objetivo del proyecto

El objetivo de este proyecto es **responder la siguiente pregunta analítica central**:

> ¿Cómo ha evolucionado el volumen de registros a lo largo del tiempo y cómo se distribuye territorialmente y por nivel, considerando variaciones interanuales, crecimiento acumulado y participación relativa?

El dashboard busca **apoyar el análisis exploratorio y comparativo**, permitiendo identificar:
- Tendencias temporales
- Diferencias territoriales
- Cambios estructurales entre niveles
- Patrones de crecimiento y concentración

Este proyecto utiliza exclusivamente datos abiertos de https://datosabiertos.mineduc.cl/titulados-en-educacion-superior/.

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

El dashboard se construye sobre un **modelo analítico de tipo estrella extendida**, en el cual **múltiples tablas de hechos temáticas** comparten dimensiones comunes (tiempo y territorio). Este diseño permite analizar el fenómeno desde distintos ejes sin perder coherencia ni rendimiento.

El modelo prioriza:
- Claridad semántica  
- Separación por dominio analítico  
- Escalabilidad y mantenibilidad  

### 4.1 Tablas de hechos

El modelo incorpora las siguientes **tablas de hechos**, todas con datos agregados y listas para análisis:

#### 4.1.1 `fact_titulados_anual`
- Granularidad: **Año**
- Métrica principal:
  - `total_titulados`
- Uso:
  - Análisis de tendencias temporales
  - Cálculo de crecimiento interanual (YoY)
  - Crecimiento acumulado

#### 4.1.2 `fact_titulados_region`
- Granularidad: **Región – Año**
- Métrica principal:
  - `total_titulados`
- Uso:
  - Comparaciones interregionales
  - Participación territorial
  - Rankings regionales

#### 4.1.3 `fact_titulados_provincia`
- Granularidad: **Provincia – Año**
- Métrica principal:
  - `total_titulados`
- Uso:
  - Análisis territorial intermedio
  - Identificación de polos provinciales

#### 4.1.4 `fact_titulados_comuna`
- Granularidad: **Comuna – Año**
- Métrica principal:
  - `total_titulados`
- Uso:
  - Análisis territorial fino
  - Rankings comunales
  - Detección de concentración local

#### 4.1.5 `fact_nivel_formacion`
- Granularidad: **Nivel de formación – Año**
- Métrica principal:
  - `total_titulados`
- Uso:
  - Comparación entre niveles formativos
  - Análisis de la composición estructural

#### 4.1.6 `fact_modalidad_jornada`
- Granularidad: **Modalidad / Jornada – Año**
- Métrica principal:
  - `total_titulados`
- Uso:
  - Comparaciones por modalidad académica
  - Análisis por tipo de jornada

#### 4.1.7 `fact_tipo_institucion`
- Granularidad: **Tipo de institución – Año**
- Métrica principal:
  - `total_titulados`
- Uso:
  - Análisis por tipo institucional
  - Participación relativa por categoría

#### 4.1.8 `fact_area_conocimiento`
- Granularidad: **Área de conocimiento – Año**
- Métrica principal:
  - `total_titulados`
- Uso:
  - Análisis disciplinar
  - Comparaciones entre áreas de estudio
  - Identificación de áreas predominantes

### 4.2 Tablas de dimensión

Las tablas de hechos se relacionan mediante **dimensiones compartidas**, garantizando consistencia analítica entre los distintos dominios.

#### 4.2.1 `dim_tiempo`
- `anio`
- `periodo`
- `fecha` (campo continuo)

Uso principal:
- Series temporales
- Cálculo de variación interanual (YoY)
- Crecimiento acumulado

#### 4.2.2 `dim_territorio`
- `region`
- `provincia`
- `comuna`

Estructura jerárquica:
Uso principal:
- Drill-down territorial
- Comparaciones geográficas

#### 4.2.3 Dimensiones analíticas específicas
Dependiendo de la tabla de hechos, se utilizan dimensiones adicionales como:
- `dim_nivel_formacion`
- `dim_modalidad`
- `dim_tipo_institucion`
- `dim_area_conocimiento`

Estas dimensiones permiten la segmentación semántica sin duplicar métricas.

### 4.3 Relaciones del modelo

- Tipo de relación: **1 a muchos**
- Dirección de filtro: **simple (single direction)**
- No se utilizan:
  - Relaciones bidireccionales
  - Relaciones ambiguas
  - Tablas puente innecesarias

Este diseño:
- Evita problemas de doble conteo
- Mejora el rendimiento del modelo
- Facilita la trazabilidad analítica

### 4.4 Justificación del diseño

Se opta por un modelo con **múltiples tablas de hechos temáticas** en lugar de una tabla única consolidada debido a que:

- Cada dominio analítico presenta granularidades distintas
- Se reduce la complejidad de filtros y visuales
- Se mejora la legibilidad del modelo
- Se facilita la extensión futura del dashboard

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
