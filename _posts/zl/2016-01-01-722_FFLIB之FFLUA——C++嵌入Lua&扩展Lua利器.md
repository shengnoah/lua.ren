---
layout: post
title: FFLIB之FFLUA——C++嵌入Lua&扩展Lua利器 
tags: [lua文章]
categories: [topic]
---
## 摘要:

在使用C++做服务器开发中，经常会使用到脚本技术，Lua是最优秀的嵌入式脚本之一。Lua的轻量、小巧、概念之简单，都使他变得越来越受欢迎。本人也使用过python做嵌入式脚本，二者各有特点，关于python之后会写相关的文章，python对于我而言更喜欢用来编写工具，我前边一些相关的算法也是用python来实现的。今天主要讲Lua相关的开发技术。Lua具有如下特点：

  * Lua 拥有虚拟机的概念，而其全部用标准C实现，不依赖任何库即可编译安装，更令人欣喜的是，整个Lua 的实现代码并不算多，可以直接继承到项目中，并且对项目的编译时间几乎没有什么影响
  * Lua的虚拟机是线程安全的，这里讲的线程安全级别指得是STL的线程安全级别，即一个lua虚拟机被一个线程访问是安全的，多个lua虚拟机被多个线程分别访问也是安全的，一个lua虚拟机被多个线程访问是不安全的。
  * Lua的概念非常少，数据结构只有table，这样当使用Lua作为项目的配置文件时，即使没有编程功底的策划也可以很快上手编写。
  * Lua没有原生的对象，没有class的关键字，这也保障了其概念简单，但是仍然是可以使用Lua面向对象编程的。
  * Lua尽管小巧，却支持比较先进的编程范式，lua 中的匿名函数和闭包会让代码写起来更加 优雅和高效，如果某人使用的C++ 编译器还比较老套，不支持C++11，那么可以尽快感受一下lua的匿名函数和闭包。
  * Lua是最高效的嵌入式脚本之一（如果不能说最的话，目前证据显示是最）。
  * Lua的垃圾回收也可以让C++程序收益匪浅，这也是C++结合脚本技术的重要优势之一。
  * Lua 的嵌入非常的容易，CAPI 相对比较简洁，而且文档清晰，当然Lua的Capi需要掌握Lua中独特的堆栈的概念，但仍然足够简单。
  * Lua的扩展也非常的容易，将C++是对象、函数导入到lua中会涉及到一些技巧，如果纯粹使用lua CAPI会稍显繁杂，幸运的是一些第三方库简化了这些操作，而FFLUA绝对是最好用的之一。

### 嵌入Lua：

嵌入lua脚本，必须要把lua脚本载入lua虚拟机，lua中的概念称之为dofile，FFLUA中封装了dofile的操作，由于lua文件可能集中放在某个目录，FFLUA中也提供了设置lua脚本目录的接口：  

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    

|

    
    
    int  (const string& str_)  
    int  load_file(const string& file_name_) throw (lua_exception_t)  
    load_file就是执行dofile操作，若出错，则throw异常对象，可以使用exception引用目标对象使用what接口输出代码出错的traceback。  
      
      
        template<typenameT>  
        int  get_global_variable(conststring& field_name_, T& ret_);  
        template<typenameT>  
        int  get_global_variable(constchar* field_name_, T& ret_);  
      
  
---|---  
  
有时需要直接执行一些lua语句，lua中有dostring的概念，FFLUA中封装了单独的接口run_string：

void run_string(constchar* str_) throw (lua_exception_t)  
嵌入lua时最一般的情况是调用lua中的函数，lua的函数比C++更灵活，可以支持任意多个参数，若未赋值，自动设置为nil，并且可以返回多个返回值。无论如何，从C++角度讲，当你嵌入lua调用lua函数时，你总希望lua的使用方式跟C++越像越好，你不希望繁复的处理调用函数的参数问题，比如C++数据转换成lua能处理的数据，即无趣又容易出错。正也正是FFLUA需要做到，封装调用lua函数的操作，把赋值参数，调用函数，接收返回值的细节做到透明，C++调用者就像调用普通的C++函数一样。使用FFLUA中调用lua函数使用call接口：

