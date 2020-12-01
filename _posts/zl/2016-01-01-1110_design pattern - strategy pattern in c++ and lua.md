---
layout: post
title: design pattern - strategy pattern in c++ and lua 
tags: [lua文章]
categories: [topic]
---
  * What is Strategy Pattern
  * Example in C++
  * Example in Lua

  
![](https://upload.wikimedia.org/wikipedia/commons/4/45/W3sDesign_Strategy_Design_Pattern_UML.jpg)

  
[Strategy Pattern Wiki](https://en.wikipedia.org/wiki/Strategy_pattern)  

> In computer programming, the strategy pattern (also known as the policy
> pattern) is a behavioural software design pattern that enables selecting an
> algorithm at runtime.

**  
一一一一一一一一一一一一一一一一一一一一一一一一  
© Hung-Chi's Blog  
[ https://hungchicheng.github.io/2017/09/25/Design-Patterns-Strategy-Pattern-
in-lua-and-C++/ ](https://hungchicheng.github.io/2017/09/25/Design-Patterns-
Strategy-Pattern-in-lua-and-C++/)  
一一一一一一一一一一一一一一一一一一一一一一一一 **

  

## What is Strategy Pattern

The strategy pattern:

  1. defines a family of algorithms,
  2. encapsulates each algorithm, and
  3. makes the algorithms interchangeable within that family.

## Example in C++

For example, we create a warrior and archer with different attack strategy.

    
    
    #include <iostream>
    using namespace std;
    
    class AttackStrategy{
    public:
        virtual void attack() = 0;
    };
    
    class SwordAttackStrategy: public AttackStrategy{
    public:
        virtual void attack(){ cout << "use sword to attack" << endl; }  // override 
    };
    
    class ArcheryAttackStrategy: public AttackStrategy{
    public:
        virtual void attack(){ cout << "use bow to attack" << endl; }  // override 
    };
    
    class HeroBase{
    private:
        AttackStrategy *m_strategy;
        string m_name;
    public:
        HeroBase( string name, AttackStrategy *strategy ):m_name( name ),m_strategy( strategy ){}
        void set_strategy( AttackStrategy *strategy ){ m_strategy = strategy; }
        void attack(){
            cout << m_name << endl;
            m_strategy -> attack();
        }
    };
    
    int main( int argc, char *argv[] )
    {
        // warrior
        SwordAttackStrategy swordAttackStrategy;
        HeroBase warrior( "warrior1", &swordAttackStrategy );
        warrior.attack();
        // archer
        ArcheryAttackStrategy archeryAttackStrategy;
        HeroBase archer( "archer1", &archeryAttackStrategy );
        archer.attack();
        return 0;
    }
    

Output:

    
    
    warrior1
    use sword to attack
    archer1
    use bow to attack
    Program ended with exit code: 0
    

[Download - Source
Code](https://github.com/hungchicheng/DesignPattern/blob/master/C%2B%2B/Strategy.cpp)  
  

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
    
    AttackStrategy = {}
    function AttackStrategy:create()
        return FuncNew( AttackStrategy ):new()
    end
    function AttackStrategy:attack() -- virtual
        -- do nothing
    end
    
    SwordAttackStrategy = AttackStrategy:create() -- inheritance AttackStrategy
    function SwordAttackStrategy:create()
        return FuncNew( SwordAttackStrategy ):new()
    end
    function SwordAttackStrategy:attack() -- override
        print( "use sword to attack" )
    end
    
    ArcheryAttackStrategy = AttackStrategy:create() -- inheritance AttackStrategy
    function ArcheryAttackStrategy:create()
        return FuncNew( ArcheryAttackStrategy ):new()
    end
    function ArcheryAttackStrategy:attack() -- override
        print( "use bow to attack" )
    end
    
    HeroBase = {}
    function HeroBase:create( n, s )
        return FuncNew( HeroBase ):new({
            m_name = n,
            m_strategy = s or AttackStrategy:create()
        })
    end
    function HeroBase:attack()
        print( self.m_name )
        print( self.m_strategy:attack() )
    end
    
    ------------------------------------------------------
    
    -- local warrior0 = HeroBase:create( "warrior0" )
    -- warrior
    local swordAttackStrategy = SwordAttackStrategy:create()
    local warrior1 = HeroBase:create( "warrior1", swordAttackStrategy )
    -- archer
    local archeryAttackStrategy = ArcheryAttackStrategy:create()
    local archer1 = HeroBase:create( "archer1", archeryAttackStrategy )
    
    -- warrior0:attack()
    warrior1:attack()
    archer1:attack()
    

Output:

    
    
    warrior1
    use sword to attack
    
    archer1
    use bow to attack
    
    [Finished in 0.0s]
    

[Download - Source
Code](https://github.com/hungchicheng/DesignPattern/blob/master/Lua/Strategy.lua)