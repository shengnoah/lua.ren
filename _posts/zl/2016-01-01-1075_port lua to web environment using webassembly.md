---
layout: post
title: port lua to web environment using webassembly 
tags: [lua文章]
categories: [topic]
---
> WebAssembly (abbreviated Wasm) is a binary instruction format for a stack-
> based virtual machine. Wasm is designed as a portable target for compilation
> of high-level languages like C/C++/Rust, enabling deployment on the web for
> client and server applications.

Recently, all major browsers like Google Chrome, Safari, Microsoft Edge all
support web assembly. Web assembly allows developers to compile C/C++
application into a browser supported format, meaning even a 3A game can be run
on the browser platform as well.

Not only the portability, Web assembly also brings great performance
improvement. It uses LLVM compiler to emit the assembly code. LLVM handles the
C/C++ static analysis and optimization. Moreover, web assembly can be just-in-
time compiled into machine code to achieve much higher performance.

For example, here is a Lua code interpreted by Lua virtual machine (5.3.5)
that directly compiled from C to web assembly.

    
    
    1
    2
    3
    4
    5
    6
    7
    8
    

|

    
    
    function hanoi(n, A, B, C)
        if n == 1 then print(A..' ---> '..C) return end
        hanoi(n-1, A, C, B)
        hanoi(1, A, B, C)
        hanoi(n-1, B, A, C)
    end
    
    hanoi(3, 'A', 'B', 'C')  
  
---|---  
  
Click RUN to see the result.

    
    
        run me, >  < ~~~
    

Is this cool? This is a full-featured Lua virtual machine running in the
browser environment. You can use all Lua build-in libraries. It is much faster
than any other Lua JS implementation as well.

This article will demonstrate how to build a Lua web assembly target and
inject it into the browser environment.

Web assembly uses Clang and LLVM as the compiler infrastructure. The default
Clang and LLVM are not compatible with web assembly, you will need to build
them from source. `binaryen` is used to generate the final output from the
assembly code. This is super tedious and the LLVM default does not support C
standard library. You will need to remap the `printf` or the `fopen` function
by yourself.

Fortunately, emscripten has wrapped all these tedious steps into a standalone
package. We will use emscripten to compile Lua in this case.

First of all, we will need to obtain the emscripten package. It is super easy
to do on macOS.

    
    
    1
    

|

    
    
    brew install emscripten  
  
---|---  
  
Then follow the instruction to set up `emcc`.

Since we are going to host Lua in the browser environment. The default Lua
host needs to be changed. Instead of executing a Lua file, we expose a
function that allows Lua to execute the given script.

Modify `lua.c` code to:

    
    
     1
     2
     3
     4
     5
     6
     7
     8
     9
    10
    11
    12
    13
    14
    15
    16
    17
    18
    19
    20
    21
    22
    23
    24
    25
    26
    27
    28
    29
    

|

    
    
    #include <stdlib.h>
    #include <stdio.h>
    
    #include "lua.h"
    #include "lauxlib.h"
    #include "lualib.h"
    
    int lua_main(const char *script)
    {
      int status, result;
      lua_State *L = luaL_newstate(); /* create state */
      if (L == NULL)
      {
        printf("lua: cannot create state: not enough memoryn");
        return 1;
      }
      luaL_openlibs(L);
      status = luaL_dostring(L, script);
      if (status != LUA_OK)
      {
        const char *msg = lua_tostring(L, -1);
        printf("lua: %sn", msg);
        lua_close(L);
        return EXIT_FAILURE;
      }
      result = lua_toboolean(L, -1);
      lua_close(L);
      return result ? EXIT_SUCCESS : EXIT_FAILURE;
    }  
  
---|---  
  
Note that `luaL_dostring` is the Lua function that executes Lua code in
protected mode. If an error occurs in the protected call, the error message
will be pushed into the top of Lua stack.

Secondly, we need to modify the `Makefile`:

    
    
    1
    2
    3
    4
    5
    6
    

|

    
    
    # change LUA_T=	lua to
    LUA_T=	lua.js
    
    $(LUA_T): $(LUA_O) $(LUA_A)
        # change $(CC) -o [[email protected]](/cdn-cgi/l/email-protection) $(LDFLAGS) $(LUA_O) $(LUA_A) $(LIBS) to
        $(CC) -o [[email protected]](/cdn-cgi/l/email-protection) $(LDFLAGS) $(LUA_O) $(LUA_A) $(LIBS) -s EXPORTED_FUNCTIONS="['_lua_main']" -s EXTRA_EXPORTED_RUNTIME_METHODS="['cwrap']" -s ALLOW_MEMORY_GROWTH=1
      
  
---|---  
  
`EXPORTED_FUNCTIONS` is to export the function to the JS environment. Later we
will create a JS wrap function to wrap the `lua_main` function. Note that,
LLVM will be appended a `_` for each function created. So it should be
`_lua_main` instead of `lua_main`.

Finally, we can build the Lua binary by calling:

    
    
    1
    

|

    
    
    make generic CC='emcc -s WASM=1' AR='emar rcu' RANLIB='emranlib'  
  
---|---  
  
It will generate `lua.js` and `lua.wasm`. We will use these two files in the
browser environment.

Now we create an `index.html` file:

    
    
     1
     2
     3
     4
     5
     6
     7
     8
     9
    10
    11
    12
    13
    14
    15
    16
    17
    

|

    
    
    <script>
    var Module = {
        print: (text) => {
            alert("stdout: " + text);
        },
    
        printErr: (text) => {
            alert("stderr: " + text);
        },
    
        onRuntimeInitialized: () => {
            const lua_main = Module.cwrap('lua_main', 'number', ['string']);
            lua_main("print('hello world'");
        }
    };
    </script>
    <script src="./lua.js">  
  
---|---  
  
First of all, we create a Module object. This object will be reused in the
`lua.js` file as well. So, the `print` and `printErr` functions are used to
redirect the C `stdout` and `stderr` to the browser environment.

The `onRuntimeInitialized` is the callback to be invoked when `lua.wasm` is
fully initialized. Here we wrap the lua_main function to a JS function
`lua_main` by using

    
    
    1
    

|

    
    
    const lua_main = Module.cwrap('lua_main', 'number', ['string']);
      
  
---|---  
  
When we call `lua_main("print('hello world'");`, an alert, `hello world`, will
be shown to the user.

Of cause, the stdout can be redirected to an HTML element as well. Here is a
simple Lua playground

    
    
    function playground()
        print('Hello World')
    end
    
    playground()
    
    

Click RUN to see the result.

    
    
        run me, >  < ~~~
    

Have fun :)