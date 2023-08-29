La CPU virtualization permite hace referencia a varios mecanismos que le permiten a un [[Sistema Operativo]] levantar programas (procesos) ,y ejecutarlos en "simultaneo" de la mejor forma posible, asegurando que todos finalicen ,y que no provoquen problemas al rendimiento de los demás.

# 1 Procesos
# 1.1 Primero que nada:¿Que es un proceso?
Un proceso en si mismo es una abstracción que el [[Sistema Operativo|OS]] provee para si mismo ,en si un programa no es nada mas que una serie de lineas inentendibles que están en código maquina guardadas en algún lugar del disco. El sistema operativo es el que "le da vida" a estas instrucciones y se encarga de ejecutarlas ,crear un proceso es la acción del sistema operativo ejecutando un programa.

# 1.2 Process API
#### *Si bien no estamos listos para discutir sobre API's acá se introduce un poco la idea.*

Prácticamente todos (si es que no, todos) los [[Sistema Operativo|OS]] deberían ser capaces de incluir al menos las siguientes acciones para procesos (Vamos a ver después que esto es una API):
- **Crear**:Crear un proceso es algo que debe estar en una API de procesos (si no que sentido tendría), al llamar un programa en un [[Shell]] ,o hacerle click al icono, el OS se encarga de crear el programa que en efecto clickaste. ^05f158
- **Destruir**:Así como hay interfaz de [[CPU Virtualization#^05f158|creación]] ,de forma análoga vamos a querer que,una vez nuestros programas hayan finalizado con éxito (o no), no queden vestigios de nuestros programas estorbando a los demás ,mas adelante se discute de [[Zombie processes]].

- **Esperar**:Es muy util poder espera a que un proceso finalice.

- **Otros controles**:También esta bueno tener procesos que se **suspenden** en vez de simplemente morir y perder todo el progreso. Suspender un procesos lo hace dejar de correr y tener capacidad de continuar luego desde ese punto.

- **Mostrar el estado**:Saber sobre los procesos en muy util ,como vamos a ver mas abajo en [[Process scheduling]], para acomodar un proceso en un schedule de forma eficiente tenemos que saber cuanto tiempo lleva corriendo ,incluso algunas implementaciones querrán saber con cuanto esfuerzo se corre un programa.

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
_*NOTA:Esto esta mejor explicado cuando hablamos de scheduling.*_

Como podemos ver en el gráfico(pensado como diagrama de estados) nuestros proceso se acodan en un **Schedule** para ejecutarse en orden, si tenemos un CPU con un solo núcleo ,solo vamos a poder ir de a un proceso como en la figura 4.3.
![[Pasted image 20230829091227.png]]

En este ejemplo tenemos un comportamiento mas interesante, para no perder el tiempo ,cuando $Process_0$ se bloquea y deja de usar el CPU, es ocupado su lugar por $Process_1$ que al terminar le vuele a dejar su lugar.
![[Pasted image 20230829091614.png]]




# Términos clave para 1.X
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