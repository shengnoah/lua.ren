---
layout: post
title: 《实现angluar》手记九:表达式与watcher结合 
tags: [lua文章]
categories: [topic]
---
## 表达式与watcher结合

在前面的章节中, `Scope.$watch`函数接受的`watchFn`仅仅是一个返回值的普通函数,
而本章节中表达式与watcher连接的关键点是将watcher中的`watchFn`替换成`parse(watchFn)`,
将parse生成的新函数作为watchFn, 当然,我们也需要对应的对`$watchCollection`,`$eval`,`$apply`,
`evalAsync`等方法做处理

    
    
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

    
    
    Scope.prototype.$watch = function(watchFn, listenerFn, valueEq) {  
        var self = this;  
        var watcher = {  
            watchFn: parse(watchFn), // wathcCollection等方法也需要做一并处理  
            listenerFn: listenerFn || function() { },  
            last: initWatchVal,  
            valueEq: !!valueEq  
        };  
    }  
      
  
---|---  
  
当然，parse需要对传入的参数做处理， 如果已经是一个函数的话就直接返回

    
    
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
    

|

    
    
    function (expr) {  
      switch (typeof expr) {  
        case 'string':  
          var lexer = new Lexer();  
          var parser = new Parser(lexer);  
          return parser.parse(expr);  
        case 'function':  
          return expr;  
        default:  
          return _.noop;  
      }  
    }  
      
  
---|---  
  
## 字面量与常量表达式

优化： 标记watcher检测的表达式为常量还是变量(比如渲染一个列表)， 在第一次被触发的时候，
如果是常量就可以直接将watcher移除，这样在digest的过程中就不会遍历该watcher(Vue在编译的过程中也有相应的静态节点标记)

为此,我们明确两个概念:

  * 字面量, 如42, [42, ‘abc’], [42, ‘abc’, aVariable]
  * 常量,42, [42, ‘abc’]是常量,而[42, ‘abc’, aVariable]不是, 因为aVarible的存在

实现的方式是为parse生成的函数添加额外的参数, `literal`, `constant`,

### literal

先来看literal

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
      
    ASTCompiler.prototype.compile = function(text) {  
      ....  
      fn.literal = isLiteral(ast);  
    }  
      
  
---|---  
  
这里, isLiteral函数进行以下检测

  * 一个空的program(函数的body为空)是字面量
  * 非空的program, 其body只有一个表达式, 且表达式的类型为`array`或者`object`

### constant

常量的判断比较复杂, 需要递归地对每一个节点做判断, 只有当所有的节点都为常量的时候才是常量

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    ASTCompiler.prototype.compile = function(text) {  
      markConstantExpressions(ast);  
      ....  
      fn.constant = ast.constant;  
    }  
      
  
---|---  
  
markConstantExpression函数针对每类节点进行标记

    
    
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

    
    
    function markConstantAndWatchExpressions(ast) {  
      var allConstants;  
      var argsToWatch;  
      switch (ast.type) {  
      case AST.Program:  
        allConstants = true;  
        _.forEach(ast.body, function(expr) {  
          markConstantAndWatchExpressions(expr);  
          allConstants = allConstants && expr.constant;  
        });  
        ast.constant = allConstants;  
        break;  
      case AST.Literal:  
        ast.constant = true;  
        ast.toWatch = [];  
        break;  
        ....  
    }  
      
  
---|---  
  
## 优化常量表达式的监测

常量表达式永远返回相同的值, 这也意味着常量表达式第一次被触发之后再也不会变”脏”,因此我们可以移除相应的watcher

`watch delegate`, 当Scope.$watch中出现watcher delegate中的表达式的时候，
该代理会绕过正常的watcher创建的过程。

    
    
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

    
    
    function constantWatchDelegate(scope, listenerFn, valueEq, watchFn) {  
        var unwatch = scope.$watch(  
            function () {  
                return watchFn(scope);  
            },  
            function (newValue, oldValue, scope) {  
                if (_.isFunction(listenerFn)) {  
                    listenerFn.apply(this, arguments);  
                }  
                unwatch();  
            },  
            valueEq  
        );  
        return unwatch;  
    }  
      
  
---|---  
  
## One-Time Expressions

只执行一次的表达式: 比如列表项, 里面的内容一旦渲染之后就不会发生变化

    
    
    1  
    2  
    3  
    

|

    
    
    <li ng-repeat="user in users">  
    {{::user.firstName}} {{::user.lastName}}  
    </li>  
      
  
---|---  
  
## input-tracking

    
    
    1  
    2  
    3  
    4  
    

|

    
    
    ```  
      
    ## stateful Filters  
    ```js  
      
  
---|---  
  
## External Assignment