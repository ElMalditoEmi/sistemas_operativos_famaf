# Intro
Para lograr la [[Virtualization]] nuestro [[Sistema Operativo]] necesita lograr que se _comparta_ de forma eficiente entre los procesos nuestro recurso a virtualizar, la CPU. El [[Sistema Operativo]] tiene esta capacidad tan bien conseguida que hay _sensación de simultaneidad_.

A la hora de pensar como lo haría uno mismo hay varios problemas que mirar bien antes de presentar la implementación como una buena.
Primero que nada queremos asegurar el _mejor rendimiento_ ,que no se este _desperdiciando uso_ de nuestro recurso. A si mismo queremos la _maxima velocidad de calculo_.
Por otra parte ,aunque parezca mas trivial, hay que asegurarse de que todos _los programas terminen_ o al menos, que no _monopolicen_ nuestro recurso. O PEOR ,que el [[Sistema Operativo]] ,que también existe como proceso ,no pierda el control de los recursos.

Todo esto cobra mas sentido a medida que vayamos desarrollando el capitulo y proponiendo como llevar la tarea.