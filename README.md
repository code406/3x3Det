# 3x3Det
3x3 matrix determinant calculator for 8086 Assembler

WARNING: Spanish ahead!

﻿PRACTICA 2: EXPLICACION DEL CODIGO PASO POR PASO

David Palomo Marcos
Ana Roa González


El programa imprime un texto explicativo para que el usuario elija la opción 1 o 2.
	SELECCIONA OPCION DE PROGRAMA:
	1)Calcular el determinante con valores por defecto
	2)Calcular el determinante con valores introducidos por teclado

El usuario deberá introducir 1 o 2, y pulsar Enter. La opción 2 aprovecha el código de la opción 1, por lo que este se explicará primero:



********** OPCION 1 **********************************************************

El cálculo del determinante se realiza con el método general (que no por el de Sarrus):
“El determinante es la suma de los productos de los elementos de una fila por sus adjuntos.”

Esto significa que en una matriz 3x3 como esta:

	| a11   a12   a13 |
	| a21   a22   a23 |
	| a31   a32   a33 |

El determinante se calcula como:

	   a11 * (a22 * a33  -  a32 * a23)
	-  a12 * (a21 * a33  -  a31 * a23)
	+  a13 * (a21 * a32  -  a31 * a22)
	----------------------------------
		    Resultado del determinante

En adelante, nos referimos a “cabeza” al hablar de a11, a12 y a13, que se multiplican por los adjuntos (resultado de lo que está entre paréntesis).
El resultado de cada multiplicación grande se guarda en una tabla de palabras llamada RES.

Estos cálculos se aplican a una tabla de dígitos decimales (bytes) llamada MDEC (matriz decimal).
Para acceder a cada elemento de la tabla se utiliza el direccionamiento base-indexado. Ejemplo: MOV AL, MDEC[BX][SI]

Para evitar hacer demasiados “MOV” antes de todos los accesos, se van incrementando o decrementando BX y SI según es necesario.
Las multiplicaciones se hacen con IMUL, para poder utilizar números negativos.

En el caso de la multiplicación de la cabeza (un byte de MDEC) por el resultado del adjunto (guardado en AX), es necesario extender en signo la cabeza para que los tamaños coincidan.
Para ello se mueve la cabeza a CH y se desplaza CX un byte hacia la derecha mediante SAR. Después, se realiza la multiplicación con CX:

	MOV CH, MDEC[BX][SI]
	MOV CL, 8
	SAR CX, CL
	IMUL CX

Los 3 resultados almacenados en RES se suman y restan como se mostró anteriormente para dar el resultado final del determinante, guardado en la variable FIN.

Para imprimir por pantalla el resultado, se hace el complemento a 2 si es negativo (NEG AX), y se separan los dígitos de FIN de la siguiente forma:

	mientras (cociente distinto de 0):
		divide el resultado entre 10
		se obtiene el resto
		se convierte el resto a ascii (sumar 48)
		se escribe el resto en la linea M2

Los restos se introducen en la variable M2, que es la segunda línea que se imprime. Además, si el resultado es negativo se imprime un “-” en M2.

Por último, se imprimen las 3 líneas (M1, M2 y M3) y el programa termina.



********** OPCION 2 **********************************************************

En caso de que el usuario escoja la opción 2 al inicio del programa, se sobreescribirá la matriz decimal con los números que introduzca antes de realizar los cálculos, pero los cálculos serán los mismos que se acaban de explicar.

Los números introducidos serán separados por espacios, y podrán ser desde -16 hasta 15. El usuario puede introducir hasta 37 caracteres (incluido el Enter). Estos se leen con la función 0AH y se guardan en la variable NUMEROS (en ASCII).

Mediante un bucle (BUCLE9) se obtienen de NUMEROS los 9 valores decimales que se han de introducir a la matriz MDEC. También se aprovecha el bucle para introducir los valores ASCII en las líneas de impresión M1, M2 y M3. Para este paso a las líneas, se guarda todo lo que introduce el usuario en una variable llamada INPUT con el formato de impresión, y luego se copia por trozos a las líneas.

Utilizamos BX como iterador y calculamos a partir de él la posición (DI) en que debemos insertar el número en INPUT.
Dentro del BUCLE9, necesitaremos otro bucle (CONVERTIR) para leer cada número introducido. Este bucle CONVERTIR termina cuando se lee un espacio o un caracter Enter.
En el bucle CONVERTIR se identifica si el número es positivo o negativo (según su primer caracter), y si es negativo se guarda en AL el valor 0C2H a modo de flag. Este flag nos indica si posteriormente debemos hacer el complemento a 2 del número o no. La última cifra del número (la de menor peso) quedará guardada en DL.

Debemos analizar qué tipo de número es: hay números positivos de un dígito y de dos dígitos, y números negativos de un dígito (ocupa dos caracteres) y de dos dígitos (ocupa 3).

Para saber el tipo de número, miramos cuántas iteraciones hizo el bucle CONVERTIR. Este número de iteraciones se guarda en CX.

1. Si es un número positivo de un dígito, el usuario habrá introducido un caracter y un espacio, por lo que CX = 2. En este caso, introducimos en la variable INPUT dos espacios (20H) y la única cifra del número (DL).
2. Si es un número negativo de dos dígitos, el usuario habrá introducido un "-", dos dígitos y un espacio, por lo que CX = 4. En este caso, introducimos en INPUT el caracter ASCII "-" (2DH), el valor en ASCII del número 1 (49) y la última cifra del número (DL).
3. Si es un número de 2 caracteres, puede ser positivo de 2 cifras o negativo de 1 cifra. Para saber su tipo, miramos el flag anterior (AL) que contiene 00 o 0C2H.
	3.1 Si AL = 0C2H, es negativo de 1 cifra: Introducimos en INPUT un espacio, un "-" y su única cifra (DL)
	3.2 Si no, es positivo de 2 cifras: Introducimos en INPUT un espacio, el valor ASCII del número 1 (49) y la última cifra del número (DL).

En los casos 2 y 3.2, sumaremos 10 a la cifra contenida en DL para obtener el número completo (solo hay números del -16 al 15, por lo que sabemos que en caso de que tenga dos cifras, la primera siempre será 1).

Después, se convierte el número de ASCII a decimal (restando 48). Si el número es negativo (indicado por el flag AL), hacemos el complemento a 2 (con NEG).
Introducimos el número decimal a la matriz decimal (MDEC) en la posición indicada por la iteración del bucle (BX). Una vez se han introducido 9 valores a la matriz, el bucle termina y la matriz decimal está completa y lista para que se calcule el determinante como en la opción 1.

Por último, se pasa el contenido de INPUT en tres partes, cada una a una línea (M1, M2 y M3) sobreescribiendo la matriz por defecto para que se impriman los valores introducidos por el usuario.
