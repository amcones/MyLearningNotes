CXX = g++-12
objects = main.o

example1 : $(objects)
	$(CXX) -o example1 $(objects)

main.o : fib.h

.PHONY : clean
clean :
	rm example1 $(objects)
