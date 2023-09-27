# 1. Scheduling policies
Una _'política de encolamiento'_, es una decision del OS ante el estado actual del uso de recursos.
Si el _scheduler_ (mecanismo encargado del [[Scheduling]]) fuera una persona ,se preguntaría cosas como: "¿Este proceso esta ejecutando hace mucho tiempo? Entonces voy a poner a otro a correr." y otras cosas por el estilo.

#  2. (Preámbulo)Desarrollando políticas de encolamiento
### 2.1 Primero simplifiquemos el problema
Mientras desarrollemos la políticas a lo largo de las próximas secciones ,vamos a asumir algunas cosas que si bien son irrealistas, van a simplificar la tarea de mejorar nuestra intuición, ademas es mas fácil ir soltando estas falsas hipótesis de a poco, que hacerse lió con todo de entrada.

Así que durante las próximas secciones asumimos las siguientes hipótesis:
- 1 Cada trabajo de un proceso requiere la misma cantidad de tiempo.
- 2 Todos los trabajos _llegan al scheduler_ al mismo tiempo.
- 3 Una vez que empieza, el programa corre hasta que termina.
- 4 Ningún programa va a pedir _I/O_, por lo que no entra en estado _blocked_
- 5 _Sabemos el tiempo que lleva la ejecución de cada trabajo_
#### Nota:
Con _'trabajo'_ simplemente nos referimos a la tarea del proceso.


Hay que volver a hacer hincapié en que todo esto que vamos a asumir es irrealista ,o directamente imposible en el caso de la _5ta hipótesis_.

### 2.2 Métricas.
También vamos a necesitar algunas métricas para poder considerar con mejor criterio que política es mejor que otra.
Para todas estas tengamos en mente una linea del tiempo que muestra explícitamente que proceso esta corriendo en un instante de tiempo T.
Algo parecido a:

| Instante de tiempo $T$ | $T_0$                    | $T_1$                | $T_2$       | $T_3$               | $T_4$               | ... | $T_n$               |
| ---------------------- | ------------------------ | -------------------- | ----------- | ------------------- | ------------------- | --- | ------------------- |
| Running                | -                        | $Process_1$          | $Process_1$ | $Process_1$         | $Process_2$         | ... | $Process_2$         |
| Ready                  | $Process_1$ ,$Process_2$ | $Process_2$          | $Process_2$ | $Process_2$         |                     | ... |                     |
| Notes                  | Llegan 2 procesos        | $Process_1$  Empieza |             | $Process_1$ termina | $Process_2$ empieza | ...    | $Process_2$ termina |

#### 2.2.1 Formulas(¿?) para metricas
##### $T_{turnaround}$ (Tiempo de devolución)
Es el tiempo considerado desde que llego el proceso hasta que ejecuto su ultima instrucción y termino. En general la podemos calcular así:
$$\Huge T_{turnaround} = T_{Completion} - T_{arrival}$$
Donde:
$\qquad$ $T_{Completion}$ es el instante de tiempo en que se completo el trabajo.
$\qquad$ $T_{arrival}$ es el instante de tiempo en que llega el trabajo al _scheduler_.



# 3. Políticas
## 3.1 FIFO(First in,first out)
El algoritmo _mas básico_ para hacer _scheduling policies_ y probablemente el primero que se nos ocurre es este.
Ponemos a ejecutar los trabajos según quien vaya llegando primero, _FIFO_ viene a ser algo como _"El primero que entra,es el primero que sale"_.

_FIFO_ funciona realmente bien considerando las hipótesis que hicimos. Habría que verlo en acción $\huge\rightarrow$

Supongamos que llegan al scheduler 3 trabajos $A$, $B$ y $C$ ,cada uno tiene que ejecutar durante 10$T$ y encima lo hacen a casi ,casi (casi) el mismo tiempo.
Para _FIFO_ necesitamos que haya un primer un segundo y un tercero (en este caso son solo 3). Vamos a asumir arbitrariamente que FIFO puso la cola en orden alfabético aunque todo llego en simultaneo.

Entonces el _scheduling_ quedaría de la siguiente forma:

