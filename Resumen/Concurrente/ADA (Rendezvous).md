- Una aplicacio en ADA es un unico programa con un cuerpo principal, y para que trabaje concurrentemente se desarrollan TASK  dentro de ese programa que pueden ejecutar independientemente y contienen primitivas de sincronizacion/comunicacion
- Se pueden implementar directamente TASK o se puede declarar un TASK TYPE para crear varias instancias iguales de ese tipo (arreglo, puntero, instacia simple)
- Cada TASK o TASK TYPE tienen una especificacion donde se declaran las operaciones exportadas (llamadas ENTRYs) y un BODY donde se implmenta la tarea

SINTAXIS:
- Estructura:
![[Pasted image 20261107165107.png]]
- Las tareas pueden exportar opreaciones o no, si lo hacen deben definir lo ENTRYs correspondientes en la especificacion.
- Los ENTRYs pueden o no tener parametros, si los tiene se debe declarar el nombre, el tipo de datos y el tipo de parametro (IN, OUT o IN OUT)
![[Pasted image 20261107170811.png]]
- El cuerpo de las tareas es donde se implementa la tarea. En esta parte no hay diferencia entre las TASK y los TASK TYPE (no se pone TYPE)
![[Pasted image 20261107171311.png]]
- Si se hubiese definido un arreglo de un tipo de tarea (por ej. arregloT1 de la imagen anterior), se generaran varias tareas identicas de ese tipo (en el ej. anterior 10 taeras). La unica manera de identificar una de otra es por su posicion en el arreglo, pero la tarea por si misma no conoce ese dato. Si lo necesita saber, otra tarea o el cuerpo principal del programa se lo debera comunicar para que guarde ese "identificador" en una variable.

COMUNICACION/SINCRONIZACION
- Rendezvous (encuentro): Es el principal mecanismo de sincronizacion en ADA y tambien es el mecanismo de comunicacion primario
- ADA tiene otros mecanismos de comunicacion QUE NO PUEDEN ser utilizados en la materia, como por ejemplo el uso de variables compartidas
- La comunicacion (el encuentro) entre dos tareas se realiza por medio de un Entry Call por parte del proceso que realiza el "pedido" y un Accept por parte del que "sirve" (resuelve) ese "pedido"

