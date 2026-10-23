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


#### Tipos de Datos Asignados y Justificación Técnica

Para maximizar la eficiencia de compresión en memoria y evitar errores de cálculo en las medidas DAX, se definieron estrictamente los tipos de datos en Power Query bajo los siguientes criterios de negocio:

##### 1. Identificadores y Claves (IDs) ➔ Tipo: Texto (`Text`) o Número Entero (`Whole Number`)
*   **Columnas:** `codigo_cliente`, `codigo_operacion`, `codigo_producto`.
*   **Justificación:** Se configuran como **Texto** (o Número Entero si son puramente numéricos secuenciales sin letras ni ceros a la izquierda) debido a que sobre estas columnas **nunca se realizarán operaciones matemáticas** (como sumas o promedios). Definirlos correctamente evita que el motor intente indexarlos como métricas continuas, acelerando los tiempos de respuesta en los joins de las relaciones.

##### 2. Fechas ➔ Tipo: Fecha (`Date`)
*   **Columnas:** `fecha_alta` (en `D_Clientes`) y `fecha_venta` (en `D_Transaccion`).
*   **Justificación:** Se eliminó el componente de hora (evitando el tipo *Date/Time*) para reducir la cardinalidad de la columna, lo que optimiza drásticamente la compresión del modelo en Power BI. Además, el tipo **Date** permite que las columnas sean 100% compatibles con las funciones de Inteligencia de Tiempo en DAX (como `SAMEPERIODLASTYEAR` o `TOTALYTD`).

##### 3. Valores Cuantitativos y Métricas ➔ Tipo: Número Entero (`Whole Number`)
*   **Columnas:** `cantidad`.
*   **Justificación:** Representa unidades físicas absolutas y enteras vendidas por transacción. Al no requerir posiciones decimales, el tipo **Número Entero** consume menos almacenamiento y optimiza los cálculos de agregación sumatorios.

##### 4. Montos, Precios y Porcentajes ➔ Tipo: Número Decimal Fijo / Moneda (`Decimal / Fixed Decimal Number`)
*   **Columnas:** `precio_unitario`, `total_venta`, `descuento(%)`.
*   **Justificación:** Las variables monetarias exigen máxima precisión financiera para evitar errores de redondeo acumulados en reportes consolidados. Se utiliza **Número Decimal Fijo (Currency)** para los montos en dinero debido a que congela exactamente 4 dígitos decimales de manera interna, garantizando un rendimiento de cálculo superior al del decimal flotante tradicional. Los porcentajes se definen como **Número Decimal** estándar (formateados luego como `%` en la vista de reporte).

##### 5. Atributos Categóricos ➔ Tipo: Texto (`Text`)
*   **Columnas:** `nombre_apellido`, `email`, `ciudad`, `provincia`, `segmento`, `estado`, `canal_venta`, `moneda`, `descripcion`.
*   **Justificación:** Columnas cualitativas destinadas exclusivamente a la segmentación, etiquetado y filtrado de reportes en los objetos visuales (matrices, gráficos de barras, segmentadores).


#### Tratamiento de Valores Nulos y Duplicados (Calidad de Datos)

Durante la fase de ETL en Power Query, se aplicaron reglas estrictas de limpieza de datos para corregir registros inconsistentes, asegurando la integridad referencial del modelo estrella sin alterar las métricas de negocio.

##### 1. Tratamiento de Duplicados
*   **En Tablas de Dimensiones (`D_Clientes` y `D_Transaccion`):** 
    *   **Acción:** Eliminación total de registros duplicados basados en sus Claves Primarias (`codigo_cliente` y `codigo_operacion`).
    *   **Justificación Técnica:** Las dimensiones actúan como tablas maestras de búsqueda. Mantener filas repetidas rompe la restricción de unicidad, lo que impedía a Power BI establecer la relación ideal de 1 a Varios ($1:\star$) y forzaba relaciones incorrectas de Muchos a Muchos. Su eliminación garantiza que cada entidad descriptiva sea única.
*   **En Tabla de Hechos (`F_Ventas`):**
    *   **Acción:** Se preservaron las filas que compartían el mismo `codigo_operacion`.
    *   **Justificación Técnica:** Una misma factura u operación puede contener múltiples líneas de venta (distintos productos, cantidades o precios). Eliminar estos registros habría significado una pérdida crítica de ingresos reales y volumen en las métricas consolidadas.

##### 2. Tratamiento de Valores Nulos (`null`) o Vacíos
La estrategia se dividió entre **eliminación** (para claves estructurales) y **reemplazo por defecto** (para atributos descriptivos) según su impacto en el modelo:

*   **Claves Primarias y Foráneas (`codigo_cliente`, `codigo_operacion`) ➔ Eliminación**
    *   **Acción:** Se filtraron y eliminaron las filas donde las claves de unión eran nulas.
    *   **Justificación Técnica:** Si una fila de la dimensión no tiene código, no puede enlazarse con los hechos. Si un hecho no tiene código, se rompe la integridad referencial dejando registros "huérfanos". Al ser campos críticos de indexación, la eliminación es la única vía para evitar errores de consistencia en el motor analítico.
*   **Atributos Categóricos Descriptivos (ej. `email`, `telefono`, `segmento`) ➔ Reemplazo con Valor por Defecto**
    *   **Acción:** Se reemplazaron los valores nulos por textos estandarizados como `"Sin Especificar"` o `"No Registrado"`.
    *   **Justificación Técnica:** Borrar a un cliente completo solo porque no dejó su correo o teléfono destruiría el historial de sus compras en la tabla de hechos (subestimando las ventas totales). El reemplazo por un valor por defecto mantiene la fila activa, permitiendo reportar la métrica financiera y agrupar los datos faltantes bajo una categoría limpia en los gráficos, facilitando además auditorías internas de datos.
*   **Métricas Cuantitativas (ej. `descuento(%)`) ➔ Reemplazo con Valor Numérico**
    *   **Acción:** Se imputaron los valores nulos con el número `0`.
    *   **Justificación Técnica:** Los campos numéricos con valores `null` bloquean o desvían los cálculos aritméticos y las agregaciones automáticas en DAX (como la función `SUM` o promedios). Reemplazar por `0` asegura la continuidad matemática de las medidas sin alterar el resultado real de la operación.
