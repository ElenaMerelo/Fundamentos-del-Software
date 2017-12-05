## Práctica 8: Compilación de programas
### Objetivos principales
+ Conocer cómo la utilidad gcc/g++ realiza las distintas etapas del proceso de generación de un archivo ejecutable a partir de distintos archivos de código fuente.
+ Conocer las dependencias que se producen entre los distintos archivos implicados en el proceso de generación de un archivo ejecutable.
+ Saber construir un archivo makefile sencillo que permita mantener las dependencias entre los distintos módulos de un pequeño proyecto software.

Además, se verán las siguientes órdenes: `gcc/g++`, `ar`, `make`.

**Ejercicio 8.1.** Pruebe a comentar en el archivo fuente main.cpp la directiva de procesamiento `#include ”functions.h”`. La línea quedaría así: `//#include ”functions.h”`. Pruebe a generar ahora el módulo objeto con la orden de compilación `g++ -c main.cpp`. ¿Qué ha ocurrido?
**Al compilarlo da el siguiente error:**
~~~
main.cpp: In function ‘int main()’:
main.cpp:7:17: error: ‘print_hello’ was not declared in this scope
     print_hello();
                 ^
main.cpp:9:52: error: ‘factorial’ was not declared in this scope
     cout << "The factorial of 7 is " << factorial(7) << endl;
~~~
**Esto es ya que dichas funciones, `print_hello()` y `factorial(n)` estaban incluidas en el fichero functions.h, y cuando se enlaza no se encuentran.**

**Ejercicio 8.2.** Explique por qué el enlazador no ha podido generar el programa archivo ejecutable programa2 del ejemplo anterior y, sin embargo, ¿por qué sí hemos podido generar el módulo main2.o?
*Ejemplo anterior:*
~~~
g++ -o programa2 main2.o factorial.o hello.o
main2.o: In function 'main':
main2.cpp:(.text+0x58): undefined reference to 'print_sin(float)'
main2.cpp:(.text+0x65): undefined reference to 'print_cos(float)'
main2.cpp:(.text+0x72): undefined reference to 'print_tan(float)'
collect2: ld returned 1 exit status
~~~
**Porque no hemos incluido los objetos .o de las funciones del seno, coseno y tangente, los cuales también se pueden incluir mediante una biblioteca. No ocurre esto con el main.o ya que ésta llama a las demás funciones.**

**Ejercicio 8.3.** Explique por qué la orden g++ previa ha fallado. Explique los tipos de errores que ha encontrado.
**Ejecutamos lo siguiente:**
~~~
$ mkdir includes

$ mv *.h includes/

$ rm *.o programa2

$ g++ -L ./ -o programa2 main2.cpp factorial.cpp hello.cpp -lmates
main2.cpp:2:23: fatal error: functions.h: No existe el archivo o el directorio
compilation terminated.
factorial.cpp:1:23: fatal error: functions.h: No existe el archivo o el directorio
compilation terminated.
hello.cpp:2:23: fatal error: functions.h: No existe el archivo o el directorio
compilation terminated.
~~~
**Falla porque hemos movido las librerías a otra carpeta, y la opción -L./ especifica que debe buscar las bibliotecas en la carpeta actual. Al no estar ahí no encuentra la definición de ninguna función.**

