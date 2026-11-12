- Los programas se comoponen solo de procesos y canales. NO EXISTEN LAS VARIABLES COMPARTIDAS
- Los canales actuan como COLAS de mensajes enviados y no recibidos (Estilo MAILBOX, todos los procesos los pueden usar para enviar o recibir mensajes)
- Los procesos interactuan entre ellos UNICAMENTE por medio del envio de mensajes
- No se requiere sincronizacion por exclusion mutua ya que no existen las variables compartidas
- Sintaxis
	- Declaracion de canales: se deben declarar los canales indicando la estructura de datos de los mensajes, puede ser:
			Un canal: chan nombreCanal(tipoDato)
			Un arreglo: chan nombreArreglo[1..m] (tipoDato)
	- Comunicacion: send / receive. El uso de cada canal es atomico, por lo que no se haran al mismo tiempo 2 operaciones sobre el mismo canal.
		Envio: la operacion es no bloqueante, deposita el mensaje al final del canal y continua su ejecucion. 
			SEND nombreCanal(mensaje) 
			SEND nombreArreglo[i] (mensaje)
		Recepcion: La operacion es bloqueante, si el canal esta vacio se demora hasta que haya al menos un mensaje en el, luego saca el primer mensaje del canal (mas viejo)
			RECEIVE nombreCanal(Variables para el mensaje)
			RECEIVE nombreArreglo[i] (Variables para el mensaje)
	- Consultas por mensajes pendientes: retorna un booleano que indica si el canal esta o no vacio. USAR CON CUIDADO cuando el canal tien mas de UN posible receptor
			EMPTY (nombreCanal)
			EMTPY (nombreArreglo[i])
	- Uso de sentencias de alternativa multiple (IF no deterministico) y alternativas iterativa multiple (DO no deterministico): Puede generar busy waiting que en PMA esta permitido



	Ejercicios:
		1)
		En una empresa de software hay N personas que prueban un nuevo producto para encontrar errores, cuando encuentran uno generan un reporte para que un empleado corrija el error (las personas no deben reciben ninguna respuesta). El empleado toma los reportes de acuerdo al orden de llegada, los evalúan y hace las correcciones necesarias.
		Hay N procesos persona para un proceso empleado (muchos a 1)
		![[Pasted image 20261104170647.png]]
		Se usa el canal de reportes como buffer donde se guardan los reportes ordenados, las personas pueden dejar su reporte y continuar trabajando y el empleado cuando ve un reporte trabaja
		2)
		Mismo caso que el anterior pero ahora las personas esperan una respuesta del empleado cuando dejan el reporte.
		![[Pasted image 20261104171346.png]]
		Se crea un nuevo canal de respuestas para enviar las mismas, y ocurren dos problemas, la persona no tiene forma de saber si la respuesta en el canal de respuestas es para el, por lo tanto se crea un arreglo de canales y tanto la persona como el empleado hacen send y receive en dicho canal con el id enviado en reportes. El otro problema es el id a enviar en reportes para saber en que canal depositar la respuesta
		3)
		Mismo caso que el ejercicio 2 pero ahora son 3 los empleados para atender reportes
		![[Pasted image 20261104171954.png]]
		Se mantiene la solucion, ya que las personas envian los reportes a un canal y los empleados lo toman de ahi, distinto seria que las personas le envien el reporte a un empleado en partidular
		4)
		Igual que el ejercicio 1 pero ahora cuando no hay reportes que atender el empleado se dedica a leer 10 minutos
		![[Pasted image 20261104172257.png]]
		El cambio se produce en empleado, chequea antes de hacer el reaceive si hay algo en el canal, y si esta vacío se pone a leer. Para eso se usa la función empty
		5)
		Modificamos el punto 4, ahora habra 3 empleados.
		Poniendo simplemente Process Empleado[0..2] podria hacer una demora innecesaria ya que 2 empleados o mas pueden leer que el canal no esta vacio y trabarse en el receive
		![[Pasted image 20261104173240.png]]
		No hay cambios en persona, aparece un nuevo proceso coordinador junto a dos canales, Pedido que va a ser usado entre Empleados-Coordinador para avisar que el empleado esta listo para recibir un reporte y Siguiente que va a ser usado entre Coordinador-Empleados para enviar el reporte.
		Atencion en al forma de enviar el reporte. Como el coordinador entre que recibe el Pedido del empleado y lo atiende puede no llegar a tiempo y que el empleado se ponga a leer, por ende le debe responder algo automaticamente y luego le envia el reporte (usa dos veces el canal de reportes para enviar un solo reporte)
		6)
		Ahora va a haber N personas que genera un reporte y esperan a que un empleado corrija el error y el mismo le de la respuesta. El reporte puede ser urgente o normal. El empleado toma los reportes de acuerdo al orden de llagada, pero dando prioridad a los urgentes, y le responde a la persona que hizo el reporte
		![[Pasted image 20261104174717.png|628]]
		Se crean 3 canales, reportes urgentes, reportes normales y uno de aviso que simula ser un semaforo. La persona luego de enviar un reporte manda un mensaje en aviso para destrabar al empleado que estaba esperando en aviso, luego este mismo hace un if de multiples alternativas preguntando primero por el canal prioritario y luego por el otro. Por ultimo envia una respuesta