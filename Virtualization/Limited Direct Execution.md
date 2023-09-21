#Process #ProcessAPI #incomplete #Mechanism 
# Intro
Para lograr la [[Virtualization]] nuestro [[Sistema Operativo]] necesita lograr que se _comparta_ de forma eficiente entre los procesos nuestro recurso a virtualizar, la CPU. El [[Sistema Operativo]] tiene esta capacidad tan bien conseguida que hay _sensación de simultaneidad_.

A la hora de pensar como lo haría uno mismo hay varios problemas que mirar bien antes de presentar la implementación como una buena.
Primero que nada queremos asegurar el _mejor rendimiento_ ,que no se este _desperdiciando uso_ de nuestro recurso. A si mismo queremos la _maxima velocidad de calculo_.
Por otra parte ,aunque parezca mas trivial, hay que asegurarse de que todos _los programas terminen_ o al menos, que no _monopolicen_ nuestro recurso. O PEOR ,que el [[Sistema Operativo]] ,que también existe como proceso ,no pierda el control de los recursos.

Todo esto cobra mas sentido a medida que vayamos desarrollando el capitulo y proponiendo como llevar la tarea.


# 1. Direct Execution
Para correr programas de la forma mas veloz que se pueda propongamos este _mecanismo_ ,que en pocas palabras,  pone al programa directamente a correr sobre la CPU ,y le da al proceso la idea de que es dueño de todo el recurso.
## 1.1 ¿Como se corre un programa bajo este mecanismo?
Cuando el usuario pide que se corra un programa, nuestro [[Sistema Operativo]] se encarga de alojarle algo de memoria, lo pone en la _lista de procesos_ ,pone las instrucciones del programa en la memoria ,localiza la _primera instrucción_ y hace un salto hasta esta. Una vez el programa termino con la ultima instrucción ,el [[Sistema Operativo]] vuelve a tomar control y prepara su siguiente acción.

--- 
## 1.2 Problemas de la Direct Execution

### 1.2.3 Monopolización de CPU
Hay una sutileza en los pasos que describimos antes ,*"...Una vez el programa termino..."* ,pero no podemos asegurar de ninguna manera que nuestro programa es por ejemplo uno de la forma:
```c
int main(void)
{
	while(1)
	{
		printf("Estoy monopolizando el cpu")
	}

	return 0;
}
```
Este programa nunca podría finalizar por si mismo,y el [[Sistema Operativo]] nunca podrá recuperar el control. Mas aún tener cosas como bucles infinitos podría no ser tan evidente como en este caso ,por lo que tratar de predecir bucles infinitos es una mala idea ,por encima de eso ,la teoría de la computación nos dice que no existe un programa que pueda predecir lo que hace otro programa ,pero eso es de otra materia.
### 1.2.3 Mantener el control 
Al darle a un procesos control absoluto del recurso ,nuestro [[Sistema Operativo]] lo pierde y ya no tiene forma de por ejemplo detener un programa y poner a ejecutar otro. Esto en consecuencia quita la posibilidad de tener el *sharing* que es vital para que se comporte como describimos en la introducción.

### 1.2.4 Garantizar la integridad de otros programas 
Uno podría tener la inocencia que dándole todo el poder de nuestro hardware a un programa ,este no va a ser maliciosos. Por ejemplo que pasa si un programa decide que va a escribir todo el disco con basura ,dejando inútil esa instalación del [[Sistema Operativo]].

---

## 1.3 Limitando la ejecución directa
En esta parte arreglamos los "problemas de jerarquía" que surgen entre el OS y el proceso.

Es evidente que hay que ponerle _limites_ a lo que un procesos puede hacer y lo que no. Poniendo estos _limites_ tenemos como objetivo mantener esas ventajas de correr un programa directamente en el procesador ,pero asegurándonos de que nunca nuestro [[Sistema Operativo]] pierda el control.
### 1.3.1 Restringir las operaciones
Para empezar a mejorar nuestro mecanismo de ejecución de procesos ,vamos a introducir los modos del procesador.
Un procesador como hardware esta preparado para colaborar con el [[Sistema Operativo]] brindándole dos modos:

