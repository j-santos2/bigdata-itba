# Entrega práctica 3 — Bases de datos NoSQL

- **Nombre:** Joaquin Santos
- **student_id:** alumno99
- **Escala:** small

Respondé cada pregunta en una a tres oraciones. Cuando se pide un dato, copiá el valor que te dio el notebook.

## CAP y PACELC

**1.**  Con la red partida la replica C no respondió en modo CP y respondió *100* en modo AP. En el modo CP se sacrifica la disponibilidad, al estar partido no devuelve valores potencialmente incorrectos. En cambio, en el modo AP es un mejor esfuerzo y responde un valor inconsistente.

**2.** En el modo AP luego de la reparación el valor de saldo es 80 y se pierde el que aún tenía C (100).

**3.** Según el gráfico de latencia la p99 con `W=1` es 9.6ms y para `W=3` es 55.6ms. La ventaja en `W=3` es que tengo más certeza de que el valor es consistente, en este caso confirman todos los nodos el valor antes de la respuesta. Si se utilizan menos, se pueden tener respuestas más rápidas al costo de retornar resultados inconsistentes.

**4.** En modo CP al partir las replicas en ABC y DE solo se pudo seguir escribiendo desde el lado ABC. Esto sucede ya que para escribir se requiere a la mayoría de los nodos, en este caso 3 de 5.

## Clave-valor

**5.** El GET en memoria demoró 0.0003ms y el GET a través de Spark 499ms, aproximadamente 1600 veces más rápido. Redis guarda los datos en memoria ya que su proposito es resolver busquedas rapidamente, obviamente al no volcar esta información en disco si se cae el servicio se pierde la información y es más caro este espacio que el de disco.

**6.** Para contar los clientes de AR hubo que leer todos los valores y parsearlos ya que redis (y otras bases clave-valor) solo almacenan datos sin semántica como valor para una clave. La ventaja de esto es que las busquedas por clave son en tiempo constante O(1) pero si se requiere leer datos usando otro campo o filtrar es necesario pasar por todos los registros.

**7.** Al pasar de 4 a 5 nodos un 79.6% de los nodos se tuvieron que mover. En cambio para hashing consistente solo un 21.2% de los nodos, una mejora importante sobre el primer método. Es conveniente usar el segundo método ya que agregar nodos implica reubicar una cantidad mucho menor de registros.

## Documental

**8.** La consulta relacional del cliente 42 devolvió 11 filas y la consulta documental solo 1 documento. Dentro del documento están representados las transacciones y productos además de los datos del cliente, que en el modelo relacional corresponde a tablas distintas que deben unirse con JOINs.

**9.** Spark cuando alguno de los campos no está le asigna *NULL*, perdiendo información ya que no es posible diferenciar un valor *NULL* de uno ausente. Se dice que el modelo documental tiene un esquema flexible ya que es capaz de tener registros con distintos campos, algunos campos pueden estar presentes para algunos registros pero ausentes para otros.

**10.** El documento más grande tiene 5.2KB y el límite que admite MongoDB como máximo es 16MB. Cuando un documento sin límite crece, al llegar a este tamaño Mongo no permitira que se siga actualizando.

**11.** El modelo documental resuelve bien consultas dentro del mismo documento ya que a diferencia del relacional no requiere hacer joins para obtener la información si estas son de 1 a muchos. En cambio, consultas de agregación o que contienen relaciones de N a M, son más costosas ya que requieren acceder a los documentos y desanidar la información en los documentos.

## Grafos

**12.** El grafo tiene 2361 nodos *DEVICES*, 5000 nodos *CUSTOMER* y 5400 aristas *USES*.

**13.** Para el patrón de 2 saltos SQL necesitó 3 joins y para el de 4 saltos 5 joins. Esto es así ya que cada relación (arista) que atraviesa debe ser un *JOIN*. Neo4j es más eficiente ya que en las bases de grafos cada nodo guarda las relaciones hacia otros nodos directamente, en cambio SQL unirlos con claves.

**14.** La busqueda de componentes conexos necesitó 14 iteraciones y el componente más grande tiene 18 nodos.

**15.** Al repartir los nodos con *hash (id) mod 4* un 74.4% de las aristas están cortadas, es decir que sus nodos se encuentran en máquinas distintas. Es muy difícil balancear este tipo de modelos ya que se basan en las relaciones y normalmente no son balanceadas. En el caso de una red social por ejemplo celebridades pueden ser seguidas por millones de personas cuando normalmente una persona no pasará de los cientos o miles, además cuando uno quiere recorrer amigos de amigos por ejemplo con cada iteración se agregan muchos más nodos al conjunto de resultado. Esto hace que sea muy difícil mantener todos ellos en la misma máquina.

## Vectorial

**16.** El vector de cada cliente representa los siguientes items: Proporción del gasto en cada categoría (electronics, home, books, sports, fashion), Proporción de pagos por canal (card, wallet, transfer), Ticket promedio (dividido por 2.000) y proporción de compras de más de 1.500. La similitud del coseno mide el ángulo o similitud entre 2 vectores.

**17.** Entre los marcados hay un 31.6% de vecinos marcados contra un 14.8% de base. Es útil para buscar para buscar vecinos parecidos ya que evalúa si un cliente tiene un comportamiento similar a otro.

**18.** Con *nprobe=4* se obtiene un recall de 0.872 con solo un 5.985% escaneado. Al utilizar este método de busqueda se pierde algo de precisión (0.872 de 1) pero se gana considerablemente en costo, solo se escanearon 5.985% de los vectores.

## Columnar

**19.** Los datos ocupan en 50MB CSV, 121.1MB en JSON y 5.3MB en Parquet. Sobre Parquet Spark solo necesito leer la columna Amount para la consulta ya que los datos se almacenan por campos (columnas), no por registros. Si los datos están ordenados pueden optimizarse muchas consultas ya que es capaz de evitar leer archivos en los que sabé que no puede encontrarse el registro, en el caso de `amount>1900` del ejemplo pueden saltearse los primeros 7 archivos y directamente buscar en el 8.

**20.** El formato columnar es muy bueno para analítica porque normalmente se realizan consultas de agregación con algunas de las columnas y no todas ellas. De esta forma es mucho menos costoso acceder a todos los datos de una columna en particular. Por esta misma razón no es conveniente para modificar filas, esto implica acceder a todas las columnas para modificar un valor en cada una de ellas, el modelo relacional es mucho más adecuado para esto.
