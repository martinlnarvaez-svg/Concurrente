## Sintaxis
### Estructura y comunicacion

~~~
Monitor nombre{
	Variables permanentes del monitor
	
	Procedure uno()
		var locales al Procedure
		{
		...
		}
	
	Codigo de inicializacion del monitor
	{
	Inicializacion variables
	}
}

Process P[id:0..N-1]{
	...
	nombre.uno()
}
~~~

- No existen variables compartidas
- Las variables permanentes del monitor solo se pueden usar dentro del monitor
- Si el monitor tiene un código de inicialización, hasta que este no termine su ejecución el monitor no atiende llamados a los *procedures*
- Los procesos interactúan entre ellos y con los recursos compartidos por medio de los monitores haciendo llamados a los *procedures* de estos
- En un procedimiento de un monitor TAMBIEN se puede llamar a procedimientos de OTRO monitor. Pero cuidado porque el monitor desde donde se hizo el llamado se mantiene ocupado (inaccesible por otro proceso) hasta que el procedimiento llamando en el segundo monitor TERMINE POR COMPLETO su ejecución
- Cuando el monitor esta libre TODOS los procesos que están haciendo llamados a sus procedimientos compiten por acceder al monitor, NO acceden de acuerdo al orden de llegada
### Sincronizacion
~~~
Monitor nombre{
	cond vc;
	
	Procedure uno(){
		...
		wait(vc)
		...
	}
	
	Procedure dos(){
		...
		signal(vc)
		...
		signal_all(vc)
		...
	}
}
~~~
- La exclusión mutua es implícita dentro de un monitor al no poder ejecutar mas de un llamado a un procedimiento a la vez, hasta que no se termina el *procedure* o no se duerme en una variable *condition* no se libera el monitor para atender otro llamado
- La sincronizacion por Condicion es explicita por medio de variables *Conditions* usadas en los monitores. Son variables permanentes del monitor (solo se pueden usar en el monitor que fueron declaradas)
	- **Wait(vc)**: Duerme al proceso en la cola asociada a la variable condicion (al final de la la cola)
	- **Signal(vc)**: Despierta al primer proceso dormido en vc (al primero que se habia dormido) para que compita nuevamente para acceder al monitor, y cuando lo haga continuar con la instruccion despues del *wait*
	- **Signal_all(vc)**: Despierta a todos los procesos dormidos en *vc* para que todos pasen a competir por acceder nuevamente al monitor
## Ejercicios
### 1
- Existen N personas que desean utilizar un cajero automático. En este primer caso no se debe tener en cuenta el orden de llegada de las personas (cuando esta libre cualquiera lo puede usar). Suponga que hay una función. UsarCajero() que simula el uso del cajero
~~~
Monitor cajero{
	procedure pasarAlCajero(){
		UsarCajero();
	}
}

Process persona[id: 0..N-1]{
	...
	cajero.pasarAlCajero();
	...
}
~~~

Ya que no se necesita ningun tipo de sincronizacion (como por ejemplo para respetar un orden) alcanza con que el monitor represente al cajero automatico y tenga un procedure que simule su uso
### 2
- Existen N personas que desean utilizar un cajero automático. En este segundo caso se debe tener en cuenta el orden de llegada de las personas. Suponga que hay una función UsarCajero() que simula el uso del cajero
~~~
Process Persona[id: 0..N-1]{
	cajero.pasar();
	// usar cajero
	cajero.salir();
}

Monitor cajero{
	bool libre = true;
	cond cola;
	int esperando = 0;
	
	Procedure pasar(){
		if(not libre){
			esperando++;
			wait(cola);
		}
		else libre = false;
	}
	
	Procedure salir(){
		if(esperando > 0){
			esperando--;
			signal(cola);
		}
		else libre = true;
	}
}
~~~
### 3
- Partiendo de la solución anterior hacemos un cambio en el enunciado, agregando que si llega una persona anciana tiene prioridad
~~~
Process persona[id: 0..N-1]{
	bool edad = ...;
	
	cajero.pasar(id,edad);
	usarCajero();
	cajero.salir();
}

Monitor cajero{
	bool libre = true;
	cond espera[N];
	int idAux;
	int esperando = 0;
	colaOrdenada fila;
	
	Procedure pasar(idP, edad: in int){
		if(not libre){
			fila.pushOrdenado(idP, edad);
			esperando++;
			wait(espera[idP]);
		}
		else libre = false;
	}
	
	Procedure salir(){
		if(esperando > 0){
			esperando--;
			fila.pop(idAux);
			signal(espera[idAux]);
		}
		else libre = true;
	}
}
~~~
Como se pide tener un prioridad (la edad de las personas) el procedure pasar pide que se le envie la edad y el id por parametro, la edad para encolar ordenado y el id para hacer el signal. Una vez llegados al procedure salir si hay gente esperando hacemos un pop, en el que extraemos el id de la persona mas anciana y le hacemos el signal
### 4
- En un banco hay 3 empleado. Y hay N clientes que deben ser atendidos por uno de ellos (cualquiera) de acuerdo al orden de llegada. Cuando uno de los empleados lo atiende el cliente le entrega los papeles y espera el resultado
~~~
Process cliente[id: 0..N-1]{
	int idE;
	text papel,res;
	
	banco.llegada(idE);
	escritorio[idE].atencion(papel,res);
}

Process Empleado[id: 0..2]{
	text datos;
	
	while(true){
		banco.proximo(id);
		escritorio[id].esperarDatos(datos);
		res = resolver();
		escritorio[id].enviarResultado(res);
	}
}

