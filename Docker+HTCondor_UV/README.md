# Docker + HTCondor - Univalle

## Resumen

En el presente archivo se describen pasos usados para la gestion y uso de 
contenedores en el centro de datos de alto rendimiento en la Universidad del 
Valle.

Los pasos para gestionar y usar contenedores son los siguientes:

* Creacion de contenedores
* Subir contenedor a *registry*
* Uso de contenedores
  - Local
  - con HTCondor

## Creacion de contenedores

La forma como se sugiere hacer la creacion de contenedores es a traves de un 
Dockerfile como el que se presenta a continuacion:

```
FROM octaveimage

MAINTAINER John Sanabria
RUN apt-get update
RUN apt-get -y install octave-image
```

Para llevar a cabo la creacion del contenedor se ejecuta el comando

```
docker build -t octaveimage .
```


## Subir contenedor a registry

El termino 'Subir contenedor' hace referencia al publicar un contenedor en un
repositorio accesible por terceras personas o servicios. 
En este caso el repositorio se conoce como *registry* en el contexto de Docker.

Antes de hacer el proceso de subir el contenedor es necesario ponerle una marca en la cual se indica explicitamente el nombre o IP del *registry* donde va a ser publicado.
A continuacion se pone una marca sobre el contenedor creado anteriormente

```
docker tag octaveimage 192.168.28.50:5000/octaveimage/uv
```

Para poner la marca a un contenedor se usa el subcomando `tag` de Docker.
Luego se indica el nombre del contenedor a marcar (`octaveimage`) y finalmente
se indica el IP y el puerto del servidor registry, acompanado de su nombre
`octaveimage/uv`.


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

