# Entrega práctica 4 — Streaming

- **Nombre:** Joaquin Santos
- **student_id:** alumno99
- **Escala:** small

Respondé cada pregunta en una a tres oraciones. Cuando se pide un dato, copiá el valor que te dio el notebook.

## Log y offsets

**1.** En el ejemplo de python la cola tenía inicialmente 5 mensajes, al leerlos quedaron 2 en ella y fueron leidos 3. La diferencia que tiene una de estilo log respecto a esta es que únicamente se realiza lectura, no se remueven los elementos leidos.

**2.** Luego de la llegada 2 *antifraude* leyó 180 mensajes y *reportes* 0. Este atraso de *reportes* no afecta de ninguna manera a *antifraude* ya que son independientes, cada uno guarda por separado cuál fue el último mensaje que procesó (offset) y puede retormarlos desde ese punto.

**3.** El offset es un valor que indica una posición respecto a un punto, en este caso la posición del mensaje a leer respecto del primero. Los offsets en está práctica se guardan en */Volumes/workspace/bigdata_alumno99/landing/streaming_lab/log/checkpoints* donde se encuentran uno por consumidor.

**4.** El replay leyó 280 mensajes. La arquitectura Kappa para procesar datos propone tener un único sistema para procesar todo en lugar de uno para batch y otro para streaming que propone Lambda. En el caso de reprocesar los datos al ser un log append-only es posible releer los mensajes desde el punto inicial o el que se requiera en lugar del checkpoint guardado.

## Ingesta y enriquecimiento

**5.** En cada llegada de la 1 a la 5 Bronze procesó 60, 120, 40, 40 y 20 filas. Al reejecutarlo sin archivos nuevos no procesó ninguna fila. El Auto Loader se encarga de leer los archivos, recordando aquellos que ya leyó para no hacerlo nuevamente.

**6.** Los motivos de rechazo en cuarentena son *INVALID_AMOUNT* y *UNKNOWN_CUSTOMER* en *stream_002* y para cada uno de los motivos son 20 filas las afectadas. Bronze guarda el texto original ya que de esta manera se conserva el dato original en caso de que cambie el contrato en el futuro.

**7.** Se trate de un *JOIN stream-tabla*. Las columnas agregadas por el join son *country* y *category*.

## Tiempo y ventanas

**8.** Event time quiere decir cuando se produjo el evento y Processing time cuando llego al sistema. En el ejemplo hay 2 eventos en particular, llamados late_ok y late_bad. El primero es un evento posterior al watermark pero que llegó más tarde que otros mensajes y se supone que corresponde a la ventana anterior, en cambio late_bad llego demasiado tarde (event time previo al watermark) y se descarta.

**9.** El watermark en cada llegada valía el event time máximo en la llegada menos 10 minutos, es decir:
|llegada|max_event  |watermark  |
|:---:  |:---:      |:---:      |
|1      |12:04      |11:54      |
|2      |12:08      |11:58      |
|3      |12:26      |12:16      |
|4      |12:27      |12:17      |
|5      |12:45      |12:35      |

**10.** Las primeras ventanas en modo Tumbling en modo append aparecieron recién en la llegada 3. No sucedió antes ya que se emite únicamente cuando se cierra alguna ventana y esto no sucedió en las primeras 2 (watermark de la llegada 2 es 11:58), para la tercera el valor del watermark ya era 12:16, cerrando las ventanas 12:00-12:05, 12:05-12:10 y 12:10-12:15. 

**11.** late_bad fue descartado ya que llegó muy tarde, su event time era anterior al watermark de la ventana. late_ok fue aceptado en una llegada posterior ya que su event time aún era posterior al watermark.

**12.** 
 - Tumbling: Son ventanas fijas que se suceden y no se solapan. Por ejemplo podría usarse para un conteo total de ventas en bloques de tiempo exactos.
 - Hopping: Son ventana de ancho fijo que avanzan con un salto menor que su tamaño por lo que se solapan. Por ejemplo para detectar picos de tráfico en una página con ventanas de 10 minutos cada 5 minutos.
 - Session: Son ventanas dinámicas que permanecen abiertas mientras sigan llegando eventos dentro de un margen de tiempo. Por ejemplo la sesión de un usuario en una página, se puede considerar abierta la ventana mientras el usuario interactúe hasta que esté inactivo por 5 minutos.

**13.** Se emitió en modo Update dos veces, en la llegada 1 con 40 compras y un monto total de $6000, y en la llegada 2 con 60 compras y un monto total de $8400. Y se emitió en modo Append una sola vez, en la llegada 3 al cerrarse la ventana con el valor acumulado de 60 compras y un monto total de $8400. La ventaja del Update es que los resultados son inmediatos, pero el consumidor debe estar preparado para procesar actualizaciones de filas recibidas anteriormente. La ventaja de Append es que garantiza consistencia emitiendo una sola vez el valor definitivo, pero hay que esperar a que cierre la ventana para obtener los resultados.

**14.** Lo que se gana al reducir el watermark es menor latencia ya que las ventanas se cierran más rapidamente, pero se pierde tolerancia a datos tardíos, mas eventos demorados querían por debajo del watermark.

## Garantías y fallas

**15.** Sin deduplicar quedaron 220 filas en el destino y deduplicando por *event_id* 200. El productor puede enviar un evento más de una vez ya que actúa bajo la garantía at-least-once en la que ante cualquier duda, reenvía el mensaje para evitar perdida de datos.

**16.** Al reiniciar con el mismo checkpoint se mantuvieron las 200 filas ya que con él sabe que datos ya procesó y no vuelve a escribir datos repetidos. Al perderlo se duplican en el destino (400 filas) ya que no sabe hasta donde procesó y comienza por el principio nuevamente. Con checkpoint se observa la garantía exactly-once y sin él se observa at-least-once.

**17.** Con MERGE no cambió el resultado ya que aunque se perdiera el checkpoint porque evalúa `t.event_id = s.event_id` y solo si no te cumple la condición (el registro no existe) lo inserta. Que la escritura sea idempotente quiere decir que por más que sea realice más de una vez el resultado no va a cambiar, en este caso teniendo como entrada los mismos eventos no duplico los resultados.

## CDC

**18.** Se leyeron 13 eventos de cambio en total de tipos *insert*, *update_preimage*, *update_postimage* y *delete*. Un `UPDATE` genera dos eventos ya que captura el antes y después del cambio. 

**19.** Si, es posible quedandose con la última version por customer_id y descartando aquellos cuyo último evento fue *delete*. La información que contiene el log y no la tabla son:
 - Un historial de modificaciones para todos los registros antes de llegar a su estado final.
 - Trazabilidad para registros que fueron eliminados.
 - Metadatos del cambio como *_commit_version*, *_commit_timestamp* y *_change_type*.

## Cierre

**20.** Cuándo elegir cada uno de los motores depende del caso de uso. En general, Spark Structured Streaming es ideal cuando se busca unificar el código e infraestructura batch y streaming en un mismo ecosistema. Apache Flink se elige para procesamiento evento por evento en tiempo real que exige latencias de milisegundos y un manejo complejo de estado a gran escala. Por otro lado, Kafka Streams es preferido para arquitecturas de microservicios basadas en Kafka que necesitan una solución liviana sin el costo operativo de desplegar un clúster independiente.

Sus ventajas principales en ese mismo orden son: la fácil integración al poder compartir lógica de batch con streaming, el estar diseñado como stream-first con latencias muy bajas, y la simplicidad arquitectónica de funcionar directamente como una biblioteca.
