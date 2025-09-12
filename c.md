Notes on C
=================================
You can seperate your program into modules like mod1.c, mod2.c, mod3.c, main.c

Communication between modules
---------------------------------
Function inside module A can call the function inside another module B. But a prototype declaration must be made in module A to let the compiler know the signature of the function when compiling module A.
The point is even though more than one module can be specified to the compiler at the same time the compiler compiles each module independently.
That means no knowledge about structure definitions, function return types or function argument types is shared across module compilations. 

Functions in seperate files can communicate through external variables. 
External variable values can be accessed and changed by another module.
First define a global variable inside a module. Inside the module that wants to access the external variable the variable data type is preceded with the keyword extern.
If you want to define a global variable and make it local to a particular module use keyword static. 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ C
int moveVal = 0; // global variable definition

void foo()
{
	...
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
mod1.c

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ C
extern int moveVal; // external variable declaration for a global variable in mod1.c

void bar()
{
	moveVal = 5;
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
mod2.c

Memory
-------------------------------------
There are three forms of memory you can use in your program. Each type determines where and how it is stored.
	
	- static - persists throughout the entire life of the program
	- stack 
		- special region of memory that stores temporary variables (variables created inside a function)
		- LIFO (last in first out) linear data structure
		- variables allocated/freed automatically
		- divided into stack frames for each function call, when a function exits all its variables are popped off
	- heap  
		- hierarchical data structure
		- large pool of memory that can be used dynamically
		- memory is not automatically managed 
		- accessed through pointers (malloc/free)
		- no restrictions on the size of the heap - only limit is physical memory

storage classes
-------------------------------------
C storage classes 
	* auto - is used to declare variables of automatic storage duration
		- created when the block in which they are defined is entered
		- exists while the block is active
		- destroyed when the block is exited
		- local variables have automatic storage duration by default
		- also note that auto is completely different meaning in C++, must be used with caution for the sake of C/C++ compatibity
	* register
		- this storage class is used to define local variables that should be stored in a register instead of RAM
		- can be used for heavily used variables which need quick access
		- but it is the compiler's choice to put the variable in a register or not so depends on hardware and implementation restrictions
		- generally compilers themselves do optimizations and put the variables in registers
		- maximum size is equal to the register size
		- you cannot obtain the address of a register variable using pointers and cannot apply the & operator to it. It does not have a memory location
		- life-time is limited to the block it is defined in, same as auto
		- can only be used in local block, cannot be used globally
	* extern - tells us that a variable is defined elsewhere, not within the same block where it is used
		- The first way to use extern variable is declare a global variable inside a module - not preceded by the word extern, then use the variable in other modules by declaring it with keyword extern
		- second way is declaring the variable outside any function with the extern keyword and assigning an initial value to it
		- functions can be declared extern and this way function can be used inside a module without an include file
		- function declarations are extern by default the other alternative is static 
	* static
		- when applied to local variables it instructs the compiler to keep the variable in existence during the life-time of the program
		- when applied to global variables static modifier causes that variable's scope to be restricted to the file in which it is declared
		- when applied to functions, static function can only be called from within the same file as the function appears
		- static variables preserve their last value and no new memory is allocated as they are not re-declared
		- making local variables static allows them to maintain their values between function calls
		- Note that these two local static variables are different

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ C
int foo1()
{
	static int count = 100; // initial value at program execution start, then it can change during execution
}

int foo2()
{
	static int count2; 
	count2 = 100;	// variable will be initialized each time this function is called 
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		
		- static variables should not be declared inside a structure because memory allocation for structure members should be contiguous. Whatever might be the case all structure members should reside in the same memory segment. The value of the structure element is fetched by counting the offset of the element from the beginning address of the structure.
		
		(#) Summary of storage classes
		
		Storage Class |       Declaration  	    |          Scope       	   | 	   Lifetime
					  |         Location	    |       (Visibility)	   |    	(Alive)
		--------------|-------------------------|--------------------------|--------------------------
		auto     	  | Inside a function/block |        Within the   	   | Until the function/block
					  |            				|      function/block  	   |	completes
		--------------|-------------------------|--------------------------|--------------------------
		register  	  | Inside a function/block |        Within the    	   | Until the function/block
					  |            				|      function/block  	   |	completes			
		--------------|-------------------------|--------------------------|--------------------------
		extern  	  | Outside all functions   | Entire file plus other   | Until the program
					  |            				| files where the variable |	terminates				
					  |            				| is declared as extern    |										  
		--------------|-------------------------|--------------------------|--------------------------
		static  	  | Inside a function/block |        Within the    	   | Until the program
		(local)		  |            				|      function/block  	   |	terminates			
		--------------|-------------------------|--------------------------|--------------------------
		static  	  | Outside all functions 	|   Entire file in which   | Until the program
		(global)	  |            				|      it is declared      |	terminates								  
		
typedef
--------------------------------
typedef can be used for casting.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ C
typedef int(*ptr_to_int_fun)(void);
char *p;
...= (ptr_to_int_fun) p;
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

variable length arrays
----------------------------------------
For C99 and C11 the following declaration is valid
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ C
int sum2d(int, int, int array[*][*]); // names can be ommitted
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

and the definition can be like the following
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ C
int sum2d(int rows, int cols, int array[rows][cols])
{
	//..
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

But C11 conforming compiler does not have to implement support for variable length arrays because it is an optional feature.
Check for support for variable length arrays
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ C
#ifdef __STDC_NO_VLA__
	printf("Variable length arrays are not supported!\n");
	exit(1);
#endif
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

flexible member arrays
-------------------------------
This is a feature introduced in C99 and there are some restrictions on the usage of this.
Flexible array has to be the last member of the struct and it also cannot be the only member of the struct
Each struct can contain one flexible array at most
The struct has to be allocated dynamically, cannot be statically initialized (size cannot be fixed at compile time)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ C
#include <stdio.h>
#include <malloc.h>

struct s {
	int arraysize;
	int array[];
};

int desiredSize = 10;
struct s *ptr;
ptr = malloc(sizeof(struct s) + desiredSize * sizeof(int));
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~