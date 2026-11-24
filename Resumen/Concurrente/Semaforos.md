## Declaraciones
- sem s; -> NO. Si o si se deben inicializar en la declaracion
- sem mutex = 1;
- sem espera[5] = ([5] 1);
## Operaciones
- P(s) -> <Await (s > 0) s = s -1>
- V(s) -> <s = s +1>
## Ejercicios
### 1
- Hay C chicos y hay una bolsa con caramelos que nunca se vacia. Los chicos de a UNO van sacando de a UN caramelo y lo comen. Los chicos deben llevar la cuenta de cuantos caramelos se han tomado de la bolsa
~~~
int cant = 0;
sem mutex = 1;

Process Chico [id: 0..C-1]{
	while(true){
		P(mutex);
		// tomar caramelo
		cant++;
		V(mutex);
		// comer caramelo
	}
}
~~~
Se inicializa un semaforo que traba la ejecucion de los chicos hasta que se deje de trabajar con el recurso compartido. En este caso traba hasta que el niño anterior sume en cant. El caramelo lo pueden comer varios niños al mismo tiempo
### 2
- Hay C chicos y hay una bolsa con caramelos limitada a N caramelos. Los chicos de a UNO van sacando de a UN caramelo y lo comen. Los chicos deben llevar la cuenta de cuantos caramelos se han tomado de la bolsa
~~~
int cant = 0;
sem mutex = 1;

Process Chico [id: 0..C-1]{
	P(mutex);
	while(cant < N){
		// tomar caramelo
		cant++;
		V(mutex);
		// comer caramelo
		P(mutex)
	}
	V(mutex);
}
~~~
Idem 1, pero ahora hay una cantidad limitada de caramelos. Se comienza protegiendo la entrada al while para trabajar en seccion critica, una vez terminada dicha seccion se libera el semaforo se come el caramelo y se vuelve a pedir paso. El V(mutex) de afuera del while esta para liberar cuando cant sea igual a N
### 3
- Hay C chicos y hay una bolsa con caramelos limitada a N caramelos administrada por UNA abuela. Cuando todos los chicos han llegado llaman a la abuela, y a partir de ese momento ella N veces selecciona un chico aleatoriamente y lo deja pasar a tomar un caramelo
~~~
int contador = 0;
bool seguir = true;
sem mutex = 1;
sem espera_abuela = 0;
sem barrera = 0;
sem espera_chicos[C] = ([C] 0);
sem listo = 0;

Process Chico[id: 0..C-1]{
	int i;
	P(mutex);
	contador++;
	if(contador == C){
		for i = 1..C V(barrera);
		V(espera_abuela);
	}
	V(mutex);
	P(barrera)
	P(espera_chicos[id]);
	while(seguir){
		// tomar caramelo
		V(listo);
		// comer caramelo
		P(espera_chicos[id])
	}
}

Process Abuela{
	int i, aux;
	P(espera_abuela);
	for i = 1..N{
		aux = (random mod C)
		V(espera_chicos[aux]);
		P(listo);
	}
	seguir = false;
	for aux = 0..C-1 V(espera_chicos[aux]);
}
~~~
Chico ahora tiene una barrera que espera que todos los chicos lleguen, una vez que llegan todos el ultimo despierta a todos y a la abuela. La abuela selecciona un id al azar y lo despierta, el chico toma el caramelo y vuelve a esperar. La abuela, una vez que se quedo sin caramelos, pone seguir en false y despierta a todos los chicos
### 4
En una empresa de genetica hay N clientes que envian secuencias de ADN para que sean analizadas y esperan los resultados para poder continuar. Para resolver estos analisis la empresa cuenta con 1 servidor que resuelve todos los pedidos de acuerdo al orden de llegada de los mismos
~~~
sem mutex = 1;
sem pedidos = 0;
sem espera[N] = ([N] 0);
int resultados[N];
cola c;

Process cliente[id: 0..N-1]{
	secuencia s;
	while(true){
		// generarSecuencia(s);
		P(mutex);
		c.push(s,id);
		V(mutex);
		V(pedidos);
		P(espera[id]);
		int res = resultados[id];
	}
}