#### User mode:
Un programa que corre en un procesador bajo este modo ,tiene limitado el set de instrucciones que puede usar ,por ejemplo un programa que esta en este modo no puede pedir lecturas y escrituras de disco ,entre otras operaciones peligrosas. Si el programa ,intentara hacerlo de igual manera ,nuestro [[Sistema Operativo|OS]] simplemente le tira _[[Process API#5. Mas allá de lo básico|kill()]]_. Los procesos comunes corren todo el tiempo en este modo.
#### Kernel mode:
Esta es la contra parte de el user mode ,cuando el procesador esta en este modo puede manejar instrucciones de mayor privilegio ,y que requieren mas responsabilidad para asegurar la integridad del [[Sistema Operativo|OS]]. El [[Sistema Operativo]] corre todo el tiempo en este modo.

Mas adelante introducimos la noción de [[Limited Direct Execution#Context switch|Context switch]] que es esencial para terminar de definir estos modos.

### 1.3.2 System calls (Syscalls)
Lo normal es preguntarse ahora, ¿como es que un programa logra realizar alguna operación restringida sin peligro?
La respuesta es que no puede... Pero puede pedir que el [[Sistema Operativo]] se encargue de hacerlo ,para esto existen las _Syscalls_. Son exactamente peticiones al [[Sistema Operativo]] para que se encargue de hacer operaciones peligrosas que un [[Procesos|proceso]] podría hacer mal ,o simplemente con malicia.
De esta forma podemos exponer todos los recursos que tiene nuestra maquina de una forma mas segura. Es por eso que tenemos la capacidad de crear procesos, alojar memoria , hacer I/O ,entre otras cosas que se llevan a cabo con syscalls.

### ¿Que implica hacer una Syscall?
Para ejecutar una Syscall hay que usar una instrucción especial del procesador llamada _trap_.
Esta instrucción cambia el modo de nuestro procesador ,de [[Limited Direct Execution#User mode|User mode]] a [[Limited Direct Execution#Kernel mode|Kernel mode]] y le devuelve el control al [[Sistema Operativo]] , es entonces cuando este ultimo revisa si _"fue despertado"_ para hacer una acción  privilegiada que pidió el proceso, una vez que la haya terminado realiza la contra-instrucción que hizo el proceso, esta es _return-from-trap_, que en efecto le devuelve el control al proceso que _'trapeó'_.

### Trap and save the context
Cuando se ejecuta la instrucción _trap_, el hardware se asegura de _guardar el contexto_ que tenia el programa que hizo una [[Limited Direct Execution#1.3.2 System calls (Syscalls)|System call]], con esto principalmente nos referimos a guardar en algún lado los valores que tenían los registros antes de trapear. En x86 se guardan las cosas en espacios de memoria conocida respectivamente por ambos modos.
### Trap table
Un problema un poco menos obvio ,es tener programas haciendo saltos a donde hay instrucciones del kernel. Cuando el [[Sistema Operativo]] **bootea** le dice al hardware que hay bloques de instrucciones que se corren solamente por el kernel, y que se usan en situaciones especificas. También al _bootear_ crea una _tabla_ que tiene las instrucciones de como manejar estas situaciones especificas (_trap handlers_).
Cuando el programa realiza una _trap_ también informa cual fue, indicando una de la _trap table_ y el kernel entonces puede saber que _handler_ usara.

### Sumario del concepto de trap

#### Ejemplo con syscalls y traps
- Un programa decide hacer una _Syscall_.

- Se lleva a cabo la instrucción _trap_.

- El [[Sistema Operativo|OS]] retoma el control sabiendo que fue por el uso de _trap_.

- Identifica en la _trap table_ el _trap handler_ correspondiente y salta a ese bloque de instrucciones.

- Una vez las haya finalizado hace _return-from-trap_.

- El proceso retoma el control y continua su ejecución.

Mas detalladamente veamos el cuadro siguiente como una linea del tiempo de arriba hacia abajo.

| Tarea de hardware                                         | OS(Kernel mode)                             | Programa(User mode)   |
| --------------------------------------------------------- | ------------------------------------------- | --------------------- |
|                                                           | Agregar el programa a la lista de procesos  |                       |
|                                                           | Alojar memoria para el programa             |                       |
|                                                           | Cargar el programa en memoria               |                       |
|                                                           | Prepararle el stack con los argv[]          |                       |
| Guardar los registros del kernel en algún lugar y el PC   |                                             |                       |
|                                                           | return-from-trap                            |                       |
| Pasar a user mode                                         |                                             |                       |
| Saltar a a la main del programa                           |                                             | Correr main()         |
|                                                           |                                             | ...                   |
|                                                           |                                             | Hacer una system call |
|                                                           |                                             | trap                  |
| Guardar los registros del proceso                         |                                             |                       |
| Pasar a kernel mode                                       |                                             |                       |
| Saltar al trap handler correspondiente                    |                                             |                       |
|                                                           | Handle trap                                 |                       |
|                                                           | Ejecutar la instrucciones de la system call |                       |
|                                                           | return-from-trap                            |                       |
| Restaurar los registros del user y guardar los del kernel |                                             |                       |
| Pasar a user mode                                         |                                             |                       |
| Hacer salto al PC que guardamos                           |                                             |                       |
|                                                           |                                             | Continuar la ejecución                       |

---
## 1.4 Cambiar de proceso (y como recuperar el control)
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

### Nota sobre concurrencia
Si bien es tema de otros mecanismos, uno se pregunta cosas como ¿Puede ocurrir una _Interrupción mientras se esta en 'handleando' una syscall_? ¿Que pasa si esto ocurre?

La respuesta corta es que esencialmente se desactivan las interrupciones, mientras el [[Sistema Operativo]] tiene el control, cosa que tiene sentido, el mecanismo esta ahí justamente para asegurar eso.
Cuando veamos herramientas de concurrencia como _spinlocks, semáforos,etc_ vamos a ver que si bien es delicado, desactivar las interrupciones es lo que tiene sentido.

---
## 1.5 Guardar y restaurar el contexto
Ahora que nuestro _OS_ es capaz de retomar el control bajo cualquier circunstancia quisiéramos volver a enfocarnos en la idea de _compartir los recursos_, osea el comportamiento de tomar turnos entre procesos para utilizar el CPU, la decision de ver a cual proceso le toca su turno la discutimos en otro momento cuando veamos al [[Process scheduling|Scheduler]] en detalle.

Por ahora caractericemos la capacidad de _para un procesos y continuarlo después_.
### Context switch
En el momento en que se decide cambiar de un proceso a otro, se ejecuta una pieza de _código de bajo nivel_ a la que nos referimos como _context switch_. De lo que se encarga es de _guardar_ el contenido de algunos registros y otra información del estado del proceso _que esta en ejecución_ y _sustituirlo_ con el de el proceso al que _vamos a cambiar_.

*Mas en detalle*, en primer lugar el OS ,decide que va a cambiar de proceso luego de alguna *interrupción*, hace _trap_ y entra en _modo kernel_, procede a _guardar contexto_; entonces ejecutamos código assembly, para guardar el contenido de los registros de propósitos generales, también el *PC(Program Counter)* , que corresponden al proceso y una vez guardados en el _kernel stack_ lo reemplaza por sus relativos del _proceso al que vamos a cambiar_. Luego de esto hay un _return-from-trap_ que resulta en tener al otro proceso ahora ejecutando en el CPU.

### Switch()
Esta es precisamente la rutina implementada en XV6 (y en UNIX en general) que se encarga de cuidadosamente guardar los registros en el Kernel stack( Kstack ), así como el PC.
Cuando el [[Sistema Operativo]] decida que va a cambiar de proceso va a llamar a *switch()*.

[[|]]

### Cambio de contexto implícito
Hay que notar que realmente en esta sección revisamos dos tipos de cambio de contexto.

El primero es el que explicamos que tiene que ver con pasar de ejecutar un proceso A a ejecutar un Proceso B.

El otro lo aprendimos implícitamente, este es el que sucede al devolverle el control al OS, recordemos que este también existe como proceso y tiene sus registros , y PC independiente de los demás procesos ,por lo que ahí también hay un cambio de contexto.
### Para concluir
A continuación un ejemplo bien concreto en forma de linea del tiempo

| Hardware                                    | Os(Kernel mode)                                                                  | Process A | Process B |
| ------------------------------------------- | -------------------------------------------------------------------------------- | --------- | --------- |
|                                             |                                                                                  | ...       |           |
|                                             |                                                                                  | Running   | Ready     |
| Timer Interrupt                             |                                                                                  |           |           |
| Guardar el contexto de A                    |                                                                                  |           |           |
| Pasar a kernel mode                         |                                                                                  |           |           |
| Darle el control al OS                      |                                                                                  |           |           |
|                                             | Handle trape                                                                     |           |           |
|                                             | Decide cambiar a B                                                               |           |           |
|                                             | Llama a _switch()_                                                               |           |           |
|                                             | Guardar los registro de A, así como el PC en la proc table                       |           |           |
|                                             | Restaurar los registros de B y su PC desde la proc table y ponerlos en el Kstack |           |           |
|                                             | return-from-trap (poner a B a ejecutar )                                         |           |           |
| Restaurar el estado de B leyendo del Kstack |                                                                                  |           |           |
| Pasar a user mode                           |                                                                                  |           |           |
| Saltar al PC de B                           |                                                                                  |           |           |
|                                             |                                                                                  | Ready     | Running   |

# [[CPU Virtualization|FIN]]

## Código de switch() en XV6.
```c
# Context switch
#
#   void swtch(struct context **old, struct context *new);
# 
# Save the current registers on the stack, creating
# a struct context, and save its address in *old.
# Switch stacks to new and pop previously-saved registers.

.globl swtch
swtch:
  movl 4(%esp), %eax
  movl 8(%esp), %edx

  # Save old callee-saved registers
  pushl %ebp
  pushl %ebx
  pushl %esi
  pushl %edi

  # Switch stacks
  movl %esp, (%eax)
  movl %edx, %esp

  # Load new callee-saved registers
  popl %edi
  popl %esi
  popl %ebx
  popl %ebp
  ret
```