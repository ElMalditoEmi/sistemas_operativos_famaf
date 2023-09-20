### [[Limited Direct Execution#Trap and save the context|Trap and save the context]]
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