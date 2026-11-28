s- Los programas se componen solo de procesos (NO EXISTEN VARIABLES COMPARTIDAS)
- los canales son de tipo LINK (unico emisor y un receptor) y sincronicos. Son estructuras implicitas y no se deben declarar
- Los procesos interactuan entre ellos UNICAMENTE por medio del envio de mensajes
- No se requiere sincronización por exclusión mutua ya que no existen las variables compartidas

Sintaxis:
- Envio (!): La operacion es bloqueante y sincronica, se demora hasta que la recepcion haya terminado.
	 destino!port (mensaje)
	 destino[i]!port (mensaje)
- Recepcion(?): La operacion es bloqueante y sincronica
     origen?port (variable)
     origen[i]?port (variable)
     origen[*]?port (variable)
- Uso de comunicacion Guardada (If y Do): Permite seleccionar de forma no deterministica entre varias alternativas de comunicacion (en caso de la practica SOLO RECEPCIONES) en base a las condiciones el proceso y los mensajes que estan listos para ser recibidos
- Guardas: cada alernativa es una guarda con la forma: B; C -> S
	- B: condicion booleana que puede no estar (en ese caso se considera true), e indica si el proceso esta o no en condiciones de procesar el mensaje recibido en C
	- C: sentencia de comunicacicon (en la practica solo RECEPCION) que seguro debe estar
	- S: conjunto de sentencias que se ejecutaran en caso de ser elegida la guarda
- Evaluación de una guarda
	- Éxito: la condición booleana es Verdadera y la comunicación se puede realizar sin producir demora
	- Fallo: La condición booleana es Falsa, sin importar lo que ocurra con la sentencia de comunicación
	- Bloqueo: La condición booleana es Verdadera (o no la tiene) pero la comunicación NO se puede realizar sin producir demora (el emisor aun no llego a la sentencia de envió)
- En el if "guardado" se evaluan todas las guardas y en base a eso:
	- Si una o mas son exitosas se selecciona una de ellas en forma NO DETERMINISTICA, se ejecuta la sentencia de recepcion (C) que forma parte de la guarda, y posteriormente el conjunto de sentencias asociadas a la guerda (S)
	- Si todas guardas FALLAN no se selecciona ninguna y se sale del IF sin realizar ninguna accion
	- Si no hay ninguna guarda EXITOSA pero hay una o mas con estado BLOQUEO entonces el proceso se demora en el IF hasta que haya una guarda exitosa. En ese momento se ejecuta igual que el primer caso
- El do "guardado" funciona de la misma manera que el IF, con la diferencia que en lugar de hacerlo una vez repite el mecanismo hasta que todas las guardas FALLAN

EJERCICIOS
1)
En una empresa de software hay un empleado Testeo que prueba un nuevo producto para encontrar errores, cuando encuentra uno generan un reporte para que otro empleado Mantenimiento corrija el error y le responda. El empleado Mantenimiento toma los reportes para evaluarlos, hacer las correcciones necesarias y responderle al empleado Testeo.
Sabemos que solo hay dos procesos comunicándose en el que se esperan sus respuestas mutuamente, por lo tanto no se debe modelar un buffer de respuestas
![[Pasted image 20261105111434.png]]
2)
Mismo ejemplo que el 1) pero ahora el empleado Testeo no debe esperar una respuesta para seguir trabajando.
![[Pasted image 20261105112005.png]]
Se crea un nuevo proceso intermediario entre Testeo y Mantenimiento que se llama Admin, su unica funcion es recibir los reportes, almacenarlos y darselos a Mantenimiento cuando este le avise que quiere un reporte. Atencion en not empty(Buffer), sin esto cuando no haya pedidos y Mantenimiento pida un reporte se rompe el programa
3)
Ahora hay N empleados Testeo que generan reportes y esperan que Mantenimiento les de una respuesta
![[Pasted image 20261105112858.png]]
Testeo genera un reporte y le pasa este mismo y su id a Admin, este espera con * el reporte y lo encola en un buffer, Mantenimiento le pide un pedido y este se lo envia. Luego Mantemiento le envia la respuesta a Testeo con el id previamente desencolado
4)
Ahora hay N emplaedos Testeo que genera reportes para que uno de los 3 emplaedos Mantenimiento corrija el error y les responda
![[Pasted image 20261105113559.png]]En las comunicaciones entre Mantenimiento y Admin se agrega que se pase el id de Mantenimiento para que Admin sepa a quien le debe responder. Y Testeo espera la respuesta de cualquier Manenimiento (Asi como tambien lo hace Admin cuando le llega un aviso de cualquier Mantenimiento)