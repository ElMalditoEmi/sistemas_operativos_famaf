---
cssclasses:
  - '"scheduling"'
aliases:
---
# 1. Scheduling policies
Una _'política de encolamiento'_, es una decision del OS ante el estado actual del uso de recursos.
Si el _scheduler_ (mecanismo encargado del [[Scheduling]]) fuera una persona ,se preguntaría cosas como: "¿Este proceso esta ejecutando hace mucho tiempo? Entonces voy a poner a otro a correr." y otras cosas por el estilo.

#  2. (Preámbulo)Desarrollando políticas de encolamiento
### 2.1 Primero simplifiquemos el problema
Mientras desarrollemos la políticas a lo largo de las próximas secciones ,vamos a asumir algunas cosas que si bien son irrealistas, van a simplificar la tarea de mejorar nuestra intuición, ademas es mas fácil ir soltando estas falsas hipótesis de a poco, que hacerse lió con todo de entrada.

Así que durante las próximas secciones asumimos las siguientes hipótesis:
- 1 Todos los trabajos _llegan al scheduler_ al mismo tiempo.
- 2 Cada trabajo de un proceso requiere la misma cantidad de tiempo.
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
# 3.1 FIFO(First in,first out)
El algoritmo _mas básico_ para hacer _scheduling policies_ y probablemente el primero que se nos ocurre es este.
Ponemos a ejecutar los trabajos según quien vaya llegando primero, _FIFO_ viene a ser algo como _"El primero que entra,es el primero que sale"_.

_FIFO_ funciona realmente bien considerando las hipótesis que hicimos. Habría que verlo en acción $\huge\rightarrow$

Supongamos que llegan al scheduler 3 trabajos $A$, $B$ y $C$ ,cada uno tiene que ejecutar durante 10$T$ y encima lo hacen a casi ,casi (casi) el mismo tiempo.
Para _FIFO_ necesitamos que haya un primer un segundo y un tercero (en este caso son solo 3). Vamos a _romper la primera hipótesis_ y vamos a asumir arbitrariamente que A llego fracciones de segundo ,antes que B y B antes que C.

Entonces el _scheduling_ quedaría de la siguiente forma:

![[Pasted image 20230923000749.png|#invert]]
