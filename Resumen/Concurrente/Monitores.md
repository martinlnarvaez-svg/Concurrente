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
- La exclusion mutua es imlicita dentro de un monitor al no poder ejecutar mas de un llamado a un procedimiento a la vez, hasta que no se termina el *procedure* o no se duerme en una variable *condition* no se libera el monitor para atender otro llamado
- La sincronizacion por Condicion es explicita por medio de variables *Conditions* usadas en los monitores. Son variables permanentes del monitor (solo se pueden usar en el monitor que fueron declaradas)
	- **Wait(vc)**: Duerme al proceso en la cola asociada a la variable condicion (al final de la la cola)
	- **Signal(vc)**: Despierta al primer proceso dormido en vc (al primero que se habia dormido) para que compita nuevamente para acceder al monitor, y cuando lo haga continuar con la instruccion despues del *wait*
	- **Signal_all(vc)**: Despierta a todos los procesos dormidos en *vc* para que todos pasen a competir por acceder nuevamente al monitor
## Ejercicios
### 1
- Existen N personas que desean utilizar un cajero automático. En este primer caso no se debe tener en cuenta el orden de llegada de las personas (cuando esta libre cualquiera lo puede usar). Suponga que hay una función. UsarCajero() que simula el uso del cajero