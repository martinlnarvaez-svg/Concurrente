
Semaforos:
Tienen las operaciones atomicas P y V. V señala la ocurrencia de un evento (Incrementa) y P se usa para demorar un proceso hasta que ocurra un evento (Decrementa)

Sintaxis:
	sem s; --> NO SE PUEDE
	sem mumtex = 1;
	sem fork[5] =  ([5]1); --> arreglo de 5 semaforos, los 5 inicializados con valor 1
	NO SE PUEDE INCIALIZAR UN SEMAFORO CON VALOR < 0

Semaforo general
	P(s): <await (s>0) s=s-1;>
	V(s): <s=s+1;>

Problemas basicos y tecnicas

- Seccion critica: Exclusion mutua

![[Pasted image 20260929210633.png|409]]![[Pasted image 20260929210641.png|301]]

La imagen de arriba es equivalente a la de abajo. Ademas es mas simple que las soluciones busy waiting

- Barreras
 ![[Pasted image 20260929211326.png|274]]
 La idea es un semaforo para cada flag de sinc. Un proceso setea el flag ejecutando V, y espera a que un flag sea seteado y luego lo limpia ejecutando P

- Productor Consumidor
![[Pasted image 20260929211746.png]]

Ahora con buffer limitado (El buffer actúa como una cola circular y esta representado por un vector)
![[Pasted image 20260929212419.png]]Vacío cuanta los lugares libres, y lleno los ocupados
Depositar y retirar se pudieron asumir atómicas pues solo hay un productor y consumidor

Si hay mas de un productor y/o consumidor esto no funciona y hay que hacer lo siguiente:
Las operaciones depositar y retirar son SC y deben ejecutar con exclusión mutua. De no hacerse se podría retirar dos veces el mismo dato o perderse datos al sobrescribirlo
![[Pasted image 20260929213020.png]]

- Varios procesos compitiendo por varios recursos compartidos

![[Pasted image 20260929213355.png]]
Cada tenedor es una SC: puede ser tomado por un único filosofo a la vez -> pueden representarse los tenedores por un arreglo de semáforos. Levantar un tenedor -> P, bajar un tenedor -> V
Cada filosofo necesita el tenedor izquierdo y el derecho.
Si todos los filósofos quieren tomar, por ejemplo, su tenedor derecho se podría generar deadlock 
![[Pasted image 20260929213806.png|352]]
Esa solución asegura que no se genera deadlock ya que al haber un filosofo tome los tenedores en orden inverso compite con otro filosofo que los esta haciendo en orden inverso, esto hace que uno de los dos o gane sus tenedores o tenga que esperar a que el otro termine para tomar sus tenedores

- Problema de lectores escritores

Con exclusión mutua:
![[Pasted image 20260929215212.png]]
Esta solución no es fair, ya que tiene preferencia a los lectores. Otro enfoque es la técnica passing the baton
Grano grueso:
![[Pasted image 20260929215619.png]]
Grano fino:
![[Pasted image 20260929232539.png]]
abreviado: 
![[Pasted image 20260929233238.png]]

Semáforos privados:
	S es un semáforo privado si exactamente un proceso ejecuta operaciones P sobre S. Son útiles para señalar procesos individuales. Los semáforos b[id] son de este tipo. Ej.:
	Hay C chicos y hay una bolsa con caramelos limitada a N caramelos administrada por UNA abuela. Cuando todos los chicos han llegado llaman a la abuela, y a partir de ese momento ella N veces selecciona un chico aleatoriamente y lo deja pasar a tomar un caramelo.![[Pasted image 20260929235019.png|653]]