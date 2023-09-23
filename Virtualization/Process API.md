#complete 
# Intro
En UNIX hay dos formas importantes de crear procesos, sin embargo hay algunas mas de las que se explican acá, que derivan de estas pero que tienen algunas posibilidades mas concretas.


# 1. Creando procesos con `fork()`
La [[Limited Direct Execution]] fork() ,es una de las llamadas que crean procesos. En resumen al ser invocada en un programa que esta corriendo ,fork se encarga clonar el proceso y añadir este nuevo clon a la [[Procesos#1.6 Estructuras de datos y procesos#1.6.1 Como se ve esto en XV6|process list]] de forma que ahora tenemos 2 veces el mismo proceso ejecutando *Casi independientemente* el uno del otro, y ambos corren desde la instrucción siguiente a la llamada a _fork()_.

Ambos programas pueden modificar sus variables como dueños absolutos y estas tampoco están vinculadas de uno a otro.
Parece medio inútil tener dos procesos que tratan de hacer lo mismo, pero si revisamos la manpage de _fork()_ ,vamos a ver que si guardamos el resultado de la llamada en una variable en el proceso "padre" el que llamo a *fork()* ,vamos a tener el _PID(Process ID)_ de su proceso "hijo" guardado en esta variable. Mientras que desde la perspectiva del proceso hijo ,tenemos en esa variable guardado el valor 0.

Ahora podemos diferenciar ambos procesos *padre e hijo* y podemos encargarles tareas distintas condicionalmente a partir de el valor del retorno de *fork()*.

Un ejemplo puede clarificar lo raro que es _fork()_ la primera vez que se explica.
```c
main(int argc, char *argv[])
{
    printf("hello world (pid:%d)\n", (int) getpid());
    int rc = fork(); //Guardamos fork en una variable
    if (rc < 0) {// Si la variable es menor que 0 es código de error
        // fork failed; exit
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) {// Si la variable esta en 0 ,es la perspectiva del hijo
        printf("hello, I am child (pid:%d)\n", (int) getpid());
    } else { //Este caso es >0,osea tener un int que representa el PID del hijo
			//Esta es la perspectiva del padre.
        printf("hello, I am parent of %d (pid:%d)\n",
	       rc, (int) getpid());
    }
    return 0;
}
```
## Ojito con el fork
Si bien puede parecer que el programa es inofensivo , tiene un problema bastante gordo ,_no es determinista_. Esto se evidencia cuando corremos muchas(*Muchísimas*) instancias del programa y veamos que a veces _aparece el mensaje del padre primer y luego el del hijo_ y en otras ocasiones _aparece primero el mensaje del hijo y luego el del padre_.
### ¿Porque pasa eso?
Es consecuencia de tener procesos paralelos ,en este programa es imposible determinar que proceso va a pedir el buffer para hacer _printf()_ primero.
### ¿Se puede evitar?
Si y para esto presentamos la siguiente [[Limited Direct Execution]].

---
# 2. Solucionando el problema anterior con wait()
La llamada _wait()_ se encarga de que ,una vez llamada en una sección de código ejecutado por el proceso padre, este se quede _literalmente a esperar_ que el proceso hijo haya finalizado su ejecución.
A continuación un *workaround* para que el código anterior se pueda decir determinista:

```c
main(int argc, char *argv[])
{
    printf("hello world (pid:%d)\n", (int) getpid());
    int rc = fork(); //Guardamos fork en una variable
    if (rc < 0) {// Si la variable es menor que 0 es código de error
        // fork failed; exit
        fprintf(stderr, "fork failed\n");
        exit(1);
    } 
    
    else if (rc == 0) {// Si la variable esta en 0 ,es la perspectiva del hijo
        printf("hello, I am child (pid:%d)\n", (int) getpid());
    } 
    
    else { //Este caso es >0,osea tener un int que representa el PID del hijo
			//Esta es la perspectiva del padre.
		wait();
        printf("hello, I am parent of %d (pid:%d)\n",
	    rc, (int) getpid());
    }
    return 0;
}
```

De esta forma nos aseguramos que el padre siempre imprima su mensaje después del hijo ,se ve obligado a espera que el hijo haya terminado de ejecutar.

---

# 3. Creando procesos 2 (la venganza)
Como ya vimos , con _[[Process API#1. Creando procesos con `fork()`|fork()]]_ podemos crear procesos que tengan un código fuente escrito por nosotros mismos (o absolutamente copiado de internet) ,pero existe una familia de syscalls que sirve para crear procesos a partir de otros programas que se encuentran listos para ejecutar en algún lugar de nuestro disco duro.

### La familia exec(),execv(),execvp()
Estas 3 llamadas tienen en común la idea que mencionamos recién ,levantar procesos hijos con programas ya compilados en algún lado ; Por ejemplo podemos hacer _ls_ para mostrar por consola los archivos en una carpeta.
Para hacer ese ejemplo vamos a usar _execvp_ que en particular se encarga de buscar los programas en el *PATH* de nuestro sistema de archivos, allí se encuentra el binario de _ls_.
### Cuidado como entendemos esto.
Esta familia de funciones al levantar el proceso con el programa que le pedimos, pisan el proceso actual que estamos corriendo ,por lo que el código que este después de un _exec()_ nunca se ejecuta (A menos que el _exec()_ haya fallado).

### Ejemplo
Lo único que pide execvp para ejecutar es:
	-El nombre del comando como un string
	-Una lista que contenga los argumentos que le queramos pasar

Notar que en la lista de argumentos también le pasamos el nombre del comando ,puede parecer medio redundante ,pero sin entrar en detalles esta hecho simplemente así.
También otra mágica decision de implementación que no vamos a discutir ,es poner _NULL_ al final ,simplemente háganlo.

```c
int main()
{
	char * arg_list[] = {"ls","-l",NULL}
	execvp("ls",arglist);

	return 1; //No llegamos nunca a este return si el execvp sale bien
}
```

De esta forma creamos de alguna manera un lanzador del programa _ls_.

### ¿Y si no quiero pisar mi proceso?
Bueno es normal ver que se combina el uso de ambos ,_exec()_ y _fork()_ para justamente este fin ,directamente vamos al ejemplo porque si se entiende y se experimenta bien con las 2 ,no hay mucha explicación que dar del siguiente programa.

```c
int main()
{
	pid_t fork_result = fork();

	if(fork_result == -1)
	{
		return 1; //No se pudo hacer el fork
	}
	else if(fork_result == 0)
	{
		char * arg_list[] = {"ls","-l",NULL}
		execvp("ls",arglist);
		//
		return 1; //SOLO EN CASO DE ERROR
	}
	else
	{
		wait();   //Tambien se puede esperar a los exec()
	}

	return 0;
}
```

Las instrucciones de este programa son:
- Crear un proceso hijo
- Pisar el proceso hijo con una instancia de _ls_
- Que el padre espere a ese hijo para finalizar la ejecución

Si hay dudas sobre este ultimo ejemplo se recomienda mucho mirar a fondo la familia _exec()_ y _fork()_ por separado , una vez hecho eso debería ser trivial el código que se presenta arriba.

---
# 4. Porque esta API esta buena
En esta sección hablamos un poco de cosas concretas de la API de procesos ,reivindicando las decisiones buenas que tomaron los que la implementaron.

## 4.1 Tener separado de fork() y los exec()'s
Es medio molesto tener que hacer un _fork()_ y después hacer _exec()_ para poder levantar otro proceso y poder seguir ejecutando otras instrucciones escritas por nosotros en código fuente **¿Porque no hacer todo esto en un solo comando?**

La respuesta es: porque perdemos cosas muy interesantes si hacemos eso. El tener estas dos llamadas como cosas separadas nos permite cambiar un poco el entorno del programa antes de hacer el exec sin que el padre tenga que llevar cuenta de estos cambios.

Por ejemplo si en el [[Process API#¿Y si no quiero pisar mi proceso?|programa del ejemplo anterior]] quisiéramos implementar una forma en la que el usuario ,pueda ingresar el comando que quiera con los argumentos que quiera, y que luego eso sea enviado al _execvp()_. Con ambas funciones separadas esto es posible ,ya que si no todo tendría que estar preestablecido de una forma en el padre y en cosas mas complejas puede llegar a ser mas molesto manejar la lógica sin poder abstraerse del proceso padre.


## 4.2 ??

# 5. Mas allá de lo básico
Como es de esperar la [[Process API]] es enormemente compleja y va mas allá de las syscalls  que pudimos introducir en el capitulo.

Por ejemplo hay otras que son importantes tales como _kill()_ que se encarga de literalmente matar procesos (una forma violenta de decir que paramos su ejecución), osea no hay nada tan simétrico como _acabar la vida de un proceso_ al que se le _dio vida en otra sección de código_ ,es análogo a la simetría de _malloc()_ y _free()_ con la que ya el lector debe estar familiarizado. Si bien no hizo falta hablar de _kill()_ mas arriba en el capitulo es porque [[UNIX]] es una maravilla y quienes los escribieron se encargaron de minimizar los protocolos que se podían minimizar.

Otras cuestiones que quedan abiertas en este cierre son las _señales_ y en particular estas rodean a la syscall _signal()_ ,otro tema enormemente profundo que tiene que ver mas con el kernel y las sutilezas que hacen que [[UNIX]] sea la bestia que es.

En estos dos tópicos estaría bueno que el lector ahonde por su cuenta ,ya que no hay un nexo muy claro con los capítulos que siguen ,al menos en el corto plazo.

# [[CPU Virtualization|FIN]]

# Términos útiles para investigar

- Each process has a name; in most systems, that name is a number
known as a *process ID (PID).*

- The _fork(_) system call is used in UNIX systems to create a new pro-
cess. The creator is called the parent; the newly created process is
called the child. As sometimes occurs in real life , the child
process is a nearly identical copy of the parent.

-  The _wait()_ system call allows a parent to wait for its child to com-
plete execution.

-  The _exec()_ family of system calls allows a child to break free from
its similarity to its parent and execute an entirely new program.

-  A *UNIX shell* commonly uses _fork()_, _wait()_, and _exec()_ to
launch user commands; the separation of fork and exec enables fea-
tures like _input/output redirection, pipes, and other cool features,_
all without changing anything about the programs being run.

-  _Process control is available in the form of signals_, which can cause
jobs to stop, continue, or even terminate.

-  Which processes can be controlled by a particular person is _encap-
sulated_ in the notion of a user; the operating system allows multiple
users onto the system, and ensures users can only control their own
processes.

-  A _superuser_ can control all processes (and indeed do many other
things); this role should be assumed infrequently and with caution
for security reasons.