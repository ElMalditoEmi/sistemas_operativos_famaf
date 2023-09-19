La CPU virtualization permite hace referencia a varios mecanismos que le permiten a un [[Sistema Operativo]] levantar programas (procesos) ,y ejecutarlos en "simultaneo" de la mejor forma posible, asegurando que todos finalicen ,y que no provoquen problemas al rendimiento de los demás.

# Contenido
## 1. [[Procesos]]
En esta parte vamos a estudiar los aspectos principales que le permiten a un [[Sistema Operativo]] correr un programa y _que son realmente los programas para el OS_ ,mas allá de binarios con un contenido casi molesto a la vista del usuario común.
## 2. [[Process API]]
Cuales son las[[syscall|syscalls]] involucradas en al creación y destrucción de los procesos. Y algunas otras características referentes a esto y la implementación que hay de esto en los sistemas [[UNIX]].

## 3. [[Limited Direct Execution]]
Este es un capitulo esencial de entender y revistar,  por encima de eso puede ser _particularmente denso_ ,porque esta íntimamente relacionado con la siguiente parte de [[Virtualization]] y no se completa todo el dibujo hasta haber visto ambos.

Ahora si vamos a empezar a entender que hace el OS para correr un proceso.

Los conceptos vitales que vamos a ver son _Kernel mode_ ; _User mode_ ; la mismísima _LDE([[Limited Direct Execution]] )_ ; _Traps/Trap handlers_ ; _Context switch_ ;_Interrupts_ ; Entre otros.