void call(constchar* func_name_) throw (lua_exception_t)  
当调用出错时，异常信息记录了traceback。

实际上，FFLUA重载了9个call函数，以来自动适配调用9个参数的lua函数。

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    

|

    
    
    template<typename RET>  
         RET call(const char* func_name_) throw (lua_exception_t);  
        ......  
        template<typename RET, typename ARG1, typename ARG2, typename ARG3, typename ARG4,  
                 typename ARG5, typename ARG6, typename ARG7, typename ARG8, typename ARG9>  
        RET call(const char* func_name_, ARG1 arg1_, ARG2 arg2_, ARG3 arg3_,  
                 ARG4 arg4_, ARG5 arg5_, ARG6 arg6_, ARG7 arg7_,  
                 ARG8 arg8_, ARG9 arg9_) throw (lua_exception_t);  
      
  
---|---  
  
需要注明的是：  
call接口的参数是范型的，自动会使用范型traits机制转换成lua类型，并push到lua堆栈中  
call接口的返回值也是范式的，这就要求使用call时必须提供返回值的类型，如果lua函数不返回值会怎样？lua中有个特性，只有nil和false的布尔值为false，所以当lua函数返回空时，你仍然可以使用bool类型接收参数，只是调用者忽略其返回值就行了。  
call只支持一个返回值，虽然lua可以返回多个值，但是call会忽略其他返回值，这也是为了尽可能像是调用C++函数，若要返回多个值，完全可以用table返回。

## 扩展LUA：

这也是非常重要的操作，嵌入lua总是和扩展lua相伴相行。lua若要操作C++中的对象或函数，那么必须先把C++对应的接口注册都lua中。Lua
CAPI提供了一系列的接口拥有完成此操作，但是关于堆栈的操作总是会稍显头疼，fflua极大的简化了注册C++对象和接口的操作，可以说是最简单的注册方式之一（如果不准说最的话）。首先我们整理一下需要哪些注册操作：

  * C++ 静态函数注册为lua中的全局函数，这样在lua中调用C++函数就像是调用C++全局函数
  * C++对象注册成Lua中的对象，可以通过new接口在lua中创建C++对象
  * C++类中的属性注册到lua，lua访问对象的属性就像是访问table中的属性一样。
  * C++类中的函数注册到lua中，lua调用其接口就像是调用talbe中的接口一样。

FFLUA中提供了一个范型接口，适配于注册C++相关数据：  

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    template<typename T>  
    void  fflua_t::reg(T a)  
    {  
        a(this->get_lua_state());  
    }  
      
  
---|---  
  
这样，若要对lua进行注册操作，只需要提供一个仿函数即可，这样可以批量在注册所有的C++数据，当然FFLUA中提供了工具类用于生成仿函数中应该完成的注册操作：

    
    
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
    

|

    
    
    template<typename CLASS_TYPE = op_tool_t, typename CTOR_TYPE = void()>  
    class fflua_register_t  
    {  
    public:  
        fflua_register_t(lua_State* ls_):m_ls(ls_){}  
        fflua_register_t(lua_State* ls_, const string& class_name_, string inherit_name_ = "");  
      
      
        template<typename FUNC_TYPE>  
        fflua_register_t& def(FUNC_TYPE func, const string& s_)  
        {  
            fflua_register_router_t<FUNC_TYPE>::call(this, func, s_);  
            return *this;  
        }  
    };  
      
  
---|---  
  
刚才提到的像lua中的所有注册操作，都可以使用def操作完成。 示例如下：

//! 注册子类，ctor(int) 为构造函数， foo_t为类型名称， base_t为继承的基类名称  

    
    
    1  
    2  
    3  
    

|

    
    
    fflua_register_t<foo_t, ctor(int)>(ls, "foo_t", "base_t")  
                .def(&foo_t::print, "print")        //! 子类的函数  
                .def(&foo_t::a, "a");               //! 子类的字段  
      
  
---|---  
  
尤其特别的是，C++中的继承可以在注册到lua中被保持这样注册过基类的接口，子类就不需要重复注册。

高级特性：

