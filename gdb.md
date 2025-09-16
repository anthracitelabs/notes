How to debug using gdb 
=============================

Remember, when you use make you must use the -g flag for gdb to perform correctly.
Start by typing gdb command with application name.
If your program runs with any command line arguments, you should input them with "set args". For example, if you would normally run your program "test" with the command line "test -paramater1 -parameter2" you would type "set args -parameter1 -parameter2".
To execute one line of code, type "step" or "s". If the line to be executed is a function call, gdb will step into that function and start executing its code one line at a time. If you want to execute the entire function with one keypress, type "next" or "n". This is equivalent to the "step over" command of most debuggers.
If you want gdb to resume normal execution, type "continue" or "c". gdb will run until your program ends, your program crashes, or gdb encounters a breakpoint.

Since all of gdb is all in one window, when you are debugging you cannot see the source code for your program. To view the source code, type "list" or "l". gdb will print out the source code for the lines around the current line to be executed. To view other lines, just type "list [linenumber]", and gdb will print out the 20 or so lines around that line. gdb remembers what lines you have seen, so if you type "list" again it will print out the next bunch of lines.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
gdb <application-name>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To set break point inside a file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
(gdb) break [filename]:[linenumber]
(gdb) run
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can also set breakpoints on function names. To do this, just type "break [functionname]". gdb will stop your program just before that function is called. Breakpoints stay set when your program ends, so you do not have to reset them unless you quit gdb and restart it.

Working with breakpoints
	- To list current breakpoints: "info break"
	- To delete a breakpoint: "del [breakpointnumber]"
	- To temporarily disable a breakpoint: "dis [breakpointnumber]"
	- To enable a breakpoint: "en [breakpointnumber]"
	- To ignore a breakpoint until it has been crossed x times:"ignore [breakpointnumber] [x]"
	
 To examine a variable, type "print [variablename]". To set the value of a variable, type "set [variablename]=[valuetoset]".
 
 To print stack trace type "bt"

Step out
----------
finish: Continue running until just after function in the selected stack frame returns. Print the returned value (if any). This command can be abbreviated as fin.
