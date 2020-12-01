---
layout: post
title: interfacing lua with c 
tags: [lua文章]
categories: [topic]
---
Lua is a lightweight, multi-paradigm programming language designed primarily
for embedded use in applications. Roberto Ierusalimschy, the author of Lua,
said that Lua is like passing a language through the eye of a needle.

Lua is amazingly small - it only has around 10000 lines of code, and it can be
embedded into any systems. It is written pure C language and any C compiler is
able to compile Lua without any issues (no dependency problems, no build tools
problems, and even on Windows platform, it is still extremely easy to build).
Actually, Lua is distributed by source code only. People are able to drag and
drop the whole Lua source code into their project and start using Lua
immediately.

There are some design highlights in Lua source code.

First of all, the Lua Stack. Lua provides an interface allowing to call C
functions in Lua environment. Lua uses a virtual stack for passing values
between Lua and C. The Lua function pushes the parameters into the stack, then
C function consumes the values and pushes the result values to stack. Finally,
C function returns an integer to indicate the number of values that returned.

Second, the Lua Metatable. Lua is a multi-paradigm programming language,
achieving by using a special data structure, table. Hashmap, array and even
Object-Oriented programming are all implemented by the table. Moreover, a
special metatable is used for high-level language features, such as
inheritance, abstraction and method overloading.

We will use the [checkerboard rendering](/posts/checkerboard-rendering/) code
to showcases these two features. In this example, we will read png data from a
file, and return as a png object. The png object provides two APIs, `set` and
`at` to access and modify the internal colors. Finally, png writes back data
to disk and dispose itself from the memory.

To interface Lua with C, we must register C function to the Lua environment
first. There are three ways to do that: 1\. Modifying Lua source code and
inject C function to the Lua interpreter directly (Extending the default Lua
interpreter). 2\. Including Lua source code into the project and host Lua code
from the main project C code (Hosting Lua as an extension script for plugins
or add-ons). 3\. Building a shared library and let Lua default interpreter
register C function at runtime (Extending Lua without modifying the default
environment, usually Lua is the main language and C is the helper functions in
this case).

The example is demonstrated in the third way.

First of all, let’s create a C file, `lua_lodepng.c`, as our entry point of
the library.

    
    
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
    

|

    
    
    // These three files must be included to access Lua functions
    
    #include <lua/lauxlib.h>
    #include <lua/lualib.h>
    #include <lua/lua.h>
    
    #include <stdio.h>
    
    static int decode_file(lua_State *L)
    {
        printf("call decode_file from Cn");
        return 0;
    }
    
    static const luaL_Reg lodepng[] = {
        {"decode_file", decode_file},
        {NULL, NULL},
    };
    
    LUAMOD_API int luaopen_lodepng(lua_State *L)
    {
        luaL_newlib(L, lodepng);
        return 1;
    }  
  
---|---  
  
Our native C function will be provided as a Lua module named as `lodepng`. In
this case, the shared library must be named as `lodepng.so`. Lua looks for
`lodepng.so` when you call `requre "lodepng"` in the current directory and
search paths. Moreover, `LUAMOD_API int luaopen_XXX(lua_State *L)` is the
entry point of the shared library and must be named as `luaopen_XXX` where the
`XXX` is replaced by the library name, `"lodepng"`, in this case.

Second, the registry table, `lodepng`. For each element in the register table,
it has a string as the key and a function pointer as the value. Basically,
when the user invokes `lodepng.decode_file` in Lua, it directs to the
`decode_file` function in C. The registry table must end with `{NULL, NULL}`.

Finally, the real C function, `static int decode_file(lua_State *L)`. So for
each C interface, it takes in a Lua state as the input, and return an integer
as the output. The Lua parameters and the returned results are passed by the
`lua_State *L`. Since Lua function supports multiple return values. The
returned integer indicates how many values are returned in this function.

Last but not least, `luaL_newlib(L, lodepng);` registers the value to the
stack, as the return value of the `lodepng` module.

To build this code, simply run:

    
    
    1
    

|

    
    
    gcc -O2 -Wall -fPIC -llua -shared -o lodepng.so lua_lodepng.c  
  
---|---  
  
Now if you run the following Lua code,

    
    
    1
    2
    

|

    
    
    local lodepng = require "lodepng"
    lodepng.decode_file("test.png")  
  
---|---  
  
You will see

    
    
    1
    

|

    
    
    call decode_file from C  
  
---|---  
  
from the output. Do remember `lodepng.so` needs to be put under the same
folder with the Lua script.

`decode_file` is the Lua interface and it supposes to have a string, filename,
as its parameter. So the C function, `decode_file` needs to access the
filename and read the data from the file. The values in the Lua stack can be
accessed by their indexes. Lua uses the positive number to indicate the values
from the bottom, or the leftmost parameters in the function, and the negative
values from the top.

