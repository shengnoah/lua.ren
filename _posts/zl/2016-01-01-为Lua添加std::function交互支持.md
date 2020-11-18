---
layout: post
title: 为Lua添加std::function交互支持 
tags: [lua文章]
categories: [lua文章]
---
我们常用的事件等机制需要设置回调函数，在C++中比较方便的是std::function配合lambda使用。当我们使用lua进行脚本化的时候，lua和C++交互便会遇到这样的问题，那么如何在lua脚本中设置和获取std::function作为变量呢？要解决这个问题，我们需要搞定两个方向——Lua->C++和
C++->Lua。

## 实现步骤

### Lua->C++

假如我们有一个回调函数需要设置`std::function<void(int)>` 该函数在Lua中可能是这样的：  

    
    
    1  
    2  
    3  
    

|

    
    
    function(tag)  
    print("tag is :" .. tag)  
    end  
      
  
---|---  
  
它是一个Lua函数，我们可以在Lua栈中获取到它，那么如果我需要在以后调用它，我可以使用luaL_ref将其注册，在需要调用的时候获取到它的引用，并使用lua_pcall调用它。这样一来思路就清晰了，我们只需要使用lambda表达式构造出一个std::function，内容就是获取到这个Lua
function并调用，在这个过程中使用lambda捕获Lua function的引用id，并在function销毁时进行解引用。

### C++->Lua

如果是一个std::function压入Lua栈呢，我们可以简单的使用userdata进行储存，使用lua_newuserdata开一块内存，将std::function拷贝进去，设置好元表，这样就完成了压入栈的操作。

### C++->Lua->C++

通过2存入std::function这样问题又来了，传进来的参数有可能是一个function也有可能是一个userdata，我们需要对其分别处理，如果是function，使用lambda进行转换，如果是userdata，直接还原内存。

经过上面的分析，我们可以实现其以下代码：  

    
    
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
    32  
    33  
    34  
    35  
    36  
    37  
    38  
    39  
    40  
    41  
    42  
    43  
    44  
    45  
    46  
    47  
    48  
    49  
    50  
    51  
    52  
    53  
    54  
    55  
    56  
    57  
    58  
    59  
    60  
    61  
    62  
    63  
    64  
    65  
    66  
    67  
    68  
    69  
    70  
    71  
    72  
    73  
    74  
    75  
    76  
    77  
    78  
    79  
    80  
    81  
    82  
    83  
    84  
    85  
    86  
    87  
    88  
    89  
    90  
    91  
    92  
    93  
    

|

    
    
    class  {  
        lua_State *state;  
        int ref;  
    public:  
        FunctionTransfer(lua_State *L, int index)  
        {  
            state = L;  
            ref = 0;  
            if (state) {  
                lua_pushvalue(state, index);  
                ref = luaL_ref(state, LUA_REGISTRYINDEX);  
            }  
        }  
      
        template<typename R , typename... P>  
        static void create(lua_State *L, int index, std::function<R(P...)> &func)  
        {  
            auto auf = std::make_shared<FunctionTransfer>(L, index);  
            func = [auf](P... p)->R{  
                lua_State *L = auf->getState();  
                lua_rawgeti(L, LUA_REGISTRYINDEX, auf->getRef());  
                int nargs = pushArgs(L, p...);  
                lua_pcall(L, nargs, 1, 0);  
                return Stack<R>::get(L, -1);  
            };  
        }  
      
        template<typename R = void, typename... P>  
        static void create(lua_State *L, int index, std::function<void(P...)> &func)  
        {  
            auto auf = std::make_shared<FunctionTransfer>(L, index);  
            func = [auf](P... p)->void{  
                lua_State *L = auf->getState();  
                lua_rawgeti(L, LUA_REGISTRYINDEX, auf->getRef());  
                int nargs = pushArgs(L, p...);  
                lua_pcall(L, nargs, 0, 0);  
            };  
        }  
      
        template<typename R, typename... P>  
        static void create(lua_State *L, int index, std::function<void(void)> &func)  
        {  
            auto auf = std::make_shared<FunctionTransfer>(L, index);  
            func = [auf]()->void{  
                lua_State *L = auf->getState();  
                lua_rawgeti(L, LUA_REGISTRYINDEX, auf->getRef());  
                lua_pcall(L, 0, 0, 0);  
            };  
        }  
      
        template<typename FT>  
        static void get(lua_State *L, int index, std::function<FT> &func)  
        {  
            if(lua_isfunction(L, index))  
            {  
                create(L, index, func);  
            }  
            else if (lua_isuserdata(L, index))  
            {  
                func = (decltype(func)(*luaL_checkudata(L, index, typeid(func).name())));  
            } else {  
                luaL_checktype(L, index, LUA_TFUNCTION);  
            }  
        }  
      
        int getRef(){return ref;}  
        lua_State* getState(){return state;}  
      
        ~FunctionTransfer()  
        {  
            if (state) {  
                luaL_unref(state, LUA_REGISTRYINDEX, ref);  
            }  
        }  
    };  
      
    template <typename FT>  
    struct Stack <std::function<FT> >  
    {  
        static inline void push (lua_State* L, std::function<FT> func)  
        {  
            new (lua_newuserdata(L, sizeof(func))) std::function<FT>(func);  
            luaL_newmetatable(L, typeid(func).name());  
            lua_setmetatable(L, -2);  
        }  
      
        static inline std::function<FT> get (lua_State* L, int index)  
        {  
            std::function<FT> func;  
            FunctionTransfer::get(L, index, func);  
            return func;  
        }  
    };  
      
  
---|---  
  
## 进一步优化

这里的std::function的userdata的metatable是没有实现__call方法的，因此实际上在lua层并不能调用这个方法。有需要的话可以自行实现。