通过以上的介绍，也许你已经了解了FFLUA的设计原则，即：当在编写C++代码时，希望使用LUA就像使用C++本地的代码一样，而在lua中操作C++的数据和接口的时候，又希望C++用起来完全跟table一个样。这样可以大大减轻程序开发的工作，从而把精力更多放大设计和逻辑上。那么做到如何lua才算像C++，C++做到如何才算像lua呢？我们知道二者毕竟相差甚远，我们只需要把常见的操作封装成一直即可，不常见操作则特殊处理。常见操作有：

  * C++ 调用lua函数，FFLUA已经封装了call函数，保障了调用lua函数就像调用本地C++函数一样方便
  * C++注册接口和对象到lua中，lua中操作对象就像操作table一样直接。
  * C++中除了自定义对象，STL是用的最多的了，C++希望lua中能够接收STL的参数，或者能够返回STL数据结构
  * Lua中只有table数据结构，Lua希望C++的参数的数据结构支持table，并且lua可以直接把table作为返回值。
  * C++的指针需要传递到lua中，同时也希望某些操作，lua可以把C++对象指针作为返回值  
以上前两者已经介绍了，而后三者FFLUA也是给予 完美支持。通过范型的C++封装，可以将C++
STL完美的转换成luatable，同时在lua返回table的时候，自动根据返回值类型将lua的table转换成C++
STL。FFLUA中只要被注册过的C++对象，都可以把其指针作为参数赋值给lua，甚至在lua中保存。当我讲述以上特性的时候，都是在保证类型安全的前提下。重要的类型检查有：

STL转成Luatable时，STL中的类型必须是lua支持的，包括基本类型和已经注册过的C++对象指针。并且STL可以嵌套使用，如vector<list
>,
不要惊讶，这是支持的，不管嵌套多少层，都是支持的，使用C++模板的递归机制，该特性得到了完美支持。vector、list、set都会转换成table的数组模式，key从1开始累加。而map类型自动适配为table字典。  
LUA中的table可以被当成返回值转换成C++
STL，转换跟上边刚好是对应的，当然有一个限制，由于C++的STL类型必须是唯一的，如vector的返回值就要求lua中的table所有值都是int。否则FFLUA会返回出错，并提示类型转换失败  
无论死调用lua中使用C++对象指针，还是LuA中返回C++对象指针，该对象必须是lua可以识别的，即已经被注册的，否则FFLUA会提示转换类型失败。

### 关于重载：

关于重载LUA
可以使用lua中内部自己的reload，也可以将fflua对象销毁后，重先创建一个，创建fflua对象的开销和创建lua虚拟机的开销一直，不会有附加开销。

## 总结：

  * FFLUA是简化C++嵌入绑定lua脚本的类库
  * FFLUA只有三个头文件，不依赖除lua之外的任何的类库，开发者可以非常容易的使用FFLUA
  * FFLUA 对于常用的STL数据结构进行了支持
  * FFLUA 即使拥有了这么多特性，仍然保持了轻量，只要用过C++，只要用过lua，FFLUA的代码就可以非常清晰的看清其实现，当你了解其内部实现时，你会发现FFLUA已经做到了极简，范型模板展开后的代码就跟你自己原生LUA CAPI 编写的一样直接。
  * FFLUA的开源代码：<https://github.com/fanchy/fflua>
  * 完整的C++示例代码：

    
    
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
    94  
    95  
    96  
    97  
    98  
    99  
    100  
    101  
    102  
    103  
    104  
    105  
    106  
    107  
    108  
    109  
    110  
    111  
    112  
    113  
    114  
    115  
    116  
    117  
    118  
    119  
    120  
    121  
    122  
    123  
    124  
    125  
    126  
    127  
    128  
    

