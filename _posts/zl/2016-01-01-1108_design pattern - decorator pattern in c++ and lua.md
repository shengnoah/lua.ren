---
layout: post
title: design pattern - decorator pattern in c++ and lua 
tags: [lua文章]
categories: [topic]
---
  * What is Decorator Pattern
  * Example in C++
  * Example in Lua

  
![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e9/Decorator_UML_class_diagram.svg/400px-
Decorator_UML_class_diagram.svg.png)

  
[Decorator Pattern Wiki](https://en.wikipedia.org/wiki/Decorator_pattern)  

> In object-oriented programming, the decorator pattern is a design pattern
> that allows behavior to be added to an individual object, either statically
> or dynamically, without affecting the behavior of other objects from the
> same class. The decorator pattern is often useful for adhering to the Single
> Responsibility Principle, as it allows functionality to be divided between
> classes with unique areas of concern.

**  
一一一一一一一一一一一一一一一一一一一一一一一一  
© Hung-Chi's Blog  
[ https://hungchicheng.github.io/2017/11/07/Design-Patterns-Decorator-Pattern-
in-lua-and-C++/ ](https://hungchicheng.github.io/2017/11/07/Design-Patterns-
Decorator-Pattern-in-lua-and-C++/)  
一一一一一一一一一一一一一一一一一一一一一一一一 **

  

## What is Decorator Pattern

## Example in C++

    
    
    #include <iostream>
    using namespace std;
    
    class I {
    public:
        virtual ~I(){}
        virtual void show() = 0;
    };
    
    class Hero: public I {
    public:
        Hero(string n):name(n){}
        ~Hero() {
            cout << name << " dtor" << 'n';
        }
        /*virtual*/
        void show() {
            cout << name << " with ";
        }
    private:
        string name;
    };
    
    class Equiment: public I {
    public:
        Equiment(I *inner) {
            m_wrappee = inner;
        }
        ~Equiment() {
            delete m_wrappee;
        }
        /*virtual*/
        void show() {
            m_wrappee->show();
        }
    private:
        I *m_wrappee;
    };
    
    class Helmet: public Equiment {
    public:
        Helmet(I *core): Equiment(core){}
        ~Helmet() {
            cout << "Helmet dtor, ";
        }
        /*virtual*/
        void show() {
            Equiment::show();
            cout << "Helmet, ";
        }
    };
    
    class Armour: public Equiment {
    public:
        Armour(I *core): Equiment(core){}
        ~Armour() {
            cout << "Armour dtor, ";
        }
        /*virtual*/
        void show() {
            Equiment::show();
            cout << "Armour, ";
        }
    };
    
    class Boots: public Equiment {
    public:
        Boots(I *core): Equiment(core){}
        ~Boots() {
            cout << "Boots dtor, ";
        }
        /*virtual*/
        void show() {
            Equiment::show();
            cout << "Boots, ";
        }
    };
    
    int main() {
        I *hero1H = new Helmet(new Hero("Hero1"));
        I *hero2HA = new Armour(new Helmet(new Hero("Hero2")));
        I *hero3HAB = new Boots(new Armour(new Helmet(new Hero("Hero3"))));
        I *hero4BH = new Helmet(new Boots(new Hero("Hero4"))); //Hero 4 with no Armour
        hero1H->show();
        cout << 'n';
        hero2HA->show();
        cout << 'n';
        hero3HAB->show();
        cout << 'n';
        hero4BH->show();
        cout << 'n';
        delete hero1H;
        delete hero2HA;
        delete hero3HAB;
        delete hero4BH;
    }
    

Output:

    
    
    Hero1 with Helmet, 
    Hero2 with Helmet, Armour, 
    Hero3 with Helmet, Armour, Boots, 
    Hero4 with Boots, Helmet, 
    Helmet dtor, Hero1 dtor
    Armour dtor, Helmet dtor, Hero2 dtor
    Boots dtor, Armour dtor, Helmet dtor, Hero3 dtor
    Helmet dtor, Boots dtor, Hero4 dtor
    Program ended with exit code: 0
    

[Download - Source
Code](https://github.com/hungchicheng/DesignPattern/blob/master/C%2B%2B/Decorator.cpp)  
  

## Example in Lua

    
    
    function FuncNew( obj ) -- for Inheritance 
        function obj:new( o )
            o = o or {}
            setmetatable( o, self )
            self.__index = self
            return o
        end
        return obj
    end
    
    Hero = {}
    function Hero:create( name )
        return FuncNew( Hero ):new({ m_name = name })
    end
    function Hero:show()
        print( self.m_name .. " with " )
    end
    
    Helmet = {}
    function Helmet:create( child )
        return FuncNew( Helmet ):new({ m_child = child })
    end
    function Helmet:show()
        self.m_child:show()
        print( "Helmet, " )
    end
    
    Armour = {}
    function Armour:create( child )
        return FuncNew( Armour ):new({ m_child = child })
    end
    function Armour:show()
        self.m_child:show()
        print( "Armour, " )
    end
    
    Boots = {}
    function Boots:create( child )
        return FuncNew( Boots ):new({ m_child = child })
    end
    function Boots:show()
        self.m_child:show()
        print( "Boots, " )
    end
    
    ------------------------------------------------------
    
    local hero1H = Helmet:create(Hero:create("Hero1"))
    local hero2HA = Armour:create(Helmet:create(Hero:create("Hero2")))
    local hero3HAB = Boots:create(Armour:create(Helmet:create(Hero:create("Hero3"))))
    local hero4BH = Helmet:create(Boots:create(Hero:create("Hero4")))
    
    hero1H:show()
    hero2HA:show()
    hero3HAB:show()
    hero4BH:show()
    print("nLua automatically deletes objects that become garbagen")
    
    

Output:

    
    
    Hero1 with 
    Helmet, 
    Hero2 with 
    Helmet, 
    Armour, 
    Hero3 with 
    Helmet, 
    Armour, 
    Boots, 
    Hero4 with 
    Boots, 
    Helmet, 
    
    Lua automatically deletes objects that become garbage
    
    [Finished in 0.0s]
    

[Download - Source
Code](https://github.com/hungchicheng/DesignPattern/blob/master/Lua/Decorator.lua)