- Hay 3 formas de Entry Call que se diferencian por el tiempo de demora
	- Simple: La ejecucion demora al llamador hasta que la otra tarea termine de ejecutar el accept correspondiente 
	![[Pasted image 20261107172525.png]]
	- Condicional (select - else): No espera a que le acepten el pedido, si la otra tarea no esta lista para realizar el accept a su pedido inmediatamente, entonces lo cancela y realiza otra cosa
	![[Pasted image 20261107172838.png]]
	- Temporal (select - or delay): Espera a lo sumo un tiempo a que la otra tarea realice el accept a su pedido, pasado el tiempo cancela el entry call y realiza otra cosa
	![[Pasted image 20261107173027.png]]

	ACCEPT
	- Por cada entry declarado en la especificacion de una tarea, en el cuerpo se debe hacer al menos un accept para dicho entry
	- Por cada entry existe, en la tarea, una cola implicita (no se puede manipular por la tarea) de entry calls pendientes
	- El accept puede tener un cuerpo, en cuyo caso hasta que no se termine de ejecutar el mismo no se pierde la sincronizacion con el entry call. O puede no tener un cuerpo, en cuyo caso inmediatamente termina la sincronizacion con el entry call
	- El acecpt demora la tarea hasta que haya al menos un entry call en la cola implicita asociada; saca al primero de ellos y, dependiendo del tipo de accept hace:
		- Si no tiene cuerpo: Termina inmediatamente el accept y abmas tareas continuan con su ejecucion
		- Si tiene cuerpo: Copia los parametros reales del llamado en los parametros formales (si el entry no tuviese parametros no haria este paso); ejecuta las sentencias que estan en el cuerpo; cuando termina los parametros formales de salida son copiados a los parametros realies (si tuviese parametros de tipo OUT o IN OUT). Luego ambas taeras continuan con su ejecucion
		![[Pasted image 20261107173928.png]]
		- ADA brinda la posibilidad de implementar wait selectivos por medio de comunicacion guardada (like PMS). Select para los accept. Permite tener varias alternativas de accept que pueden o no tener asociada una condicion booleana (no se puede utilizar los parametros del entry como parte de la condicion) utilizando la clausula when
		![[Pasted image 20261107174141.png|628]]
		- A su vez se le puede agregar al final un ELSE o un OR DELAY que actua de forma similar al de los entry call:
			- En el caso del ELSE, si no se puede ejecutar ninguna alternativa en forma inmediata, entonces se ejecuta el conjunto de sentencias asociadas al ELSE
			- En el caso del OR DELAY, espera a lo sumo un tiempo a poder aceptar alguna alternativa, si no lo logra ejecuta el conjunto de sentencias asocaiadas al OR DELAY
			![[Pasted image 20261107174506.png]]
	- Cada entry tiene asociado un atributo que puede ser consultado SOLO por la tarea a la que pertenece el entry que indica la cantidad de entry call pendientes (cantidad de elementos de la cola impicita asociada al entry): NombreDelEntry'count
	- Este atributo puede ser usado en los when, por ejemplo para dar prioridad a un entry sobre otro. Como veremos en alguno de los ejercicios
	- Algo importante para tener en cuenta es que NO TIENE SENTIDO usarlo en el when de un accept dentro de un select para comprobar que haya un entry call pendiente. Por ejemplo: when (nombreE1'count>0) => accept nombreE1
	 Ya que si no hay un pedido pendiente (es decir que nombreE1'count = 0) esta alternativa no podra ser elegida por el momento ya que no se puede hacer el accept inmediatamente. Y ademas esto podria llevar a producir BUSY WAITING si se hace lo mismo en todas las alternativas del select


EJERCICIOS
1) Se debe modelar la atención en un banco por medio de un único empleado. Los clientes llegan y son atendidos de acuerdo al orden de llegada.
	![[Pasted image 20261107233541.png]]
	Se instancian las tareas cliente (task type), empleado y un arreglo de clientes. El cliente le hace un pedido al empleado y el empleado esta en un loop infinito esperando pedidos en el accept y resolviendolos dentro del accept (le manda la respuesta en los parametros de Pedido)   
2) Se debe modelar la atención en un banco por medio de un único empleado. Los clientes llegan y esperan a lo sumo 10 minutos a ser atendido de acuerdo al orden de llegada. Pasado ese tiempo se retira sin ser atendido![[Pasted image 20261107233841.png]]
	Misma solucion que el punto anterior, ahora en cliente se le agrega el SELECT con la condicion de que si pasan 600 segundos vaya por otro camino en el que no se hace nada y luego finaliza
3) Se debe modelar la atención en un banco por medio de un único empleado. Los clientes llegan y esperan a ser atendidos; pueden ser Regulares o Prioritarios. El empleado los atiende de acuerdo al orden de llegada pero dando prioridad a los clientes Prioritarios
	![[Pasted image 20261108000712.png]]
	Se hacen dos entrys en empleado, uno para prioritarios y otro para regulares. Dentro de cliente se solicita el llamado la atencion de prioritario o de regular segun lo que sean y dentro de empleado se tiene un select no deterministico en el que se le da prioridad a los prioritarios. Se consigue esta prioridad haciendo que el accept de regulares se de unicamente si los prioritarios tienen un cantidad de 0 llamados
4) Se debe modelar la atención en un banco por medio de DOS empleados. Los clientes llegan y son atendidos de acuerdo al orden de llegada por cualquiera de los dos.
	![[Pasted image 20261108001539.png]]
	Se crea un nuevo proceso administrador que va a tener los entrys pedido (para que el cliente solicite atencion) y siguiente (para que el empleado avise que esta libre). En su cuerpo primero acepta la solicitud del empleado libre y luego acepta la solicitud de atencion del cliente, pasandole los datos de una solicitud a otra (Esto se hace para no tener que almacenar en una cola los datos). Despues empleado y cliente funcionan practicamente igual
5) Se debe modelar un buscador para contar la cantidad de veces que aparece un número dentro de un vector distribuido entre las N tareas contador. Además existe un administrador que decide el número que se desea buscar y se lo envía a los N contadores para que lo busquen en la parte del vector que poseen, y calcula la cantidad total.
	![[Pasted image 20261108002811.png]]
	Se tiene la tarea admin y va a haber un arreglo de tareas contador. Admin tiene los entrys valor (asigna el valor a buscar) y resultado (toma el resultado del contador y lo suma en el total) y se va a quedar loopeando la cantidad de contadores * 2 aceptando no deterministicamente cualquiera de estas dos opciones. Por otro lado contador lo primero que hace es pedirle el valor a buscar a admin mediante la entry valor, busca la cantidad de apariciones de ese valor en su vector y le solicita enviar la cantidad de veces que lo encontro al admin mediante la entry Resultado