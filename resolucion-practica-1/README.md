# Resolución Práctica 1

Nombre: `Joaquin Santos`

student_id: `alumno99`

### 01 Ingesta  bronze
---
1) Observaciones:
- En csv/json los datos no tienen un esquema estricto, estos son semi-estructurados. Aunque es posible intentar inferir el esquema requiere una lectura de todos los datos (costoso computacionalmente) y además los resultados pueden ser incorrectos si los datos tienen alguna anomalía como en el caso de Amount que fue inferido como texto de todos modos. En cambio, parquet y delta son estructurados y guardan un esquema estricto junto con los datos.
- En parquet y delta (que internamente utiliza parquet) son formatos orientados a columnas, en cambio csv y json no lo son.
- El formato delta contiene metadatos y propiedades adicionales sobre parquet. Por ejemplo, tiene métricas como numFiles y sizeInBytes, y optimizaciones como el codec utilizado para comprimir los datos y enableDeletionVectors.

2) 5V en Big Data
- Volumen: Con el tamaño `small` utilizado para la práctica que parece ser un subconjunto de las tablas se ingeren 5000, 500, 50011 y 200000 registros entre los 4 archivos de entrada.
- Velocidad: Eventos y probablemente transactions también parecen ser flujos de datos que sería interesante poder capturar en tiempo real.
- Variedad: Los datos se encuentran en 4 archivos y en 3 formatos distintos, csv, json y parquet.
- Veracidad: Transactions contiene 50011 filas pero, luego de un diagnóstico de calidad, puedo observar que solo 50000 son valores únicos y 52 contienen valores de `Amount` inválidos.
- Valor: Todas las fuentes de datos contienen información valiosa que cuando se puede concentrar y establecer conexiones, permite realizar un análisis mucho más rico y completo.

### 02 Desafío
---
La detección de un cambio de esquema es un proceso en el cual se identifica que la estructura de los datos cambio. En este caso events ahora contenía una columna más que originalmente no estaba, `app_version`, y hay un nuevo tipo de event, `refund`.
Aceptarla técnicamente es permitir procesar el cambio y fusionar o adaptar el esquema para poder guardar la nueva estructura de datos.
Decidir que es válido para el negocio, quiere decir que se valide que el cambio tiene sentido y aporta al negocio. Por ejemplo, puede ser importante conocer que versión de la aplicación que utilizaron los usuarios cuando realizarón cierta acción.