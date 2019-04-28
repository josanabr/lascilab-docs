# Docker + HTCondor - Univalle

## Resumen

En el presente archivo se describen pasos usados para la gestion y uso de 
contenedores en el centro de datos de alto rendimiento en la Universidad del 
Valle.

## Introducción

De hace un par de años para atrás en el cluster de la Universidad del Valle no se volvió a instalar software en los nodos de procesamiento del cluster.
La razón por la cual no se volvió a instalar software es por la gran cantidad de conflictos que los nuevos paquetes de software generaban y entonces dificultaban la normal operación de diferentes herramientas y aplicaciones que se usan por los académicos de la Universidad. 
Para eliminar estos conflictos en el software se pensó en los contenedores ya que estos ofrecen una solución tecnológica donde se "empaquetan" aplicaciones en un "archivo" y en dicho "archivo" se encuentran todas las librerías, dependencias y scripts necesarios para la operación normal de la aplicaciones usadas por los investigadores..

Una de las tecnologías más populares en el mundo de los contenedores es Docker.
En Univalle usamos esta herramienta para llevar a cabo la ejecución de las aplicaciones científicas requeridas por los investigadores de la Universidad del Valle.

En las siguientes secciones se dará una guía que muestra como usar la tecnología ofrecida por Docker para ejecutar aplicaciones en el cluster de Univalle.

## Consiguiendo el contenedor

Lo primero que ustede debe hacer es conseguir un contenedor que tenga desplegada la aplicación que usted desea usar. 
[Docker Hub](https://hub.docker.com) es un repositorio donde cientos de miles de usuarios comparten diversas aplicaciones científicas y servicios de red de código abierto desplegadas en contenedores de Docker.

Lo primero entonces que se debe hacer es conseguir el contenedor en **Docker Hub** que contenga la aplicación que usted requiere. 
Para efectos de esta demostración se hará uso de la aplicación [Octave](https://www.gnu.org/software/octave/).

Octave es un clon de MATLAB&copy; de código abierto y libre distribución.
Esto quiere decir que muchos de los scripts escritos para MATLAB&copy; funcionarán en Octave excepto que el script de MATLAB haga uso de algún tipo particular de *toolbox* no disponible en Octave.
Para este ejemplo se usará el contenedor de Octave que se encuentra en este [enlace](https://hub.docker.com/r/schickling/octave/).

En caso que la aplicación que usted requiere no se encuentra disponible en **Docker Hub** es una buena noticia porque podrá usted contribuir a la comunidad de Docker con un nuevo contenedor.
Existen muchos tutoriales de como llevar a cabo la creación de un nuevo contenedor o imagen de Docker. 
[Aquí](https://www.howtoforge.com/tutorial/how-to-create-docker-images-with-dockerfile/) usted podrá encontrar un tutorial de como hacerlo pero seguro si busca en **Google** podrá encontrar muchos sitios más.

## Uso de contenedores

### Local


Una vez creado el contenedor es hora de hacer uso de el. 
El comando para ejecutar un contenedor es:

```
docker run --rm -ti -v $(pwd):/source octaveimage /usr/bin/env octave /source/x.octave
``` 

A continuacion se describira, en forma de lista, los argumentos del comando `docker`

* `run` es el subomcando que permite la ejecucion de contenedores
* `--rm` indica que una vez se termine la ejecucion del contenedor se borre cualquier rastro de el
* `-ti`
* `-v $(pwd):/source` indica que el contenedor podra acceder al directorio desde donde se ejecuto el contenedor a traves del directorio `/source` dentro del contenedor
* `octaveimage` el nombre del contenedor que se va a ejecutar
* `/usr/bin/env octave /source/x.octave` es la linea de comandos que se espera ejecutar dentro del contenedor. Observe que se ejecutara un script en octave 
llamado `x.octave` y que se debe localizar en el directorio desde donde se lanza la ejecucion de este contenedor

<!--
Corriendo un script en Octave pasando argumentos
------
docker run --rm -ti -v $(pwd):/source octave /usr/bin/env octave /source/x.octave "hello world" (2)

Reusing a container to build other
------
demooctave/Dockerfile
docker build -t demooctave .

Using the previous container
------
docker run --rm -ti -v $(pwd):/source demooctave /usr/bin/env octave -qf /source/graphic.octave 10 /source/demo
-->

###  Con HTCondor

HTCondor es una herramienta de software que permite la ejecucion de tareas (desatendidas o no interactivas) en un ambiente de computacion oportunista.

Asumiendo que se desea enviar la ejecucion de una tarea con Docker y HTCondor,
este es el archivo de submission de la tarea:

```
universe 	= docker
docker_image	= 192.168.28.50:5000/demooctave/uv
executable 	= /bin/bash
arguments	= demooctave.sh 	
should_transfer_files   = YES
transfer_input_files    = demooctave.sh,graphic.octave 
when_to_transfer_output = ON_EXIT
output                  = out.$(Process)
error                   = err.$(Process)
log                     = log.$(Process)
queue 1
```

Note que en este caso se va a ejecutar un archivo llamado `demooctave.sh`, el cual es un script en Bash. 
Este es el contenido, para este ejemplo de dicho archivo.

```
#!/usr/bin/bash
/usr/bin/env octave -qf $1 $2 $3 #/otro/graphic.octave 10 /otro/demo
```

<!--
Referencias
------
(1) https://hub.docker.com/r/schickling/octave/ - Como correr un contenedor que mapea puertos de forma local
(2) https://www.gnu.org/software/octave/doc/v4.0.3/Executable-Octave-Programs.html - Como se pasan argumentos a un script en Octave
(3) https://docs.google.com/document/d/15aYKa8Hbml3aMGnGElHk2cVCgRzntRFKlMkOh0GMD9U/edit?usp=sharing
-->

