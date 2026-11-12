Los semáforos tienen el problema de que al haber variables compartidas globales a los procesos, sentencias de control de acceso a la SC dispersas en el codigo, al agregar procesos, se debe verificar accesos correcto a las variables compartidas. Para solucionar esto existen los monitores.

Los monitores son módulos de programa con mas estructura, y que pueden ser implementados tan eficientemente como los semáforos. Son un mecanismo de abstracción de datos
- Encapsulan las representaciones de recursos
- Brindan un conjunto de operaciones que son los únicos medios para manipular esos recursos
Contienen variables que almacenan el estado del recurso y procedimientos que implementan las operaciones sobre ellos

La exclusión mutua es implícita. Un monitor va a estar siendo utilizado en un momento solo por un proceso. Los demás se encolan y cuando el proceso termine de ser usado, se va a elegir uno encolado de forma no deterministica.

Los monitores tienen interfaz y cuerpo. Solo los nombres de los procedures (la interfaz) son visibles desde afuera. Los procedures pueden acceder solo a variables permanentes, sus varaibles locales, y parametros que le sean pasados en la invocacion.
La comunicacion entre procesos sera haciendo uso del monitor que va a contener los datos compartidos, que son variables o estructuras de datos accesibles por varios procesos. Dentro del monitor van a haber variables condicionales que permitiran a los procesos esperar o notificar eventos especificos
![[Pasted image 20260930002319.png]]

- Sincronizacion por condicion
Es con variables condicion. Cond cv;
cv es una cola de procesos demorados, no visible directamente al programador
wait(cv) -> el proceso se demora al final de la cola de cv y deja el acceso exclusivo al monitor
signal(cv) -> despierta al proceso que esta al frente de la cola y lo saca de ella. El proceso despertado recien podra ejecutar cuando readquiera el acceso exclusivo al monitor
signal_all(cv) -> despierta todos lo procesos demorados en cv, quedando vacia la cola asociada a cv

Disciplinas:
Signal and continued:
	El proceso que hace el signal continua usando el monitor, y el proceso despertado pasa a competir por acceder nuevamente al monitor para continuar con su ejecucion (en la instruccion posterior al wait)

Signal and wait:
	El proceso que hace el signal pasa a competir por acceder nuevamente al monitor, mientras que el proceso despertado pasa a ejecutar dentro del monitor a partir de la instruccion posterior al wait
![[Pasted image 20260930002950.png]]


Passing the condition:
	Transferencia de la condición que se esta esperando de un proceso a otro.
	La idea es permitir que un proceso notifique a otro sobre un cambio en una condición especifican lo que generalmente se realiza mediante el uso de variables condicionales.
	Si hay un proceso dormido esperando por determinada condición, el que lo estaba bloqueando lo despierta (le pasa la condición para seguir). Si no hay ningún proceso bloqueado, la condición se hará verdadera para que el próximo proceso que venga pueda usarla


![[Pasted image 20260930003435.png]]
El monitor arbitra el acceso a la BD. Los procesos dicen cuándo quieren acceder y cuándo terminaron

![[Pasted image 20260930003528.png]]
Se usa cola ordenada y variables condiciones privadas