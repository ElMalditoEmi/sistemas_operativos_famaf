#complete #Process #ProcessAPI #XV6

# 1.1 Primero que nada:¿Que es un proceso?
Un proceso no es sino una abstracción que el [[Sistema Operativo|OS]] provee para si mismo ,en si un programa no es nada mas que una serie de lineas inentendibles que están en código maquina guardadas en algún lugar del disco. El sistema operativo es el que "le da vida" a estas instrucciones y se encarga de ejecutarlas ,crear un proceso es la acción del sistema operativo ejecutando un programa.

# 1.2 Process API
#### *Si bien no estamos listos para discutir sobre API's acá se introduce un poco la idea.*

Prácticamente todos (si es que no, todos) los [[Sistema Operativo|OS]] deberían ser capaces de incluir al menos las siguientes acciones para procesos (Vamos a ver después que esto es una API):
- **Crear**:Crear un proceso es algo que debe estar en una API de procesos (si no que sentido tendría), al llamar un programa en un [[Shell]] ,o hacerle click al icono, el OS se encarga de crear el programa que en efecto clickaste. ^05f158
- **Destruir**:Así como hay interfaz de [[CPU Virtualization#^05f158|creación]] ,de forma análoga vamos a querer que,una vez nuestros programas hayan finalizado con éxito (o no), no queden vestigios de nuestros programas estorbando a los demás ,mas adelante se discute de [[Zombie processes]].

- **Esperar**:Es muy util poder espera a que un proceso finalice.

- **Otros controles**:También esta bueno tener procesos que se **suspenden** en vez de simplemente morir y perder todo el progreso. Suspender un procesos lo hace dejar de correr y tener capacidad de continuar luego desde ese punto.

- **Mostrar el estado**:Saber sobre los procesos en muy util ,como vamos a ver mas abajo en [[Process scheduling]], para acomodar un proceso en un schedule de forma eficiente tenemos que saber cuanto tiempo lleva corriendo ,incluso algunas implementaciones querrán saber con cuanto esfuerzo se corre un programa.

En el [[Process API|próximo capitulo]] se discuten las implementaciones reales de toda esta API.

# 1.3 ¿Como se levanta un proceso?(o mas o menos)
### 1.3.1 Cargar el código
Lo primero que hace un OS cuando se le encarga la tarea de crear un proceso es cargar el código que esta en alguna parte del disco, a la memoria para que se pueda encargar ejecutarlo el hardware responsable. Los sistemas operativos modernos no suelen cargar todo el código de una sola vez en la memoria ,si no que "lazily"(van cargando las instrucciones a medida que avanza la ejecución).
### 1.3.2 Preparar memoria
Una vez ya se cargaron las instrucciones en RAM , nuestro programa necesita mas memoria para poder guardar: argumentos de funciones, estructuras de datos ,variables ,etc.

Para las funciones el OS provee lo que se llama **run-time stack** para que pueda haber: variables locales ,puntos de retorno, parámetros ,etc. Por ejemplo también para un programa en C, el OS también inicializa los valores del stack con **argc** y **argv**.

También al programa hay que darle un **heap** para que pueda almacenar sus estructuras de datos, así como usar puntero u otros manejos de memoria necesarios, lo que en C viene a ser malloc() y su familia de funciones para trabajar en memoria dinámica.

### 1.3.3 Preparar I/O
Un programa por lo general necesita comunicarle al usuario lo que esta haciendo o hizo, el OS para esto provee STDOUT , STDERR ,STDIN que son dos canales de salida y uno de entrada respectivamente para este fin. A su vez si el usuario va a usar archivos también se prepara todo acá, pero esto se ve en la parte de [[Persistence]].


# 1.4 Process states
Ahora que tenemos una imagen mas clara de lo que es un proceso ,podemos hablar de que estado puede tener este. Para verlo de forma simple un proceso puede estar:

- ***Corriendo***: Significa que el OS lo levanto y esta ejecutando instrucciones en el procesador en el momento.
- ***Listo***:El proceso esta preparado para correr pero el OS decidió que todavía no puede entrar a la cancha.
- ***Bloqueado***: Un proceso se bloquea al hacer algo que no le permite estar listo para estar ejecutando, un ejemplo puede ser haberse ido a leer el disco(perform I/O).

# 1.5 Acomodar procesos según su estado
_*NOTA:Esto se ve mejor explicado cuando hablamos de scheduling mas adelante.*_

Como podemos ver en el gráfico(pensado como diagrama de estados) nuestros proceso se acodan en un **Schedule** para ejecutarse en orden, si tenemos un CPU con un solo núcleo ,solo vamos a poder ir de a un proceso como en la figura 4.3 del libro.


| Time | $Process_0$ | $Process_1$ | Notes                |
| ---- | ----------- | ----------- | -------------------- |
| 1    | Running     | Ready       |                      |
| 2    | Running     | Ready       |                      |
| 3    | Running     | Ready       |                      |
| 4    | Running     | Ready       | $Process_0$ now done |
| 5    | -           | Running     |                      |
| 6    | -           | Running     |                      |
| 7    | -           | Running     |                      |
| 8    | -           | Running     | $Process_1$ now done |



En este ejemplo tenemos un comportamiento mas interesante, para no perder el tiempo ,cuando $Process_0$ se bloquea y deja de usar el CPU, es ocupado su lugar por $Process_1$ que al terminar le vuele a dejar su lugar.

| Time | $Process_0$ | $Process_1$ | Notes                                      |
| ---- | ----------- | ----------- | ------------------------------------------ |
| 1    | Running     | Ready       |                                            |
| 2    | Running     | Ready       |                                            |
| 3    | Running     | Ready       | $Process_0$ Initiates I/O                  |
| 4    | Blocked     | Running     | $Process_0$ Is blocked so $Process_1$ runs |
| 5    | Blocked     | Running     |                                            |
| 6    | Blocked     | Running     |                                            |
| 7    | Ready       | Running     | I/O done                                   |
| 8    | Ready       | Running     | $Process_1$ now done                       |
| 9    | Running     | -           |                                            |
| 10   | Running     | -           | $Process_0$                                           |

# 1.6 Estructuras de datos y procesos
Un sistema operativo ,como casi todo programa ,maneja estructuras de datos. Con respecto a los procesos el sistema maneja una muy importante llamada **Process List** (lista de procesos), que esta compuesta por todos los procesos que están listos en el momento, y también información útil sobre el proceso que se encuentra _corriendo_ ,así como de los procesos que se quedaron _Bloqueados_ y de los que el OS se va a tener que encargar de poner en _Ready_ una vez completen una llamada a I/0.
## 1.6.1 Como se ve esto en XV6

```C
	// the registers xv6 will save and restore
	// to stop and subsequently restart a process
	struct context {
		int eip;
		int esp;
		int ebx;
		int ecx;
		int edx;
		int esi;
		int edi;
		int ebp;
	};
	// the different states a process can be in
	enum proc_state { UNUSED, EMBRYO, SLEEPING,
	RUNNABLE, RUNNING, ZOMBIE };
	// the information xv6 tracks about each process
	// including its register context and state
	struct proc {
		char *mem; // Start of process memory
		uint sz; // Size of process memory
		char *kstack; // Bottom of kernel stack
	// for this process
	enum proc_state state; // Process state
	int pid; // Process ID
	struct proc *parent; // Parent process
	void *chan; // If !zero, sleeping on chan
	int killed; // If !zero, has been killed
	struct file *ofile[NOFILE]; // Open files
	struct inode *cwd; // Current directory
	struct context context; // Switch here to run process
	struct trapframe *tf; // Trap frame for the
	// current interrupt
	};
```

En el código se ve la implementación de la abstracción proceso que se mostró en los capítulos anteriores ,en este caso en XV6. Entre las partes mas importantes esta:
- *Register Context* : Que se encarga de que al detener un proceso que tenia que seguir ejecutando ,el estado actual de sus registros se guarde para poder continuar su ejecución en algún otro momento. Esta técnica/mecanismo se conoce como [[Limited Direct Execution#Context switch|Context Switch]]  y se ve en el [[CPU Virtualization#2. Process API|próximo capitulo]].
# [[CPU Virtualization|FIN]]
### Términos clave para este capitulo
• The *process* is the major OS abstraction of a running program. At
any point in time, the process can be described by its state: the con-
tents of memory in its *address space*, the contents of CPU registers
(including the program counter and *stack pointer*, among others),
and information about I/O (such as open files which can be read or
written).
• The *process API* consists of calls programs can make related to pro-
cesses. Typically, this includes *_creation, destruction, and other use-
ful calls._*
• *Processes exist in one of many different process states*, including
running, ready to run, and blocked. Different events (e.g., getting
scheduled or descheduled, or waiting for an I/O to complete) tran-
sition a process from one of these states to the other.
• A process list contains information about all processes in the sys-
tem. Each entry is found in what is sometimes called a process
control block (PCB), which is really just a structure that contains
information about a specific process.