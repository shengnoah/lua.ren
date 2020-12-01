---
layout: post
title: reading note of programming in lua 4th edition i 
tags: [lua文章]
categories: [topic]
---
**Content**

* * *

Here is the summary of reading note about some Lua basic grammar and data
structures.

### Basic

#### Stand-Alone Interpreter

  * Useful Lua idiom
    * `x = x or v` is equivalent to `if not x then x = v end`.
    * `((a and b)or c) is equivalent to C expression`a?b:c`.

#### Arithmetic Operators

  * Division
    * float division is `/`, which is the same as other programming lanuage.
    * integer division is `//`, which is called “floor division”.

#### Strings

  * Strings in Lua are immutable values.
  * Strings in Lua are subject to automatic memory management, like all other Lua objects (table, functions, etc).
  * Use `..` to concatenate two strings.

  * Any numeric operation applied to a string tries to convert the string to a number. This coersion is also applied in other places that expect a number, such as the argument to `math.sin`.

* * *

### Tables

#### Safe Navigation

To know whether a given function from a given library is present, use this
statement: `res = lib?.object1?.object2?.function` or `res = (((lib or
{}).object1 or {}).object2 or {}).function`.

#### Libraries

  * table.insert: insert an element in a given position of a sequence.
  * table.remove: removes and returns an element from the given position in a sequence.
  * table.move: moves the elements in table a from index f until e (both inclusive) to position t or another table.
  * table.pack: receive any number of arguments and returns a new table with all its arguments (just like {…}).
  * table.unpack: transform a real Lua list (a table) into a return list, which can be given as the parameter list to another function.

* * *

### Functions

#### Generic

  * Return multiple results from a function: `return res1, res2, res3`. And a statement like `return (f(x))` always returns one single value.
  * Variadic function, taking a variable number of arguments, uses three dots(…) in the parameter list:

    
    
    1  
    2  
    3  
    

|

    
    
    function func(...)  
       do sth  
    end  
      
  
---|---  
  
#### Tail Calls

  * Lua does tail-call elimination. So in following code:

    
    
    1  
    2  
    3  
    4  
    

|

    
    
    function f(x)  
    	x = x + 1  
    	return g(x)  
    end  
      
  
---|---  
  
When g returns, control will return directly to the point calling f, and thus,
do not use any extra stack space when doing a tail call.

* * *

### IO

  * io.read: read strings from current input stream. The input parameter includes `"a", "l", "L", "n" and number`.
  * io.write: write strings to current output stream. Use `string.format` for full control over the numbers to strings conversion.
  * io.open: open a file.
  * io.flush: flush the current output stream.
  * io.popen: runs a system command and connects the command output (or input) to a new local stream and returns that stream.
  * os.exit: terminates the execution of a program.
  * os.getenv: gets the value of an environment variable. ex: `os.getenv("HOME")`.
  * os.execute: runs a system command.

* * *

### Variable and Control Structure

  * Lua treats all values (including 0 and empty string) as true except false and nil.

[Share](javascript:void\(0\))