Monitor banco{
	cola elibre;
	cond esperaC;
	int esperando = 0;
	int cantLibre = 0;
	
	procedure llegada(idE: out int){
		if(cantLibres == 0){
			esperando++;
			wait(esperaC);
		}
		else cantLibres--;
		elibres.pop(idE);
	}
	
	procedure proximo(idE: in int){
		elibres.push(idE);
		if(esperando > 0){
			esperando--;
			signal(esperaC);
		}
		else cantLibres++;
	}
}

Monitor escritorio[id: 0..2]{
	cond vcCliente;
	cond vcEmpleado;
	text datos;
	text resultados;
	boolean listo = false;
	
	procedure atencion(D: in text; R: out text){
		datos = D;
		listo = true;
		signal(vcEmpleado);
		wait(vcCliente);
		R = resultados;
		signal(vcEmpleado);
	}
	
	procedure esperarDatos(D: out text){
		if(not listo) wait(vcEmpleado);
		D = datos;
	}
	
	procedure enviarResultados(R: in text){
		resultados = R;
		signal(vcCliente);
		wait(vcEmpleado);
		listo = false;
	}
}
~~~
Cuando un cliente llega le avisa al banco que llego, este le asigna el id del empleado que lo va a atender (si no hay empleados libres lo hace esperar encolandolo para mantener el orden), por otro lado los empleados van avisando que estan libres, enviandoles sus ids al banco. Ahi es cuando se desencola el cliente y se le asinga un empleado, el cliente se comunica con el escritorio del empleado asignado y le solicita atencion, enviandole un papel y esperando un resultado. Mientras tanto el empleado le dice al escritorio suyo esperarDatos, los recibe, resuelve y envia el resultado al escritorio. **IMPORTANTE** usar la variable booleana en escritorio para que se produzca deadlock
### 5
- Se debe simular un partido de futbol 11. Cuando los 22 jugadores llegaron a la cancha juegan durante 90 minutos y luego todos se retiran
~~~
Process jugador[id: 0..21]{
	cancha.llegada();
}

Process partido{
	cancha.iniciar();
	delay(90 minutos);
	cancha.terminar();
}

Monitor cancha{
	int cant = 0;
	cond espera;
	cond inicio;
	
	procedure llegada(){
		cant ++;
		if(cant == 22) signal(inicio);
		wait(espera);
	}
	
	procedure iniciar(){
		id(cant < 22) wait(inicio);
	}
	
	procedure terminar(){
		signal_all(espera);
	}
}
~~~
Cuando llega un jugador avisa que llego, el ultimo que llegue va a despertar el partido (si es que el partido llego antes que los 22 jugadores), dicho partido se va a jugar 90 minutos y va avisar que termino, haciendo un signal_all a todos los jugadores dormidos.
### 6
En una empresa de genética hay N clientes que envían secuencias de ADN para que sean analizadas y esperan los resultados para poder continuar. Para resolver estos análisis la empresa cuenta con un servidor que resuelve los pedidos de acuerdo al orden de llegada de los mismos.
~~~
Process cliente[id: 0..N-1]{
	text S;
	text res;
	
	while(true){
		S = generarSecuencia();
		admin.pedido(id,S,res);
	}
}

Process Servidor{
	text sec;
	text res;
	int aux;
	
	while(true){
		admin.sig(aux,sec);
		res = analizarSec(sec);
		admin.resultado(aux,res);
	}
}

Monitor admin{
	cola c;
	cond espera;
	cond haypedido;
	text res[N];
	
	procedure pedido(idC: in int; S: in text; R: out text){
		c.push(idC,s);
		signal(haypedido);
		wait(espera);
		R = res[idC];
	}
	
	procedure sig(idC: out int; S: out text){
		if(c.isEmpty()) wait(haypedido);
		c.pop(idC,S);
	}
	
	procedure Resultado(idC: in int; R: in text){
		res[idC] = R;
		signal(espera);
	}
}
~~~
### 7
- Modificamos el enunciado anterior para que haya 2 servidores en lugar de 1. Y partimos de la solucion anterior.
~~~
Process cliente[id: 0..N-1]{
	text S;
	text res;
	
	while(true){
		S = generarSecuencia();
		admin.pedido(id,S,res);
	}
}

Process Servidor[id: 0..1]{
	text sec;
	text res;
	int aux;
	
	while(true){
		admin.sig(aux,sec);
		res = analizarSec(sec);
		admin.resultado(aux,res);
	}
}

Monitor admin{
	cola c;
	cond espera[N];
	cond haypedido;
	text res[N];
	
	procedure pedido(idC: in int; S: in text; R: out text){
		c.push(idC,s);
		signal(haypedido);
		wait(espera[idC]);
		R = res[idC];
	}
	
	procedure sig(idC: out int; S: out text){
		while(c.isEmpty()) wait(haypedido);
		c.pop(idC,S);
	}
	
	procedure Resultado(idC: in int; R: in text){
		res[idC] = R;
		signal(espera[idC]);
	}
}
~~~
Lo que cambia es el aviso de que hay resultado dentro del monitor, ahora tenemos tantas conditions espera como clientes haya, por lo tanto el signal y el wait van a ir dirigidos a [idC]. Además también se cambia el chequeo el procedure sig, ahora no se chequea con un if c.isEmpty, sino que se chequea con un while, ya que el monitor anterior pudo haber dejado la cola vacía y que el siguiente lea algo que esta vacio