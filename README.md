# modulo-6-ventas_Power_BI
Normalización de la estructura Ventas y limpieza de datos

Pasos aplicados - justificación y detalle:

Primero modifiqué los nombres técnicos de las columnas con nombres de fácil interpretación, manteniendo la estructura snake_case.
Segundo cambié el tipo de columnas en aquellas que correspondía, para que el trabajo con las tablas no tuviera errores de interpretación en etapas próximas.
Tercero eliminé registros duplicados y luego las filas en blanco, las cuales no aportan valor y pueden generar conclusiones erróneas al hacer cálculos con los datos.
Luego dupliqué la tabla principal en 2 tablas más: 
 - a la primera le cambié el nombre a D_Clientes (Tabla de Dimensiones) dejando solamente las columnas con datos relevantes a los clientes
 - a la segunda le cambié el nombre a D_Transaccion (Tabla de Dimensiones) dejando solamente las columnas con datos relevantes a las transacciones realizadas.
A la tabla Ventas la llamé F_Ventas siendo la designada como Tabla de Hechos.
El criterio para separar las columnas en las tablas correspondientes fue determinar qué datos fueron considerados como atributos categóricos (utilizables como filtros y segmentaciones y cuáles fueron considerados como Medidas, que representan valores cuantitativos pertenecientes a las ventas.
