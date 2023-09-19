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
Cuando el usuario pide que se corra un programa, nuestro [[Sistema Operativo]] se encarga de alojarle algo de memoria, lo pone en la _lista de procesos_ ,pone las instrucciones del programa en la memoria ,localiza la _primera instrucción_ y hace un salto hasta esta _en User mode_. Una vez el programa termino ,el [[Sistema Operativo]] vuelve a tomar control y se pone en _Kernel mode_.

Antes de seguir sepamos que son estos modos.
#### ¿Que diablos es User y Kernel mode?
Primero ,_User mode_ es el estado en el que el CPU esta al momento de estar manejando un proceso ,lo importante a saber es que tiene un _Set de instrucciones_ limitado, y para realizar operaciones mas delicadas tiene que recurrir al _Kernel mode_, este otro tiene otro _Set de instrucciones_ y como ya dijimos se encarga de las cuestiones delicadas de manejar ,como por ejemplo administrar la memoria ,o organizar las estructuras del [[Sistema Operativo]] tales como la lista de procesos.

Es importante saber que ambos tienen _Contexto distinto_ con esto ,nos referimos a que nuestro CPU, tiene guardados en los _registros_ valores distintos cuando esta en _Kernel mode_ , y en _User mode_.

## 1.2 Problemas de la DE

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
Este programa nunca debería finalizar ,y el [[Sistema Operativo]] nunca podrá recuperar el control. Mas aún tener cosas como bucles infinitos podría no ser tan evidente como en este caso ,por lo que hay que solucionar esto.

### 1.2.3 Mantener el control 
Al darle a un procesos control absoluto del recurso ,nuestro [[Sistema Operativo]] lo pierde y ya no tiene forma de por ejemplo detener un programa y poner a ejecutar otro. Esto en consecuencia quita la posibilidad de tener el *sharing* que es vital para que se comporte como describimos en la introducción.

## 1.3 Demos soluciones: LDE([[Limited Direct Execution]])
Para empezar dar la solución a los problemas que trae la ejecución directa es evidente que hay que ponerle _limites_ a nuestros procesos ,para que no tengan el mismo poder que el [[Sistema Operativo]].