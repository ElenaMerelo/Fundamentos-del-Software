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
	g++ ($OPT) -c $< -o $@ -I$(INC)

$(LIB)/libmates.a : $(OBJS_LIB)
	$(AR) -rvs $(LIB)/libmates.a $^

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
