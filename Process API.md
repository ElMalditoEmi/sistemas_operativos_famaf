# Intro
En UNIX hay dos formas importantes de crear procesos, sin embargo hay algunas mas de las que se explican acá, que derivan de estas pero que tienen algunas posibilidades mas concretas.


# 1. Creando procesos con `fork()`
La [[syscall]] fork() ,es una de las llamadas que crean procesos. En resumen al ser invocada en un programa que esta corriendo ,fork se encarga clonar el proceso y añadir este nuevo clon a la [[Procesos#1.6 Estructuras de datos y procesos#1.6.1 Como se ve esto en XV6|process list]] de forma que ahora tenemos 2 veces el mismo proceso ejecutando *Casi independientemente* el uno del otro, y ambos corren desde la instrucción siguiente a la llamada a _fork()_.

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
Si y para esto presentamos la siguiente [[syscall]].

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


