README.md
=========

En este documento se hara una rapida revision de conceptos y herramientas de utilidad para aquellas personas que en el dia a dia tienen trabajos relacionados con la gestion de recursos Unix bajo el entorno Bash.

El '>'
======
Es un mecanismo que permite direccionar la salida de un comando en un archivo. Un caso especial de este mecanismo es el de '>>' que concatena la salida de un comando en un archivo existente. Si el archivo no existe entonces se crea.

	# ls > salida-ls
	# cat salida-ls
	# echo "hola mundo" >> salida-ls
	# cat salida-ls
	# wc salida-ls
	# wc -l salida-ls
	# wc -c salida-ls
	# wc -w salida-ls

- Que hace el comando `ls`?
- El archivo `salida-ls` de que tipo es?
- Que hace el comando `echo`?
- Que hace el comando `cat`?
- Que hace el comando `wc`? Que hacen los flags `-l`, `-c` y `-w`?

El pipe '|'
===========

Es un mecanismo usado en Bash para permitir la comunicacion entre procesos. Los procesos son entidades independientes dentro del sistema operativo y que en principio no tienen como comunicarse. Ejecute este comando

	# cat salida-ls | wc -l

+ De acuerdo a lo que ha aprendido, como contaria usted el numero de procesos que estan corriendo en su sistema operativo? 
**Hint**: El comando para mostrar los procesos del sistema es `ps -A`

Variables
=========
El interprete de comandos tiene la posibilidad de definir variables

	# export VAR1="hola mundo"
	# echo ${VAR1}

Se pueden almacenar valores numericos en las variables 

	# export VAR2=3
	# echo ${VAR2}

Se pueden definir variables que son vectores o arreglos

	# declare -a VAR3=('hola' 'mundo' 'hello' 'world')
	# echo ${VAR3[0]}
	# echo ${VAR3[@]}

Se puede extraer un subconjunto de elementos de un arreglo

	# echo ${VAR3[@]:1:2}