![](https://i.imgur.com/JrL4bdA.png)

    
    
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
    

|

    
    
    static int decode_file(lua_State *L)
    {
        // Check the leftmost parameter, filename
        const char *filename = luaL_checkstring(L, 1);
    
        unsigned int width, height;
        unsigned char *image;
        // Load png file, https://lodev.org/lodepng/
        lodepng_decode32_file(&image, &width, &height, filename);
    
        // Return 3 values, the image pointer, width and height.
        lua_pushlightuserdata(L, image);
        lua_pushinteger(L, width);
        lua_pushinteger(L, height);
    
        // Since there are 3 return values, return 3 in this case.
        return 3;
    }  
  
---|---  
  
After recompiling this, we can access the width and height from Lua code.

    
    
    1
    2
    

|

    
    
    local lodepng = require "lodepng"
    local png, width, height = lodepng.decode_file("test.png")  
  
---|---  
  
Let’s move on to create a metatable so that we can return a png object when we
call the `decode_file` function.

    
    
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
    30
    31
    

|

    
    
    static const luaL_Reg pngmetatable[] = {
        {"at", png_at},
        {"set", png_set},
        {"encode_file", png_encode_file},
        {"dispose", png_dispose},
        {"__tostring", png_tostring},
        {NULL, NULL},
    };
    
    LUAMOD_API int luaopen_lodepng(lua_State *L)
    {
        luaL_newlib(L, lodepng);
    
        // We first need to create a metatable
        // Later we assign this metatable to the
        // returned value of decode_file
    
        // lodepngpng = {}
        luaL_newmetatable(L, "lodepngmetable");
    
        // lodepngpng.__index = lodepngpng
        lua_pushvalue(L, -1);
        lua_setfield(L, -2, "__index");
    
        // Register functions in pngmetatable
        // to the metatable
        luaL_setfuncs(L, pngmetatable, 0);
    
        lua_pop(L, 1);
        return 1;
    }  
  
---|---  
  
Again, creating metatable is very similar to create the functions in the
library. First, you create a registry table, then register code to the
metatable. So in this case, `png_at`, `png_set` are all the C function
pointers. We also need to modify the `decode_file` code to let it return a
table instead of three values.

    
    
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
    

|

    
    
    static int decode_file(lua_State *L)
    {
        const char *filename = luaL_checkstring(L, 1);
        unsigned int width, height;
        unsigned char *image;
        lodepng_decode32_file(&image, &width, &height, filename);
    
        lua_newtable(L);
        lua_pushliteral(L, "width");
        lua_pushinteger(L, width);
        lua_settable(L, -3);
        lua_pushliteral(L, "height");
        lua_pushinteger(L, height);
        lua_settable(L, -3);
        lua_pushliteral(L, "data");
        lua_pushlightuserdata(L, image);
        lua_settable(L, -3);
        luaL_setmetatable(L, "lodepngmetable");
        return 1;
    }  
  
---|---  
  
The last line, `luaL_setmetatable(L, "lodepngmetable");` assign the metable
`lodepngmetable` to the returned table.

After we implemented all functions in the `pngmetatable` registry, we shall
able to call like this:

    
    
    1
    2
    3
    4
    5
    

|

    
    
    png = lodepng.decode_file("test_sample.png")
    print(png)
    
    print(png:at(0, 0))
    png:set(0, 0, png:at(1, 0))  
  
---|---  
  
The `:` opreator will pass the object as the first parameter into the
function, i.e. `png:at(0, 0)` is equal to `png.at(png, 0, 0)`. So in the C
function `png_at`, we are able to retrive png object data from the leftmost
parameter.

    
    
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
    30
    

|

    
    
    static int png_at(lua_State *L)
    {
        // png is the leftmost parameter
        lua_getfield(L, 1, "width");
        lua_Integer width = luaL_checkinteger(L, -1);
        lua_getfield(L, 1, "height");
        lua_Integer height = luaL_checkinteger(L, -1);
        lua_getfield(L, 1, "data");
        unsigned char *image = (unsigned char *)lua_touserdata(L, -1);
    
        // x and y are the second and third parameters
        lua_Integer x = luaL_checkinteger(L, 2);
        lua_Integer y = luaL_checkinteger(L, 3);
    
        if (x < 0 || x >= width || y < 0 || y >= height)
        {
            lua_pushinteger(L, 0), lua_pushinteger(L, 0),
            lua_pushinteger(L, 0), lua_pushinteger(L, 0);
            return 4;
        }
    
        // Return r,g,b,a
        lua_Integer index = (y * width + x) * 4;
        lua_pushinteger(L, (unsigned int)image[index]);
        lua_pushinteger(L, (unsigned int)image[index + 1]);
        lua_pushinteger(L, (unsigned int)image[index + 2]);
        lua_pushinteger(L, (unsigned int)image[index + 3]);
    
        return 4;
    }  
  
---|---  
  
In this case, we read the `data`, `width` and `height` from the leftmost
parameter and return the r, g, b, a value to the stack. The positive index and
negative index are especially useful when you need to access tables.

Last but not least, the `data` is a `lightuserdata` and must be managed by the
user himself. So we can either write a `dispose` function or overwrite the
`__gc` function in metatable to clean up the data.

    
    
    1
    2
    3
    4
    5
    6
    7
    

|

    
    
    static int png_dispose(lua_State *L)
    {
        <span class="n"