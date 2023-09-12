# Intro
En UNIX hay dos formas importantes de crear procesos, sin embargo hay algunas mas de las que se explican acá, que derivan de estas pero que tienen algunas posibilidades mas concretas.


## 1. Creando procesos con `fork()`
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


#  #Hay que explicar porque esto no es determinista y terminamos con fork()