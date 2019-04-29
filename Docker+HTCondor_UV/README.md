# Docker + HTCondor - Univalle

## Resumen

En el presente archivo se describen pasos usados para la gestion y uso de 
contenedores en el centro de datos de alto rendimiento en la Universidad del 
Valle.

## Introducción

De hace un par de años para atrás en el cluster de la Universidad del Valle no se volvió a instalar software en los nodos de procesamiento del cluster.
La razón por la cual no se volvió a instalar software es por la gran cantidad de conflictos que los nuevos paquetes de software generaban y entonces dificultaban la normal operación de diferentes herramientas y aplicaciones que se usan por los académicos de la Universidad. 
Para eliminar estos conflictos en el software se pensó en los contenedores ya que estos ofrecen una solución tecnológica donde se "empaquetan" aplicaciones en un "archivo" y en dicho "archivo" se encuentran todas las librerías, dependencias y scripts necesarios para la operación normal de la aplicaciones usadas por los investigadores.

Una de las tecnologías más populares en el mundo de los contenedores es Docker.
En Univalle usamos esta herramienta para llevar a cabo la ejecución de las aplicaciones científicas requeridas por los investigadores de la Universidad del Valle.

**En las siguientes secciones se dará una guía que muestra como usar la tecnología ofrecida por Docker para ejecutar aplicaciones en el cluster de Univalle.**

## Consiguiendo el contenedor

Lo primero que ustede debe hacer es conseguir un contenedor que tenga desplegada la aplicación que usted desea usar. 
[Docker Hub](https://hub.docker.com) es un repositorio donde cientos de miles de usuarios comparten diversas aplicaciones científicas y servicios de red de código abierto desplegadas en contenedores de Docker.

Lo primero entonces que se debe hacer es conseguir el contenedor en **Docker Hub** que contenga la aplicación que usted requiere. 
Para efectos de esta demostración se hará uso de la aplicación [Octave](https://www.gnu.org/software/octave/).

Octave es un clon de MATLAB&copy; de código abierto y libre distribución.
Esto quiere decir que muchos de los scripts escritos para MATLAB&copy; funcionarán en Octave excepto que el script de MATLAB haga uso de algún tipo particular de *toolbox* no disponible en Octave.
**Para este ejemplo se usará el contenedor de Octave que se encuentra en este [enlace](https://hub.docker.com/r/schickling/octave/).**

En caso que la aplicación que usted requiere no se encuentra disponible en **Docker Hub** es una buena noticia porque podrá usted contribuir a la comunidad de Docker con un nuevo contenedor.
Existen muchos tutoriales de como llevar a cabo la creación de un nuevo contenedor o imagen de Docker. 
[Aquí](https://www.howtoforge.com/tutorial/how-to-create-docker-images-with-dockerfile/) usted podrá encontrar un tutorial de como hacerlo pero seguro si busca en **Google** podrá encontrar muchos otros tutoriales.

## Uso de contenedores

### Local

Para efectos de la demostración usaremos un script en Octave muy sencillo y lo único que hace es una serie de operaciones sobre unos vectores. 
Usando una terminal en su computador (en el cual debe estar instalado Docker) haga lo siguiente:

* Edite un archivo y copie el código abajo. Guarde estas líneas en un archivo llamado [`fitting.m`](fitting.m)

```
npts = 4
x = [ 1:npts ]'/npts
y = x + 0.1 * sin(10. * x)
disp(y)
```

* Ahora ejecute este script usando el contenedor que tiene desplegado Octave. La forma como se ejecuta este script con la herramienta **Docker** es de la siguiente forma:

```
docker run --rm -v $(pwd):/workspace -w /workspace schickling/octave octave fitting.m
```

* Al ejecutar este script debe obtener una salida por pantalla similar a la sigiuente:

```
GNU Octave, version 3.6.2
Copyright (C) 2012 John W. Eaton and others.
This is free software; see the source code for copying conditions.
There is ABSOLUTELY NO WARRANTY; not even for MERCHANTABILITY or
FITNESS FOR A PARTICULAR PURPOSE.  For details, type `warranty'.

Octave was configured for "x86_64-pc-linux-gnu".

Additional information about Octave is available at http://www.octave.org.

Please contribute if you find this software useful.
For more information, visit http://www.octave.org/help-wanted.html

Read http://www.octave.org/bugs.html to learn how to submit bug reports.

For information about changes from previous versions, type `news'.

npts =  4
x =

   0.25000
   0.50000
   0.75000
   1.00000

y =

   0.30985
   0.40411
   0.84380
   0.94560

   0.30985
   0.40411
   0.84380
   0.94560
```


###  Con HTCondor

HTCondor es una herramienta de software que permite la ejecucion de tareas (desatendidas o no interactivas) en un ambiente de computacion oportunista.

Asumiendo que se desea enviar la ejecucion del script visto en la [sección anterior](#local) se deben llevar a cabo los siguientes pasos:

* Ingresar con su usuario a **lascilab.univalle.edu.co**.

* Crear una carpeta, `fitting`, ingresar a ella.

* Script que ejecuta el script `fitting.m`. A este archivo asígnele el nombre [`fitting.sh`](fitting.sh).

```
#!/bin/bash
octave fitting.m
```

* A continuación se presenta el código de HTCondor para invocar la ejecución del contenedor de Octave con el script [`fitting.sh`](fitting.sh). A este archivo asígnele el nombre de [`fitting.condor`](fitting.condor).

```
universe 	= docker
docker_image	= schickling/octave
executable 	= /bin/bash
arguments	= fitting.sh 	
should_transfer_files   = YES
transfer_input_files    = fitting.sh,fitting.m
when_to_transfer_output = ON_EXIT
output                  = out.fitting.$(Process).txt
error                   = err.fitting.$(Process).txt
log                     = log.fitting.$(Process).txt
queue 1
```

* Para ejecutar la tarea se ejecuta el siguiente comando

```
condor_submit fitting.condor
```

* La salida de la ejecución del comando debe quedar en el archivo `out.fitting.0.txt`. 

## Video

En este [enlace](https://youtu.be/9FJrceXlP2c) usted podrá encontrar un video ejemplificando lo descrito en este documento.
