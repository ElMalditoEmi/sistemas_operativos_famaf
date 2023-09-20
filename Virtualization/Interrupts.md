## [[Limited Direct Execution#1.4 Cambiar de proceso (y como recuperar el control)|Interrupts]]
Ya resolvimos uno de los problemas de la ejecución directa ,y ahora vamos a centrarnos en otro.
Cuando decimos que el programa esta corriendo directamente sobre la CPU, es literal ,por lo que _el OS no esta corriendo_ mientras esto sucede, ¿como va a ser capaz entonces de hacer cualquier cosa?

Vamos a destacar los _dos enfoques_ con los que podemos lograr esto. En un [[Sistema Operativo]] lo mas seguro es tener ambos.
#### Enfoque cooperativo
Con esta mirada vamos a confiar que los procesos se comportan de forma razonable. Los procesos largos ,periódicamente tienen instrucciones de liberar la CPU para que el [[Sistema Operativo]] pueda decidir correr otras tareas.
Esto realmente es inevitable y de hecho frecuente porque tenemos syscalls y siempre que quieran hacer operaciones privilegiadas le pasan el control al OS para que las haga por ellos en _kernel mode_. Ademas en [[UNIX]] tenemos la syscall _yield()_ que tiene como único propósito devolver el control al OS por voluntad propia.
Otra situación donde le devolvemos el control al OS ,es al intentar ejecutar instrucciones _ilegales_ (E.G: dividir por 0;Escribir memoria ajena),esto hace que el procesos _trapee_ para que el [[Sistema Operativo]] se encargue de tomar cartas en el asunto.

En un [[Sistema Operativo]] que cambia de proceso con enfoque puramente cooperativo tiene solo estas opciones para retomar el control, o sea que un desafortunado bucle infinito va a obligarnos a reiniciar la computadora.
#### Enfoque no cooperativo 
El hardware como siempre es el mayor aliado del kernel, en esta ocasión vamos a ver como ayuda a recuperar el control por mas de que no suceda ninguna situación que vimos del enfoque cooperativo.
##### Timer interrupt
Nuestro hardware provee un cronometro, que cada un cierto tiempo, cambia a Kernel mode y devuelve el control de forma _automática e inevitable_. A partir de ahora vamos a denominar *interrupt* o *interrupción* a esta situaciones donde por la fuerza retomamos el control de la maquina.

Con esto ya no se puede monopolizar el CPU, ya que inevitablemente el [[Sistema Operativo]] retoma el control y puede decidir correr otros procesos distintos libremente.

Cabe mencionar que al hacer una _interrupción_ el hardware también es responsable de hacer _trap_ para _guardar el contexto del proceso_ y volver a ese punto de su ejecución si así lo decide el [[Sistema Operativo]].

### Sobre los enfoques.
En un [[Sistema Operativo]] consistente consideramos ambas visiones, si a veces los procesos son tan razonables como para terminar o hacer _yield()_ antes de un _timer interrupt_ estamos cubiertos.
Y como es ingenuo pensar que todos los procesos se comportan así, también estamos cubiertos.