Mas detalles sobre variables tipo arreglo [aqui](http://www.thegeekstuff.com/2010/06/bash-array-tutorial).

- - -
Se puede guardar la salida de un comando en una variable

	# export LS=$( ls )
	# echo ${LS}

Se puede guardar en una variable el resultado de una operacion aritmetica

	# export TOTAL=$(( VAR2 + 4 ))

El operador '**' permite calcular la potencia de un par de numeros. 

+ Imprima por pantalla el valor de elevar 2 a la potencia 4. 
+ Guarde este resultado en una variable e imprima esta variable por pantalla


Condicional 'if'
================

Se pueden hacer validaciones sobre variables numericas

	# if [ $VAR2 -eq 3 ]; then 
		echo "es igual a 3"
  	else
		echo "no es igual a 3"
  	fi

Mas informacion sobre condicionales [aqui](http://www.thegeekstuff.com/2010/06/bash-conditional-expression/)

  + En el link anterior se encuentra la implemetanción de una calculadora. Agregue a dicha calculadora un metodo `potencia`, de modo que si el usuario digita `11 ** 12`, la calculadora imprima `3138428376721`

Tambien se pueden tener condicionales sobre cadenas de caracteres,

	# export STR1="hola"
	# export STR2="hola"
	# if [ "${STR1}" == "${STR2}" ]; then
		echo "son iguales"
	  else
		echo "no son iguales"
	  fi

Es importante observar los espacios en blanco dejados a proposito entre el simbolo `[` y `"${...`. Lo mismo sucede con el lado derecho de la expresion.

Ciclo 'for'
===========

Los 'loops' permiten iterar sobre una serie de valores,

	# for i in 1 2 3; do
		echo ${i}
	done

Este codigo imprimira por pantalla los valores que reciba la variable `i`.

Se puede iterar tambien sobre la salida de un comando

	# for i in $( ls -a ); do 
		echo "[${i}]"
	done

O sobre los valores de una variable arreglo

	# lsminusa=( $( ls -a ) )
	# for i in "${lsminusa[@]}"; do
		echo "[${i}]"
	  done

+ Escriba un codigo que guarde en la variable tipo arreglo `onlybash`, todos aquellos archivos que tengan la palabra `bash`
+ Escriba un codigo, usando el ciclo `for`, que sume los valores de la siguiente secuencia de numeros `1 1 2 3 5 8` e imprima el resultado final.

Comandos 'utiles'
=================

'grep'
------

Este comando se usa para filtrar la salida de un comando

	# ls -la ${HOME} | grep bash

Este comando muestra todos los archivos que en el directorio `${HOME}` tienen la palabra `bash`.

	# ls -la ${HOME} | grep -v bash

Este comando muestra todos los archivos que en el directorio `${HOME}` NO tienen la palabra `bash`.

El flag `-i` en el comando `grep` indica que se ignore si es mayuscual o minuscula.

'tail'
------

Este comando permite ver la salida de las ultimas lineas de un comando

	# ls -la | tail -n 3

Muestra las ultimas 3 lineas de la salida del comando `ls -la`

	# ls -la | tail -n +3

Muestra todas las lineas de la salida del comando `ls -la` EXCEPTO las tres primeras.

'head'
------

Este comando permite ver la salida de las primeras lineas de un comando

	# ls -la | head -n 3

Muestra las primeras 3 lineas de la salida del comando `ls -la`

	# ls -la | tail -n -3

Muestra todas las lineas de la salida del comando `ls -la` EXCEPTO las tres ultimas.

'tr'
----

Este comando permite sustituir un caracter por otro en la salidad de un comando

	# ls -la | tr -s ' ' '|'

Este comando convierte todos los espacios en blanco de la salida del comando `ls -la` en `|`

Scripts y funciones
======================

En Bash es posible escribir programas (o scripts) donde se almacenan un conjunto de instrucciones de Bash y se ejecutan de una sola vez como si fueran un comando mas del sistema. Abra un nuevo archivo y digite lo siguiente

	#!/bin/bash
	echo "hola mundo"

Guarde el contenido en ese archivo (e.g. `hola.sh`) y desde la linea de comandos ejecute `source ./hola.sh`. 
La extension `.sh` no es necesaria pero se usa como convencion para que las demas personas sepan que ese archivo es un script en shell.

Usted puede pasar argumentos a sus scripts. 
Modifique su script de la siguiente manera

	#!/bin/bash
	echo "hola ${1}"

Ahora ejecute el script de la siguiente forma

	# source ./hola.sh john
	# source ./hola.sh "john sanabria"
	# source ./hola.sh 

Usted puede validar el numero de argumentos que se le pasan a un script con la variable `$#`. Si no se le pasan argumentos al script entonces el valor es `0`, si se le pasa un argumento entonces el valor es `1` y asi sucesivamente.

+ Modifique el script anterior de modo que si el usuario pasa un argumento imprima `hola ` mas la cadena que le pasaron como argumento, de lo contrario imprima `hello`.

Dentro de un script usted puede definir funciones

	#!/bin/bash

	function demo() {
		echo "hola mundo"
	}

	demo

Tambien se le pueden pasar argumentos a las funciones

	#!/bin/bash

	function demo() {
		echo "hola ${1}"
	}

	demo 
	demo "mundo"
	demo "john"


+ Escriba un script en bash que defina un funcion llamad `pow` y que reciba dos argumentos y eleve el primer argumento a la potencia del segundo. Si el usuario no pasa los argumentos adecuados entonces imprime por pantalla un mensaje de error.

<!---
* Se muestra como es la creacion de maquinas virtuales con VBoxManage
-->

* Haga un script en bash que de una maquina virtual muestre,
	- RAM usada
	- CPU asignados
	- Numero de tarjetas de red y de que tipo son

* Haga un script en bash que indique el numero de maquinas virtuales que estan corriendo

* Haga un script en bash que indique la cantidad de RAM que un conjunto de maquinas virtuales en ejecucion esta consumiendo