![[Pasted image 20230923000749.png|#invert]]

Calculando el $T_{turnaround}$ de cada proceso $A,B,C$ respectivamente tenemos 10,20,30.
_En promedio_ con esta política tenemos $T_{turnaround} = \frac{10+20+30}{3} = 20$.

Si bien pareciera que lo hace 'bien', en general para las políticas podemos presentar escenarios malos donde se ve claramente las flaquezas de la estrategia.
### Rompiendo la [[Scheduling#2.1 Primero simplifiquemos el problema|primera hipótesis]].
Si no podemos asumir que nuestros procesos duran todos lo mismo en una política [[Scheduling#3.1 FIFO(First in,first out)|FIFO]] nos podemos encontrar con una situación como:

$A,B ,C$ ejecutan durante $100,10,10$ respectivamente. Vamos a tener algo como:

![[Pasted image 20230926205012.png]]
Calculando el _promedio_ de $T_{turnaround} = \frac{100+110+120}{3} = 110$ que en general empeora mucho nuestros tiempos, no digamos situaciones donde A es aun mas grande.

Por si no se ve claro, el $T_{turnaround}$ de los procesos $B,C$ se ve muy desmejorado, a pesar de que son procesos de trabajos cortos en tiempo.

## 3.2 SJF(Shortest job first).
Después de ver el problema que trajo la política anterior, lo primero que se nos ocurre es poner los trabajos mas cortos primeros, de esta forma salvamos la situación anterior, mejorando los promedios, quedando:
![[Pasted image 20230926205710.png]]
Y mejorando el promedio de $T_{turnaround} = 110$ a un mucho mas aceptable promedio de $T_{turnaround} = \frac{10+20+120}{3} = 50$.

### Rompiendo la [[Scheduling#2.1 Primero simplifiquemos el problema|segunda hipótesis]].
Ahora que tenemos otra política que funciona, obviamente vamos a intentar romperla, para esto vamos a romper otra hipotesis.
A partir de ahora no vamos a asumir que todos los procesos llegan al mismo tiempo (Que es una situación razonable en cualquier [[Sistema Operativo]] con _time sharing_).

Supongamos que ahora $A,B,C$ corren por la misma cantidad de tiempo que antes $100,10,10$ respectivamente, pero en este caso $A$ llega en $T = 0$ ,pero ambos $B$  y $C$ lo hacen en $T=10$. Tenemos lo siguiente:

![[Pasted image 20230926210547.png]]

Como no sabemos por cuanto tiempo trabajan $B$ y $C$ porque todavía no llegan al _scheduler_ hasta $T=10$. Empezamos a correr $A$ , y entonces $B$ , $C$ se ven obligados a esperar de todas formas. Con esta situación el promedio vuelve a ser tan malo como el caso malo en _FIFO_.

## STCF(Shortest Time to Completion First).
En este caso vamos a romper una de las hipótesis, con el fin de mejorar nuestra política anterior, y salvando el caso que mostramos justo antes.
### Rompiendo la [[Scheduling#2.1 Primero simplifiquemos el problema|tercer hipótesis]].
Desde ahora, _no necesariamente_ cada proceso que inicia esta obligado a terminar todo su trabajo para darle el _CPU_ a otro procesos. Acá es donde entran los conceptos del [[Limited Direct Execution#1.4 Cambiar de proceso (y como recuperar el control)|capitulo anterior]], en particular las cuestiones que tienen que ver con _interrupts_.
Con esto le damos la capacidad al _scheduler_ de cambiar de un trabajo (quizás para continuar el otro después).

En la política _STCF_ vamos a poder mejorar el comportamiento de una política [[Scheduling#3.2 SJF(Shortest job first).|SJF]], en un caso como en el que empeoramos su promedio de $T_{turnaround}$. La idea ahora es, terminar de correr siempre primero los trabajos mas cortos, y después de estos los mas largos.
Cuando un procesos haya sido detenido por una _interrupt_ si no esta mas cerca de terminar que otro proceso, este ultimo es el que toma el lugar en el CPU.

Vamos como se comporta ahora nuestra nueva política. A fin de dejar mas clara la linea del tiempo, consideremos que el [[Sistema Operativo]] hace un [[Limited Direct Execution#Timer interrupt|Timer interrupt]] cada $10$ unidades de tiempo.
![[Pasted image 20230926212444.png]]
Al llegar, como no hay mas trabajo $A$ comienza la ejecución. Al cabo de $10T$ sucede la _timer interrupt_, $B,C$ que llegaron al scheduler un poquito antes que la interrupción están listos para ejecutar. Es entonces que la política tiene efecto, parando la ejecución de $A$ y dándole el procesador al proceso $B$ que esta mas cerca de completarse que $A$ ;Y al terminar $B$ haciendo la misma evaluación, inicia la ejecución de $C$. Por ultimo como $A$ es el único trabajo lo corre esta su fiscalización.
