# modulo-6-ventas_Power_BI
Normalización de la estructura Ventas y limpieza de datos

### Correcciones y Optimización del Modelo de Datos

El modelo de datos inicial presentaba limitaciones de rendimiento y ambigüedades en el filtrado debido a una separación incompleta de las tablas tras el proceso de normalización. A continuación, se detalla el criterio completo de diseño aplicado para corregir la estructura hacia un **Modelo en Estrella (Star Schema)** óptimo.

Pasos aplicados - justificación y detalle:

Primero modifiqué los nombres técnicos de las columnas con nombres de fácil interpretación, manteniendo la estructura snake_case.
Segundo cambié el tipo de columnas en aquellas que correspondía, para que el trabajo con las tablas no tuviera errores de interpretación en etapas próximas.
Tercero eliminé registros duplicados y luego las filas en blanco, las cuales no aportan valor y pueden generar conclusiones erróneas al hacer cálculos con los datos.

#### Criterio de Separación: Dimensiones vs. Hechos
Para garantizar la escalabilidad del reporte y evitar relaciones incorrectas (como Muchos a Muchos o relaciones Uno a Uno bidireccionales), se dividieron los datos bajo el siguiente estándar de BI:

*   **Tablas de Dimensiones (`D_Clientes`, `D_Transaccion`):** Almacenan exclusivamente los **atributos categóricos y descriptivos** del negocio. Estas tablas actúan como entidades maestras donde cada registro es único (Claves Primarias sin duplicados), encargadas de filtrar el modelo.
*   **Tabla de Hechos (`F_Ventas`):** Almacena los **valores cuantitativos, métricas y hechos transaccionales** (como cantidades y montos), junto con las claves foráneas necesarias para conectar las dimensiones. Es la tabla que acumula el volumen histórico de operaciones.


#### Claves de Unión y Cardinalidad del Modelo

Las relaciones se configuraron con **cardinalidad de Uno a Varios (1:*)** y **dirección de filtro único**, asegurando que los filtros fluyan correctamente desde las dimensiones hacia la tabla de hechos:

1.  **Dimensión Clientes (`D_Clientes`) → Hechos Ventas (`F_Ventas`)**
    *   **Clave de unión:** `codigo_cliente`
    *   **Cardinalidad:** `1:*` (Un cliente único puede tener muchas ventas asociadas).
2.  **Dimensión Transacción (`D_Transaccion`) → Hechos Ventas (`F_Ventas`)**
    *   **Clave de unión:** `codigo_operacion`
    *   **Cardinalidad:** `1:*` (Una operación o factura cabecera puede contener múltiples registros o líneas de producto en la tabla de hechos).

Con esta corrección, se eliminó la redundancia de datos, se resolvió la restricción que impedía vincular la tabla de clientes y se optimizó el comportamiento del motor de almacenamiento (VertiPaq) de Power BI para el cálculo de medidas DAX.