**Ejercicio extra** Busque en internet el problema que se puede dar en las reglas virtuales cuando se crea un archivo en el directorio con el mismo nombre de la regla (por ejemplo, clean). ¿Cómo se soluciona?
**Si existiese un fichero que se llamase “clean” dentro del directorio del Makefile, make consideraría que ese objetivo ya está realizado, y no ejecutaría los comandos asociados:**
~~~
   $ touch clean
   $ make clean
   make: `clean' está actualizado.
~~~
**Ejercicio 8.4.** Copie el contenido del makefile previo a un archivo llamado makefileE ubicado en el mismo directorio en el que están los archivos de código fuente .cpp. Pruebe a modificar distintos archivos .cpp (puede hacerlo usando la orden touch sobre uno o varios de esos archivos) y compruebe la secuencia de instrucciones que se muestra en el terminal al ejecutarse la orden make. ¿Se genera siempre la misma secuencia de órdenes cuando los archivos han sido modificados que cuando no? ¿A qué cree puede deberse tal comportamiento? **No se genera siempre la misma secuencia de órdenes cuandos los archivos han sido modificados ya que se recompilarán dichos archivos. Al hacer `make` la primera vez:**
~~~
$ make -f makefileE
ar -rvs mates.a sin.o cos.o tan.o
r - sin.o
r - cos.o
r - tan.o
~~~
**Al hacer `make` una segunda vez devuelve lo mismo. Si modificamos alguno de los archivos, por ejemplo hello.cpp:**
~~~
$ cat hello.cpp
#include <iostream>
#include "functions.h"

using namespace std;

void print_hello(){
   cout << "Hello World!";
}
$ atom hello.cpp #Modifico el archivo
$ cat hello.cpp
#include <iostream>
#include "functions.h"

using namespace std;

void print_hello(){
   cout << "Hello dlksafjasfls World!";
}
$make -f makefileE
g++ -I./includes -c hello.cpp
ar -rvs mates.a sin.o cos.o tan.o
r - sin.o
r - cos.o
r - tan.o
g++ -L./ -o programa2 main2.o factorial.o hello.o libmates.a
~~~
**Como vemos solo ha sido necesaria la obtención del archivo objeto hello.o y el posterior enlazado dado que los otros dos archivos objeto ya existían y no han sido modificados sus archivos fuente.**

**Ejercicio 8.6.** Usando como base el archivo makefileG, sustituya la línea de orden de la regla cuyo objetivo es programa2 por otra en la que se use alguna de las variables especiales y cuya ejecución sea equivalente.
~~~
# Nombre archivo: makefile
# Uso: make
# Descripción: Mantiene todas las dependencias entre los módulos y la biblioteca que utiliza el programa2.
# $@ se refiere a lo que hay antes de los dos puntos
# $< se refiere a lo primero que hay después de los dos puntos
# $^ se refiere a todo lo que hay después de los dos puntos

LIB_DIR=./
OBJS=main2.o factorial.o hello.o
SRCS=$(OBJS:.o=.cpp)
INC=./includes
OPT=-Wall -g


OBJS_LIB=sin.o cos.o tan.o

all : programa2

programa2 : $(OBJS) $(LIB_DIR)/libmates.a
	g++ -o $@ $(OBJS) -lmates -L$(LIB_DIR)

programa2.o : $(SRCS)
	g++ $(OPT) -c $^ -o $(OBJS) -I$(INC)

main2.o : main2.cpp
	g++ ($OPT) -c $< -o $@ -I$(INC)

factorial.o : factorial.cpp
	g++ ($OPT) -c $< -o $@ -I$(INC)

hello.o : hello.cpp
	g++ -c $< -I$(INC)

libmates.a : $(OBJS_LIB)
	ar -rvs libmates.a $^

sin.o : sin.cpp
	g++ -c $< -I$(INC)

cos.o : cos.cpp
	g++ -c $< -I$(INC)

tan.o : tan.cpp
	g++ -c $< -I$(INC)

cleanAll : cleanObj cleanLib
	rm programa2

cleanObj :
	rm $(OBJS)

cleanLib :
	rm $(OBJS_LIB) libmates.a
~~~
**Probamos su funcionamiento:**
~~~
$ make
ar -rvs libmates.a sin.o cos.o tan.o
r - sin.o
r - cos.o
r - tan.o
g++ -o programa2 main2.o factorial.o hello.o -lmates -L./
~~~
**Ejercicio 8.7.** Utilizando como base el archivo makefileG y los archivos fuente asociados, realice los cambios que considere oportunos para que, en la construcción de la biblioteca estática libmates.a, este archivo pase a estar en un subdirectorio denominado libs y se pueda enlazar correctamente con el resto de archivos objeto.
**Primeramente creamos el directorio libs(`mkdir libs`), y ya dentro del makefile creamos la variable LIB=./libs. Ahoro donde antes ponía libmates.a hemos de poner $(LIB)/libmates.a, lo demás se mantiene igual:**
~~~
# Nombre archivo: makefile
# Uso: make
# Descripción: Mantiene todas las dependencias entre los módulos y la biblioteca
#              que utiliza el programa2.
# $@ se refiere a lo que hay antes de los dos puntos
# $< se refiere a lo primero que hay después de los dos puntos
# $^ se refiere a todo lo que hay después de los dos puntos
# Author: Elena Merelo Molina


LIB=./libs
OBJS=main2.o factorial.o hello.o
SRCS=$(OBJS:.o=.cpp)
INC=./includes
OPT=-Wall -g


OBJS_LIB=sin.o cos.o tan.o

all : programa2

programa2 : $(OBJS) $(LIB)/libmates.a
	g++ -o $@ $(OBJS) -lmates -L$(LIB)

programa2.o : $(SRCS)
	g++ $(OPT) -c $^ -o $(OBJS) -I$(INC)

main2.o : main2.cpp
	g++ ($OPT) -c $< -o $@ -I$(INC)

factorial.o : factorial.cpp
	g++ ($OPT) -c $< -o $@ -I$(INC)

hello.o : hello.cpp
	g++ -c $< -I$(INC)

$(LIB)/libmates.a : $(OBJS_LIB)
	ar -rvs $(LIB)/libmates.a $^

sin.o : sin.cpp
	g++ -c $< -I$(INC)

cos.o : cos.cpp
	g++ -c $< -I$(INC)

tan.o : tan.cpp
	g++ -c $< -I$(INC)

cleanAll : cleanObj cleanLib
	rm programa2

cleanObj :
	rm $(OBJS)

cleanLib :
	rm $(OBJS_LIB) libmates.a
~~~
**Vemos que funciona ya que al escribir en la terminal `make` devuelve:**
~~~
ar -rvs ./libs/libmates.a sin.o cos.o tan.o
ar: creando ./libs/libmates.a
a - sin.o
a - cos.o
a - tan.o
g++ -o programa2 main2.o factorial.o hello.o -lmates -L./libs
~~~

**Ejercicio 8.8.** Busque la variable predefinida de make que almacena la utilidad del sistema que permite construir bibliotecas. Recuerde que la orden para construir una biblioteca estática a partir de una serie de archivos objeto es ar (puede usar la orden grep para filtrar el contenido; no vaya a leer línea a línea toda la salida). Usando el archivo makefileG, sustituya la orden ar por su variable correspondiente.
**Ejecuto en la terminal `make -p|grep "ar"` y encuentro que hay una variable predefinida para definir bibliotecas: `AR = ar`. Así, en el makefile anterior solo hay que cambiar la parte correspondiente a la creación de la librería:**  
~~~
$(LIB)/libmates.a : $(OBJS_LIB)
	$(AR) -rvs $(LIB)/libmates.a $^
~~~
**Ejercicio 8.9.** Dado el siguiente archivo makefile, explique las dependencias que existen y para qué sirve cada una de las líneas del mismo. Enumere las órdenes que se van a ejecutar a consecuencia de invocar la utilidad make sobre este archivo.
~~~
# Nombre archivo: makefileH
# Uso: make -f makefileH
# Descripción: Mantiene todas las dependencias entre los módulos que utiliza el
# programa1.
CC=g++  #Decimos así el compilador que se va a usar
CPPFLAGS=-Wall –I./includes #Para que muestre los warnings
SRCS=main.cpp factorial.cpp hello.cpp #Lista de dependencias
OBJS=$(SRCS:.cpp=.o) #Pasamos los .cpp a .o y los guardamos en una variable
EXE=programa1 #Nombre del ejecutable

all: $(OBJECT_MODULES) $(EXECUTABLE)  #Compila el programa entero

$(EXECUTABLE): $(OBJECT_MODULES)    #Definimos las dependencias del ejecutable  
  $(CC) $^ -o $@                    #Compilamos y creamos los archivos objeto

# Regla para obtener los archivos objeto .o que dependerán de los archivos .cpp
# Aquí, $< y $@ tomarán valores respectivamente main.cpp y main.o y así sucesivamente
.o: .cpp
  $(CC) $(CPPFLAGS) $< -o $@
~~~
**Ejercicio 8.10.** Con la siguiente especificación de módulos escriba un archivo denominado makefile3 que automatice el proceso de compilación del programa final de acuerdo a la siguiente descripción:
Compilador: gcc o g++
Archivos cabecera: calc.h (ubicado en un subdirectorio denominado cabeceras)
Archivos fuente: main.c stack.c getop.c getch.c
Nombre del programa ejecutable: calculadoraPolaca
Además, debe incluir una regla denominada borrar, sin dependencias, cuya funcionalidad sea la de eliminar los archivos objeto y el programa ejecutable.
~~~
# Nombre del archivo: makefile3
# Uso: make -f makefile3
# Autora: Elena Merelo Molina

CC=g++
INC=./cabeceras
SRCS=main.c stack.c getop.c getch.c
OBJS=$(SRCS:.c=.o)
EXE=calculadoraPolaca
CPPFLAGS=-Wall -I$(INC)

all : $(EXE)

$(EXE) : $(OBJS)
	$(CC) $^ -o $@

.o : .c
	$(CC) $(CPPFLAGS) $< -o $@

clean:
	rm *.o $(EXE)
~~~
**Ejercicio extra** Con la siguiente especificación de módulos escriba un archivo makefile que automatice el proceso de compilación del programa final.
El archivo programa.cpp usa funciones de ordenación de los elementos de un array incluidas en el archivo ordenacion.cpp, en ordenacion.h, funciones de manejo de arrays incluidas en array.cpp, cuyo archivo de cabecera es array.h, y también utiliza funciones para imprimir los arrays, incluidas en impresion.cpp.

El archivo ordenacion.cpp utiliza funciones de manejo de arrays incluidas en el archivo array.cpp, cuya declaración se encuentra en array.h.
El archivo impresion.cpp se encarga de proporcionar utilidades de impresión generales para distintos tipos de datos, entre ellos la impresión de arrays. Las declaraciones de sus funciones se encuentran en utilities.h.
~~~
# Nombre de archivo: makefile4
# En include están ordenacion.h, array.h y utilities.h
# En src están los SRC_MODULES, en obk los OBJ_MODULES
# Uso: make -f makefile4
# Autora: Elena Merelo Molina

CC=g++
INC=./include
OBJ=./obj
SRC=./src
BIN=./bin
CPPFLAGS= -Wall -g
SRC_MODULES=programa.cpp ordenacion.cpp array.cpp impresion.cpp
OBJ_MODULES=$(SRCS:.cpp=.o)

all : $(BIN)/programa

$(BIN)/programa : $(OBJ)/$(OBJ_MODULES)
	$(CC) $^ -o $@ -I$(INC)

$(OBJ)/ordenacion.o : $(SRC)/ordenacion.cpp $(SRC)/array.cpp
	$(CC) $(CPPFLAGS) -o $@  -c $^ -I$(INC)

$(OBJ)/array.o : $(SRC)/array.cpp
	$(CC) $(CPPFLAGS) -o $@ -c $< -I$(INC)

$(OBJ)/impresion.o : $(SRC)/impresion.cpp
	$(CC) $(CPPFLAGS) -o $@ -c $< -I$(INC)

clean:
  rm *.o $(BIN)/programa
~~~
