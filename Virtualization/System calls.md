### [[Limited Direct Execution#1.3.2 System calls (Syscalls)|1.3.2 System calls (Syscalls)]]
Lo normal es preguntarse ahora, ¿como es que un programa logra realizar alguna operación restringida sin peligro?
La respuesta es que no puede... Pero puede pedir que el [[Sistema Operativo]] se encargue de hacerlo ,para esto existen las _Syscalls_. Son exactamente peticiones al [[Sistema Operativo]] para que se encargue de hacer operaciones peligrosas que un [[Procesos|proceso]] podría hacer mal ,o simplemente con malicia.
De esta forma podemos exponer todos los recursos que tiene nuestra maquina de una forma mas segura. Es por eso que tenemos la capacidad de crear procesos, alojar memoria , hacer I/O ,entre otras cosas que se llevan a cabo con syscalls.

### ¿Que implica hacer una Syscall?
Para ejecutar una Syscall hay que usar una instrucción especial del procesador llamada _trap_.
Esta instrucción cambia el modo de nuestro procesador ,de [[Limited Direct Execution#User mode|User mode]] a [[Limited Direct Execution#Kernel mode|Kernel mode]] y le devuelve el control al [[Sistema Operativo]] , es entonces cuando este ultimo revisa si _"fue despertado"_ para hacer una acción  privilegiada que pidió el proceso, una vez que la haya terminado realiza la contra-instrucción que hizo el proceso, esta es _return-from-trap_, que en efecto le devuelve el control al proceso que _'trapeó'_.