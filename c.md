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

