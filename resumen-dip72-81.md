## Planificador de tiempo-real
Los procesos de tiempo-real ejecutables que existan en el sistema se ejecutarán antes que el resto, salvo que exista uno con mayor prioridad. Los algoritmos de planificación de tiempo real dependen de:
1. Cuándo el sistema realiza análisis de planificabilidad.
2. De hacerlo, si se realiza estática o dinámicamente
3. El resultado del análisis porudce un plan de planificación acorde al cual se desarrollarán las tareas en tiempo de ejecución.

La cola de ejecución es simple, como se aprecia en la figura:
(poner foto diapos73)

El *mapa de bits* permite seleccionar la cola con procesos en 2 (64 bits) o 4 (32 bits) instrucciones en ensamblador.

## Planificado en sistemas multipocesadores
En los sistemas multipocesador más tradicionales, los procesos no se vinculan a los procesadores. Hay una única cola para todos los procesadores o, si se utiliuza algún tipo de esquema vasado en prioridades, hay múltiples colas basadas en prioridad, alimentando a un único colectivo de procesadores. Puede verse como una arquitectura de colas multiservidor. A la hora de planificar, hay que considerar algunos aspectos:
1. Compartición de la carga de CPU imparcial
2. Afinidad de una tarea por un procesador
3. Migración de tareas sin sobrecarga.

En resumen, se realiza igual que en monoprocesadores, pero se tiene en cuenta el número de CPUs, la asignación y la liberación de proceso y procesador.

## Gestión de la energía
En los diseños actuales es importante considerar la gestión de la potencia, con el objetivo de reducir los costes de consumo de energía y los costes de refrigeración. Esta gestión se realiza a dos niveles:
* Nivel de CPU: P-states, C-states y T-states.
* Nivel de SO: CPUfreq (paquetes cpufrequtils y cpupower) y planificación.

Los procesadores actuales pueden funcionar a diferentes voltajes. A mayor voltaje, más alta es la frecuencia de funcionamiento y por tanto, mayor rendimiento. Cuando hay carga máxima del sistema, el procesador aumenta el voltaje para incrementar la frecuencia y por el contrario, cuando está ocioso (poca carga de trabajo), la disminuye para ahorrar energía.

### Espeficación ACPI
Todos los procesadores actuales cumplen la especificación ***Advanced Configuration and Power Interface***, que se define como la especificación abierta para la gestión de potencia y gestión térmica, controladas por el SO. Se definen cuatro estados Globales (*G-Estados*):
* G0: estado de funcionamineot (estados-C y estados-P)
* G1: estado dormido (S-estados)
* G2: estado apagado soft
* G3: estado apagado mecánico

### Estados de la CPU.
* S-estados: (dormidos en G1). Van de S1 a S5. Son estados en los que se van apagando diferentes componentes del ordenador, empezando por los que más consumen.
* C-estados: son los estados de potencia en G0.C0 (activo), C1(halt), C2(stop), C3(deep sleep)... 
* P-estados: el voltaje del procesador se modifica a escalones, no de forma continua. Por, ejemplo, pasar de 5V a 3V. Éstos dependen del procesador.
* T-estados: son los conocidos como *estados throttles*. Estań relacionados con la gestión técnica, como estabilizar la temperatura, pero no se dedican a la gestión de energía. Introducen ciclos ociosos.

### Estructura CPUfreq
* El **subsitema CPUfreq** es el responsable de ajustar explícitamente la frecuencia del procesador.
*  Es una estructura modularizada que separa políticas (gobernadores) de mecanismos (*drivers* específicos de CPUs).

(Poner imagen de la diapositiva 80)

#### Gobernadores
Permiten gestionar la potenciar en base a unos criterios. Es decir, según el gobernador que estemos utilizando, la gestión se hará de una determinada manera. Son los siguientes:
* **Perfomance** - mantiene la CPU a la máxima frecuencia posible dentro de un rango especificado por el usuario
* **Powersave** - intentan ahorrar energía, manteniendo la CPU a la menor frecuencia posible dentro de un rango
* **Userspace** -exporta la información disponible de la frecuencia a nivel usuario (sysfs) permitiendo su control al mismo.
* **On-demand** - ajusta la frecuencia dependiendo del uso actual de la CPU.
* **Conservative** - funciona al igual que el *on-demand*, pero los ajustes son menos agresivos.

Existen una serie de herramientas para la gestión de energía y potencia del sistema, como cpufreqtils, cpupower, powerTop y TLP.

### Planificación y energía. Algoritmos
* En CPM con recursos compartidos entre núcleos de un paquete físico, el rendimiento máximo se obtiene cuando el planificados distribuye la carga equitativamente entre todos los paquetes.
* En CPM sin recursos compartidos entre nñucleos de un mismo paquete físico, se ahorrará energía sin afectar al rendimiento si el planificados distribuye primero la carga entre núcleos de un paquete, antes de buscar paquetes vacíos.

