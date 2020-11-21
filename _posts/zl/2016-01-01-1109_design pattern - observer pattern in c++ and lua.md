---
layout: post
title: design pattern - observer pattern in c++ and lua 
tags: [lua文章]
categories: [topic]
---
  * What is Observer Pattern
  * Example in C++
  * Example in Lua

  
![](https://upload.wikimedia.org/wikipedia/commons/0/01/W3sDesign_Observer_Design_Pattern_UML.jpg)

  
[Observer Pattern Wiki](https://en.wikipedia.org/wiki/Observer_pattern)  

> The observer pattern is a software design pattern in which an object, called
> the subject, maintains a list of its dependents, called observers, and
> notifies them automatically of any state changes, usually by calling one of
> their methods.

**  
一一一一一一一一一一一一一一一一一一一一一一一一  
© Hung-Chi's Blog  
[ https://hungchicheng.github.io/2017/09/29/Design-Patterns-Observer-Pattern-
in-lua-and-C++/ ](https://hungchicheng.github.io/2017/09/29/Design-Patterns-
Observer-Pattern-in-lua-and-C++/)  
一一一一一一一一一一一一一一一一一一一一一一一一 **

  

## What is Observer Pattern

## Example in C++

    
    
    #include <iostream>
    #include <set>
    using namespace std;
    
    class Observer{
    public:
        virtual void update(int p) = 0;
    };
    
    class Subject{
    protected:
        std::set< Observer* > m_observerList;
    public:
        void attach( Observer *o ){ m_observerList.insert( o ); };
        void detach( Observer *o ){ m_observerList.erase( o ); };
        virtual void notify () = 0;
    };
    
    class  Subject1:public Subject{
    private:
        int m_state;
    public:
        void notify (){
            for ( auto &o : m_observerList ){
                o->update(m_state);
            }
        };
        void setState( int s ){
            m_state = s;
            notify();
        }
        int getState(){ return m_state; }
    };
    
    class Observer1:public Observer{
        string m_name;
        int m_state;
    public:
        Observer1( string name ):m_name( name ){}
        void update( int p ){ m_state = p; } // override
        string getName(){ return m_name; }
        int getState(){ return m_state; }
    };
    
    class Observer2:public Observer{
        string m_name;
        int m_state;
    public:
        Observer2( string name ):m_name( name ){}
        void update( int p ){ m_state = p; } // override
        string getName(){ return m_name; }
        int getState(){ return m_state; }
    };
    
    int main(int argc, char* argv[])
    {
        Subject1 product;
        Observer1 shop1( "shop1--" );
        Observer2 shop2( "shop2--" );
        
        product.attach( &shop1 );
        product.attach( &shop2 );
        product.setState( 12 );
        cout<< shop1.getName() << shop1.getState() <<endl;
        cout<< shop2.getName() << shop2.getState() <<endl;
        
        product.detach( &shop2 );
        product.setState( 11 );
        cout<< shop1.getName() << shop1.getState() <<endl;
        cout<< shop2.getName() << shop2.getState() <<endl;
        
        return 0;
    }
    

Output:

    
    
    shop1--12
    shop2--12
    shop1--11
    shop2--12
    

[Download - Source
Code](https://github.com/hungchicheng/DesignPattern/blob/master/C%2B%2B/Observer.cpp)  
  

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
    
    Observer = {}
    function Observer:create()
        function self:update( p ) -- virtual update
            -- do nothing
        end
        return FuncNew( Observer ):new()
    end
    
    Subject = {}
    function Subject:create()
        self.m_observerList = {}
        function self:attach( observer )
            table.insert( self.m_observerList, observer )
        end
        function self:detach( observer )
            for k,v in pairs( self.m_observerList ) do
                if v == observer then
                    table.remove( self.m_observerList, k )
                end
            end
        end
        function self:notify() -- virtual notify
            -- do nothing
        end
        return FuncNew( Subject ):new()
    end
    
    Subject1 = Subject:create() -- inheritance Subject
    function Subject1:create()
        local m_state = nil
        function self:notify() -- override notify
            for k,v in pairs( self.m_observerList ) do
                v:update( m_state )
            end
        end
        function self:setState( s ) 
            m_state = s
            self:notify()
        end
        function self:getState( s ) 
            return m_state
        end
        return FuncNew( Subject1 ):new()
    end
    
    Observer1 = Observer:create() -- inheritance Subject
    function Observer1:create( n )
        local m_name = n
        local m_state = nil
        function self:update( p ) -- override update
            m_state = p
        end
        function self:getName()
            return m_name
        end
        function self:getState()
            return m_state
        end
        return FuncNew( Observer1 ):new()
    end
    
    Observer2 = Observer:create() -- inheritance Subject
    function Observer2:create( n )
        local m_name = n
        local m_state = nil
        function self:update( p ) -- override update
            m_state = p
        end
        function self:getName()
            return m_name
        end
        function self:getState()
            return m_state
        end
        return FuncNew( Observer2 ):new()
    end
    
    ------------------------------------------------------
    
    local product = Subject1:create()
    local shop1 = Observer1:create( "shop1--" )
    local shop2 = Observer2:create( "shop2--" )
    product:attach( shop1 )
    product:attach( shop2 )
    product:setState( 12 )
    --print( shop1.m_state )
    print( shop1:getName() .. tostring( shop1:getState() ) )
    print( shop2:getName() .. tostring( shop2:getState() ) )
    print( "" )
    product:detach( shop2 )
    product:setState( 11 )
    print( shop1:getName() .. tostring( shop1:getState() ) )
    print( shop2:getName() .. tostring( shop2:getState() ) )
    

Output:

    
    
    shop1--12
    shop2--12
    
    shop1--11
    shop2--12
    [Finished in 0.0s]
    

[Download - Source
Code](https://github.com/hungchicheng/DesignPattern/blob/master/Lua/Observer.lua)