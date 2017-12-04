## Práctica 8: Compilación de programas
### Objetivos principales
+ Conocer cómo la utilidad gcc/g++ realiza las distintas etapas del proceso de generación de un archivo ejecutable a partir de distintos archivos de código fuente.
+ Conocer las dependencias que se producen entre los distintos archivos implicados en el proceso de generación de un archivo ejecutable.
+ Saber construir un archivo makefile sencillo que permita mantener las dependencias entre los distintos módulos de un pequeño proyecto software.

Además, se verán las siguientes órdenes: `gcc/g++`, `ar`, `make`.

**Ejercicio 8.1.** Pruebe a comentar en el archivo fuente main.cpp la directiva de procesamiento `#include ”functions.h”`. La línea quedaría así: `//#include ”functions.h”`. Pruebe a generar ahora el módulo objeto con la orden de compilación `g++ -c main.cpp`. ¿Qué ha ocurrido?
**Al compilarlo da el siguiente error: **
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
*Ejemplo anterior: *
~~~
g++ -o programa2 main2.o factorial.o hello.o
main2.o: In function 'main':
main2.cpp:(.text+0x58): undefined reference to 'print_sin(float)'
main2.cpp:(.text+0x65): undefined reference to 'print_cos(float)'
main2.cpp:(.text+0x72): undefined reference to 'print_tan(float)'
collect2: ld returned 1 exit status
~~~
**Porque no hemos incluido los objetos .o de las funciones del seno, coseno y tangente, los cuales también se pueden incluir mediante una biblioteca. No ocurre esto con el main.o ya que ésta llama a las demás funciones.**

** Ejercicio 8.3.** Explique por qué la orden g++ previa ha fallado. Explique los tipos de errores que ha encontrado.
**Ejecutamos lo siguiente: **
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
** Falla porque hemos movido las librerías a otra carpeta, y la opción -L./ especifica que debe buscar las bibliotecas en la carpeta actual. Al no estar ahí no encuentra la definición de ninguna función.**

** Ejercicio extra** Busque en internet el problema que se puede dar en las reglas virtuales cuando se crea un archivo en el directorio con el mismo nombre de la regla (por ejemplo, clean). ¿Cómo se soluciona?
** Si existiese un fichero que se llamase “clean” dentro del directorio del Makefile, make consideraría que ese objetivo ya está realizado, y no ejecutaría los comandos asociados: **
~~~
   $ touch clean
   $ make clean
   make: `clean' está actualizado.
~~~
** Ejercicio 8.4.** Copie el contenido del makefile previo a un archivo llamado makefileE ubicado en el mismo directorio en el que están los archivos de código fuente .cpp. Pruebe a modificar distintos archivos .cpp (puede hacerlo usando la orden touch sobre uno o varios de esos archivos) y compruebe la secuencia de instrucciones que se muestra en el terminal al ejecutarse la orden make. ¿Se genera siempre la misma secuencia de órdenes cuando los archivos han sido modificados que cuando no? ¿A qué cree puede deberse tal comportamiento? ** No se genera siempre la misma secuencia de órdenes cuandos los archivos han sido modificados ya que se recompilarán dichos archivos. Al hacer make la primera vez: 
~~~
$ make -f makefileE
ar -rvs mates.a sin.o cos.o tan.o
r - sin.o
r - cos.o
r - tan.o
~~~
Al hace make una segunda vez devuelve lo mismo. Si modificamos alguno de los archivos, por ejemplo hello.cpp: 
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
Como vemos solo ha sido necesaria la obtención del archivo objeto hello.o y el posterior enlazado dado que los otros dos archivos objeto ya existían y no han sido modificados sus archivos fuente. **

** Ejercicio 8.6.** Usando como base el archivo makefileG, sustituya la línea de orden de la regla cuyo objetivo es programa2 por otra en la que se use alguna de las variables especiales y cuya ejecución sea equivalente.
~~~
# Nombre archivo: makefile
# Uso: make 
# Descripción: Mantiene todas las dependencias entre los módulos y la biblioteca que utiliza el programa2.
# $@ se refiere a lo que hay antes de los dos puntos
# $< se refiere a lo primero que hay después de los dos puntos
# $^ se refiere a todo lo que hay después de los dos puntos
# Author: Elena Merelo Molina

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
**Probamos su funcionamiento: **
~~~
$ make
ar -rvs libmates.a sin.o cos.o tan.o
r - sin.o
r - cos.o
r - tan.o
g++ -o programa2 main2.o factorial.o hello.o -lmates -L./
~~~