Process Servidor{
	secuencia sec;
	int aux;
	while(true){
		P(pedidos);
		P(mutex);
		c.pop(sec,aux);
		V(mutex);
		resultados[aux] = resolver(sec);
		V(espera[aux]);
	}
}
~~~
El cliente llega, genera una secuencia bloquea la cola y pushea la secuencia junto con su id, desbloquea la cola y desbloquea pedidos (pedidos le avisa al servidor que hay algo en la cola). Luego se queda esperando su resultado.
Servidor espera que le llegue un pedido, bloquea la cola, popea, desbloquea la cola y devuelve el resultado al ID que desencolo. **IMPORTANTE** resolver V(pedidos) y no haciendo un if(not c.isEmpty()) ya que si se hace con if se genera busy waiting
### 5
- En una empresa de genetica hay N clientes que envian secuencias de ADN para que sean analizadas y esperan los resultados para poder continuar. Para resolver estos analisis la empresa cuenta con 2 servidores que van alternando su uso para no exigirlos de mas (en todo momento uno esta trabajando y el otro descansando); cada 5 horas cambia el servidor con el que se trabaja. El servidor que esta trabajando, toma un pedido (de a uno de acuerdo al orden de llegada de los mismos), lo resuelve y devuelve el resultado al cliente correspondiente. Cuando terminan las 5 horas se intercambian los servidores que atienden los pedidos. Si al terminar las 5 horas el servidor se encuentra atendiendo un pedido, lo termina y luego se intercambian los servidores.
~~~
sem mutex = 1;
sem pedidos = 0;
sem espera[N] = ([N] 0);
sem inicio = 0;
sem turno[2] = (1,0);
int resultados[N];
cola c;
bool finTiempo = false;

Process cliente[id: 0..N-1]{
	secuencia s;
	while(true){
		// generarSecuencia(s);
		P(mutex);
		c.push(s,id);
		V(mutex);
		V(pedidos);
		P(espera[id]);
		int res = resultados[id];
	}
}

Process Reloj{
	while(true){
		P(inicio);
		delay(5hs);
		finTiempo = true;
		V(pedidos);
	}
}

Process Servidor{
	secuencia sec;
	int aux;
	while(true){
		P(turno[id]);
		finTiempo = false;
		V(inicio);
		ok = true;
		while(ok){
			P(pedidos);
			if(finTiempo){
				ok = false;
				V(turno[1-id]);
			}
			else{
				P(mutex);
				c.pop(sec,aux);
				V(mutex);
				resultados[aux] = resolver(sec);
				V(espera[aux]);
			}
		}
	}
}
~~~
Se crea un nuevo proceso "reloj" que lo unico que hace es esperar que lo prendan, hacer un delay de 5hs, actualizar la variable booleana finTiempo y dar el aviso en pedidos.
Servidor ahora al entrar al while espera su turno, cuando le llega pone finTiempo en false y le avisa al reloj que arranque. Realiza pedidos hasta que el reloj actualiza finTiempo, una vez que sucede eso se pone una variable interna en false y se le da paso al siguiente servidor.
### 6
- En una montaña hay 30 escaladores que en una parte de la subida deben utilizar un unico paso de a uno a la vez y de acuerdo al orden de llegada del mismo
~~~
cola c;
sem mutex = 1
sem espera[30] = ([30] 0 );
boolean libre = true;

Process Escalador[id: 0..29]{
	int aux;
	P(mutex);
	if(libre){
		libre = false;
		V(mutex);
	}
	else{
		c.push(id);
		V(mutex);
		P(espera[id]);
	}
	// usa el paso
	P(mutex);
	if(empty(C)) libre = true;
	else{
		c.pop(aux);
		V(espera[aux]);
		V(mutex);
	}
}
~~~
Llega un escalador y espera a que las variables libre y c esten liberadas (semaforo mutex), si libre es true lo pone en false, si es false pushea su id a la cola y se queda esperando en el semaforo de su id, en ambos casos libera el semaforo mutex. Una vez que le llega su turno usa el paso y traba el semaforo mutex para usar la cola y preguntar si esta vacia, de estarlo pone libre en true, de no estarlo popea y despierta al id popeado, luego destraba el semaforo mutex