El administrador puede elegir entre dos algoritymos de planificación, bien optar por un rendimiento óptimo del sistema (y por tanto, mayor consumo de energía) o bien priorizar el ahorro de energía sobre el rendimiento. Se realiza modifricando las entradas *sched_mc_power_saving* y *sched_smt_power_saving*.

* Si se opta por rendimiento óptimo, el gestor asigna cada hebra de proceso a un núcleo, teniendo así una sola hebra por núcleo. De esta forma no hay colisión entre hebras, aumentando la velocidad al no haber colisión entre cachés.
* En el modelo de ahorro de energía se colocan dos hebras en cada procesador, pudiendo así apagar alguno de los procesadores para ahorrar energía.

Por ejemplo, en un procesador quad-core (4 núcleos) ponemos cada hebra en un núcleo. Al estar todos los procesadores trabajando, aumenta el rendimiento, pero tmabién el consumo. Si optamos por ahorrar energía, el gestor coloca dos hebras en cada núcleo, quedando, por tanto, dos núcleos libres, que se apagan para ahorrar energía. Como desventaja, al tener dos núcleos con dos hebras cada uno, el rendimiento disminuye.

(Poner foto de la diapositiva 84)


## Grupos de control  
El planificador trata con ***entidades planificables***, cada una de las cuales es un proceso o un grupo de procesos (tareas). El tiempo de CPU se reparte entre las distintas entidades, a cada una se le asigna un porcentaje de tiempo de CPU y gracias al planificador sabemos que se va a respetar este tiempo.  
Cada entidad tiene un determinado tiempo asignado y este a su vez se reparte equitativamente entre todas las tareas de la entidad. Por ejemplo, si la entidad tiene el 30% de la cpu cada proceso de esta tendrá un 10% de la CPU.
Esto permite definir grupos de planificación (los cgroups): Diferentes procesos se asignan a diferentes grupos. El planificador reparte la CPU imparcialmente entre grupos, y luego entre procesos de un grupo. Así, se reparte imparcialmente la CPU entre usuarios.

Un ***grupo de control*** (abreviado como cgroup) es una colección de procesos que están vinculados por los mismos criterios y asociados con un conjunto de parámetros o límites. Estos grupos son jerárquicos, lo que significa que cada grupo hereda los límites de su grupo padre.  Definen jerarquías en las que se agrupan los procesos de manera que un administrador puede determinar con gran detalle la manera en la que se asignan los recursos  o llevar la contabilidad de los mismos. El kernel proporciona acceso a múltiples controladores (también llamados subsistemas) a través de la interfaz cgroup; por ejemplo, el controlador "memory" limita el uso de la memoria, "cpuacct" contabiliza el uso de CPU, etc. Gracias a los cgroups los recursos de hardware se pueden dividir adecuadamente entre tareas y usuarios, aumentando la eficiencia general.

**Los grupos de control suministran un mecanismo para**:  
**Asignar/limitar/priorizar recursos**: CPU, memoria, y dispositivos.  
No sólo podemos limitar el acceso a CPU como hemos visto sino también a otros recursos como memoria y dispositivos de E/S.  
**Contabilidad**: medir el uso de recursos.  
Contabilizar el uso de recursos para obtener estadísticas de los mismos.  
**Aislamiento**: espacios de nombres separados por grupo.  
**Control**: congelar grupos o hacer checkpoint/restart. Permite congelar una máquina virtual y restaurarla posteriormente.  
**de una colección de procesos.**  

Los Cgroups están organizados jerárquicamente, como los procesos. La diferencia fundamental es que pueden existir muchas jerarquías diferentes de cgroups simultáneamente en un sistema, mientras que  en el modelo de proceso de Linux hay un único árbol de procesos. El modelo de cgroup es uno o más árboles de tareas separados e inconexos .
Se necesitan múltiples jerarquías separadas de cgroups porque cada jerarquía está conectada a uno o más subsistemas.

Un *subsistema* (también llamado *controlador de recursos* o simplemente controlador) representa un único recurso, como el tiempo de CPU o la memoria. La definición de un subsistema es bastante general: es algo que actúa sobre un grupo de tareas, es decir, procesos.

### Subsistemas disponibles:

