# Proyecto de Inteligencia de Negocios - Caso Bike Z

## 1. Descripción del Proyecto
**Bike Z** es una compañía dedicada a la comercialización de artículos para motocross que ha experimentado un rápido crecimiento durante los últimos años. Este repositorio contiene la solución integral de Inteligencia de Negocios (BI) diseñada para apoyar a la gerencia en el proceso de toma de decisiones estratégicas.

La primera fase del proyecto abarca el modelamiento dimensional en SQL Server y el ETL en SQL Server Integration Services (SSIS).

---

## 2. Estructura del Repositorio

```text
CERTAMEN2-BI/ 
|-- README.md                  # Documentación general y Fase 1 del proyecto
|-- docs/ 
|   |-- modelo-dimensional/    # Diagrama del modelo Estrella
|   `-- presentacion/          # Presentación del proyecto 
|-- database/ 
|   |-- scripts/               # Scripts DDL (CREATE TABLE) y consultas de extracción
|   `-- backup/                # Respaldos (.bak) de la BD transaccional y dimensional
|-- etl/ 
|   `-- proyecto-ssis/         # Solución de Visual Studio con paquetes SSIS (.dtsx)
```

---

## 3. Fase 1: Análisis del Caso y Definición de KPIs

A partir de la base de datos transaccional de origen se definieron los siguientes Indicadores KPI:

1. **Ingresos Totales por Ventas (Crecimiento Comercial):**
   * **Métrica:** `SUM(cantidad * precio_unitario)` segmentado por tiempo (mensual, trimestral, anual).
   * **Utilidad:** Permite monitorear la evolución financiera de Bike Z y evaluar su ritmo real de expansión comercial.
2. **Rendimiento por Categoría de Producto:**
   * **Métrica:** Volumen de unidades vendidas y monto facturado por categoría, producto y color de artículo (`DIM_PRODUCTO`).
   * **Utilidad:** Identifica los artículos de motocross líderes en ventas para optimizar compras y reposición de inventario.
3. **Desempeño Geográfico por Territorio y País:**
   * **Métrica:** Ingresos acumulados y demanda por grupo geográfico, país y territorio (`DIM_TERRITORIO`).
   * **Utilidad:** Focaliza las estrategias de marketing y distribución en los mercados de mayor rentabilidad.
4. **Productividad de la Fuerza de Ventas (Empleados):**
   * **Métrica:** Monto total generado y cantidad de órdenes gestionadas por cada vendedor (`DIM_EMPLEADO`).
   * **Utilidad:** Evalúa objetivamente el rendimiento del equipo comercial para planes de incentivos y capacitación.
5. **Valor y Recurrencia por Cliente:**
   * **Métrica:** Monto facturado acumulado y volumen de unidades adquiridas por cliente registrado (`DIM_CLIENTE`).
   * **Utilidad:** Sustenta campañas de fidelización y retención dirigidas a los compradores estratégicos.

---

## 4. Diseño del Modelo Dimensional

Se diseñó un modelo dimensional bajo un **Esquema en Estrella**:

### Proceso de negocio
El Proceso de negocio es de ventas de artículos de motocross, soportado operativamente por las tablas `Ventas` y `DetalleVentas`.

### Nivel de granularidad
La granularidad es **una fila por cada línea de detalle de producto dentro de una transacción de venta**, identificada mediante la llave primaria compuesta `(cod_venta, cod_detalle_venta)`. 

### Dimensiones
Se definieron cuatro tablas de dimensiones desnormalizadas con llaves subrogadas autoincrementales (`IDENTITY(1,1)`):
* **`DIM_PRODUCTO`**: `sk_producto` (PK), `cod_producto`, `nom_producto`, `nom_color`, `cod_categoria`, `nom_categoria`.
* **`DIM_CLIENTE`**: `sk_cliente` (PK), `cod_cliente`, `nom_cliente`, `nom_apellido`.
* **`DIM_EMPLEADO`**: `sk_empleado` (PK), `cod_empleado`, `nom_cargo`, `fec_nacimiento`, `cod_estado_civil`, `cod_genero`, `fec_contratacion`, `num_horas_vacaciones`, `cod_territorio`.
* **`DIM_TERRITORIO`**: `sk_territorio` (PK), `cod_territorio`, `nom_territorio`, `nom_grupo`, `cod_pais`, `nom_pais`.

### Medidas de la tabla de hechos
La tabla central es **`FACT_VENTAS_BIKEZ`**, conformada por:
* **llave Primaria Compuesta:** `cod_venta`, `cod_detalle_venta`.
* **llaves Foráneas Subrogadas:** `sk_producto`, `sk_cliente`, `sk_empleado`, `sk_territorio`.
* **Atributos Temporales:** `fec_venta` (`DATETIME`), `fec_envio` (`DATETIME`).
* **Medidas:**
  * `cantidad` (`INT`) 
  * `precio_unitario` (`DECIMAL(18,2)`) 
  * `monto_total` (`DECIMAL(18,2)`) (`cantidad * precio_unitario`).

---