|

    
    
      
    #include <string>  
    #include <assert.h>  
    using namespace std;  
      
    #include "lua/fflua.h"  
      
    using namespace ff;  
      
    class base_t  
    {  
    public:  
        base_t():v(789){}  
        void dump()  
        {  
            printf("in %s a:%dn", __FUNCTION__, v);  
        }  
        int v;  
    };  
    class foo_t: public base_t  
    {  
    public:  
        foo_t(int b):a(b)  
        {  
            printf("in %s b:%d this=%pn", __FUNCTION__, b, this);  
        }  
        ~foo_t()  
        {  
            printf("in %sn", __FUNCTION__);  
        }  
        void print(int64_t a, base_t* p) const  
        {  
            printf("in foo_t::print a:%ld p:%pn", (long)a, p);  
        }  
      
        static void dumy()  
        {  
            printf("in %sn", __FUNCTION__);  
        }  
        int a;  
    };  
      
    //! lua talbe 可以自动转换为stl 对象  
    void dumy(map<string, string> ret, vector<int> a, list<string> b, set<int64_t> c)  
    {  
        printf("in %s begin ------------n", __FUNCTION__);  
        for (map<string, string>::iterator it =  ret.begin(); it != ret.end(); ++it)  
        {  
            printf("map:%s, val:%s:n", it->first.c_str(), it->second.c_str());  
        }  
        printf("in %s end ------------n", __FUNCTION__);  
    }  
      
    static void lua_reg(lua_State* ls)  
    {  
        //! 注册基类函数, ctor() 为构造函数的类型  
        fflua_register_t<base_t, ctor()>(ls, "base_t")  //! 注册构造函数  
                        .def(&base_t::dump, "dump")     //! 注册基类的函数  
                        .def(&base_t::v, "v");          //! 注册基类的属性  
      
        //! 注册子类，ctor(int) 为构造函数， foo_t为类型名称， base_t为继承的基类名称  
        fflua_register_t<foo_t, ctor(int)>(ls, "foo_t", "base_t")  
                    .def(&foo_t::print, "print")        //! 子类的函数  
                    .def(&foo_t::a, "a");               //! 子类的字段  
      
        fflua_register_t<>(ls)  
                    .def(&dumy, "dumy");                //! 注册静态函数  
    }  
      
    int main(int argc, char* argv[])  
    {  
      
        fflua_t fflua;  
        try   
        {  
            //! 注册C++ 对象到lua中  
            fflua.reg(lua_reg);  
              
            //! 载入lua文件  
            fflua.add_package_path("./");  
            fflua.load_file("test.lua");  
              
            //! 获取全局变量  
            int var = 0;  
            assert(0 == fflua.get_global_variable("test_var", var));  
            //! 设置全局变量  
            assert(0 == fflua.set_global_variable("test_var", ++var));  
      
            //! 执行lua 语句  
            fflua.run_string("print("exe run_string!!")");  
              
            //! 调用lua函数, 基本类型作为参数  
            int32_t arg1 = 1;  
            float   arg2 = 2;  
            double  arg3 = 3;  
            string  arg4 = "4";  
            fflua.call<bool>("test_func", arg1, arg2, arg3,  arg4);  
              
            //! 调用lua函数，stl类型作为参数， 自动转换为lua talbe  
            vector<int> vec;        vec.push_back(100);  
            list<float> lt;         lt.push_back(99.99);  
            set<string> st;         st.insert("OhNIce");  
            map<string, int> mp;    mp["key"] = 200;  
            fflua.call<string>("test_stl", vec, lt, st,  mp);  
              
            //! 调用lua 函数返回 talbe，自动转换为stl结构  
            vec = fflua.call<vector<int> >("test_return_stl_vector");  
            lt  = fflua.call<list<float> >("test_return_stl_list");  
            st  = fflua.call<set<string> >("test_return_stl_set");  
            mp  = fflua.call<map<string, int> >("test_return_stl_map");  
              
            //! 调用lua函数，c++ 对象作为参数, foo_t 必须被注册过  
            foo_t* foo_ptr = new foo_t(456);  
            fflua.call<bool>("test_object", foo_ptr);  
              
            //! 调用lua函数，c++ 对象作为返回值, foo_t 必须被注册过   
            assert(foo_ptr == fflua.call<foo_t*>("test_ret_object", foo_ptr));  
            //! 调用lua函数，c++ 对象作为返回值, 自动转换为基类  
            base_t* base_ptr = fflua.call<base_t*>("test_ret_base_object", foo_ptr);  
            assert(base_ptr == foo_ptr);  
       
        }  
        catch (exception& e)  
        {  
            printf("exception:%sn", e.what());  
        }  
        return 0;  
    }  
      
  
---|---  
  
完整的LUA示例代码：

    
    
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
    68<br