**blkio**: este subsistema establece límites en el acceso de entrada y salida hacia y desde dispositivos de bloques, como unidades físicas (disco, USB).  
**CPU**: este subsistema usa el planificador para proporcionar el acceso de las tareas de un cgroup a la CPU.   
**cpuset**: este subsistema asigna CPUs individuales (en un sistema multinúcleo) y nodos de memoria a tareas en un cgroup.  
**cpuacct**: este subsistema genera informes automáticos sobre los recursos de la CPU utilizados por las tareas en un cgroup.   
**Freezer**: suspende o reanuda las tareas en un cgroup.  
**Dispositivos (devices)**:  permite o niega el acceso a los dispositivos a las tareas en un cgroup.  
**Memoria (memory)**: este subsistema establece límites en el uso de memoria por las tareas de un cgroup y genera informes automáticos sobre los recursos de memoria utilizados por esas tareas.  
**net_cls**: etiqueta paquetes de red con un identificador de clase (classid) que permite al controlador de tráfico (tc) identificar los paquetes originados en una tarea de un grupo.  
**ns**: el subsistema del espacio de nombres.

El subsistema "ns" se agregó en el desarrollo de cgroups para integrar espacios de nombres y grupos de control. Si el cgroup "ns" estaba montado, cada espacio de nombres también crearía un nuevo grupo en la jerarquía de cgroup. Este fue un experimento que luego se consideró que no era adecuado para la API de cgroups y se eliminó del kernel.

### Los grupos de control se pueden usar de múltiples maneras:

+ Accediendo manualmente al sistema de archivos virtual (seudo-sistema de archivos) cgroup (groupsfilesystem).
+ Creando y administrando grupos sobre la marcha usando herramientas como cgcreate, cgexec y cgclassify de la librería libcgroup que permiten crear un grupo de control, ejecutarlo, clasificarlo, etc.  
El problema de estas dos formas de uso es que no son persistentes en el tiempo, es decir, cuando apago la máquina se pierde la configuración. Si queremos que sea persistente utilizamos:  
+ el "rules engine daemon" (demonio engine rules) que puede mover automáticamente procesos de ciertos usuarios, grupos o comandos a cgroups como se especifica en su archivo de configuración. Es una hebra kernel que lee un archivo de configuración donde se establecen los grupos de control y los límites de cada grupo (de CPU, memoria,…) que se deseen. El demonio lee el archivo de configuración cuando se arranca la máquina y crea los cgroups que en él se indican. Como el fichero se mantiene entre arranques del sistema la configuración de los cgroups es persistente.
+ Indirectamente a través de otro software que usa cgroups, como Docker, virtualización de Linux Containers (LXC), libvirt, systemd, Open Grid Scheduler / Grid Engine,y lmctfy de Google. No se controlan  los cgroups directamente sino a través de este software que actúa sobre los grupos de control.

### Para gestionar los grupos de control con el primer método hacemos lo siguiente:

1. Primero monto un subsistema como si fuera un sistema de archivos (el sistema de archivos que vamos a montar es cgroup). Los subsistemas se habilitan como una opción de montaje (-o subsistema que queremos controlar) de cgroupfs:  
`mount -t cgroup -o$subsistema`
2. Habilitamos los archivos del subsistema en cada cgroup (creando un directorio):  
`/dev/cgroup/migrupo/subsysA.optionB`
3. Cuando los tenemos definidos podemos verlos en `/proc/cgroups` (aquí están los grupos que se han definido y cómo están configurados)  
4. En Ubuntu, podemos instalarlo con  
`$ sudo aptitude install cgroups-bin libcgroup1`
5. Esto nos monta por defecto los siguientes fs:  
`$ ls -l /sys/fs/cgroup  
cpu cpuacct devices memory`  
+ Podemos ver los grupos de control con cat /proc/cgroups  

**Ejemplo**: crear dos grupos para asignar tiempo de CPU (tenemos dos aplicaciones, navegadores y reproductores de música. El navegador puede necesitar menos consumo de cpu que los reproductores de música por ello le asignamos menos tiempo de cpu):  
Creamos el correspondiente subdirectorio en sys/fs/cgroup/cpu:  
`$ mkdir Navegadores; mkdir multimedia`  
Asignamos el porcentaje de cpu al grupo escribiendo en el archivo cpu.shares.  
Escribimos el peso 2048, que es el que le vamos a dar al grupo de los reproductores de música, en este fichero:  
`$ echo 2048 > /sys/fs/cgroup/cpu/multimedia/cpu.shares`  
A los navegadores le asignamos la mitad del peso anterior:  
`$ echo 1024 > /sys/fs/cgroup/cpu/multimedia/cpu.shares`  
Movemos una tarea al cgrupo escribiendo su PID en el archivo tasks.  
`Firefox&`   
Miro el Pid del navegador con echo $! y lo escribo en el fichero tasks. Los procesos cuyo pid aparezca en este fichero consumirán la mitad de cpu que los que escribamos en el otro fichero  
`echo $! > /sys/fs/cgroup/cpu/navegadores/tasks  
mplayer micancion.mp3&`   
Los procesos en este fichero consumen más cpu que los del grupo navegadores. El planificador garantiza que tienen le doble de cpu.  
`echo $! > /sys/fs/cgroup/cpu/multimedia/tasks`  


