---
layout: post
title: 深入 Lua Garbage Collector(一) 
tags: [lua文章]
categories: [topic]
---
  
看到一个 **Bob Nystrom** 写的[ **C** 语言实现的 **Garbage Collector**
](http://journal.stuffwithstuff.com/2013/12/08/babys-first-garbage-collector/)

借着这个小程序顺便深入地了解一下Lua的垃圾回收机制

## Garbage Collector算法小结

这是之前做的一点小笔记：

## C Garbage Collector

首先还是先来看看这个 **C** 的基本的垃圾回收器

### 采用的算法

用的是经典的 **Mark & Sweep** 算法

在上面的笔记里面已经介绍的很清楚了

该算法的工作原理几乎与我们对 **可访问性(reachability)** 的定义完全一样：

  1. 从根节点开始，依次遍历整个对象图。每当你访问到一个对象，在上面设置一个 标记(mark) 位，置为 true 。

  2. 一旦搞定，找出所有标记位为 not 的对象集，然后删除它们。

### 对象对

要想清理垃圾，首先我们得制造点垃圾出来

所以假设我们正在为一种简单的语言编写一个解释器。它是动态的类型并且有两种类型的变量： **int** 和 **pair** 。
下面是用枚举来标示一个对象的类型：

    
    
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

|

    
    
    typedef enum {
    
        OBJ_INT,
    
        OBJ_PAIR
    
    } ObjectType;
    
    // 因为一个对象在虚拟机中可以是这两个当中的任意一种类型，所以在C中用tagged union 来实现对象
    
    typedef struct sObject {
    
        //表示对象类型：pair or int
    
        ObjectType type;
    
        unsigned char marked;
    
        //仅维持一张由所有分配过(内存)的对象(组成)的链表。
    
        struct sObject*next;
    
        //union持有这个数据
    
        //union就是一个结构体，它将字段重叠在内存中
    
        union {
    
            int value;
    
            struct {
    
                struct sObject* head;
    
                struct sObject* tail;
    
            };
    
        };
    
    } Object;  
  
---|---  
  
* * *

### 小虚拟机

虚拟机要么基于栈( **JVM** , **CLR** )，要么基于寄存器( **Lua**
)，其实本质上说都是基于栈的，它用来保存一个表达式中间需要用到的临时变量和局部变量

下面我们建立一个简洁的虚拟机模型：

    
    
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

|

    
    
    // 小虚拟机
    
    typedef struct {
    
        Object* stack[STACK_MAX];
    
        int stackSize;
    
        //虚拟机会保留链表头的痕迹
    
        Object* firstObject;
    
        //追踪创建了多少个对象
    
        int numObjects;
    
        //多少数目之后清理
    
        int maxObjects;
    
    } VM;  
  
---|---  
  
* * *

### 虚拟机的操作

    
    
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

    
    
    // 初始化一个虚拟机
    
    VM* () {
    
        VM* vm = malloc(sizeof(VM));
    
        vm->stackSize = 0;
    
        Object* firstObject = NULL;
    
        vm->numObjects = 0;
    
        vm->maxObjects = 8;
    
        return vm;
    
    }
    
    //操作虚拟机堆栈
    
    void push(VM* vm, Object* value) {
    
        assert(vm->stackSize < STACK_MAX, "Stack overflow!!!");
    
        vm->stack[vm->stackSize++] = value;
    
    }
    
    Object* pop(VM* vm) {
    
        assert(vm->stackSize > 0, "Stack underflow!!!");
    
        return vm->stack[--vm->stackSize];
    
    }  
  
---|---  
  
* * *

### Mark

    
    
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

|

    
    
    void mark(Object* object) {
    
        //检查循环!
    
        if (object->marked) return;
    
        object->marked = 1;
    
        if (object->type == OBJ_PAIR) {
    
            mark(object->head);
    
            mark(object->tail);
    
        }
    
    }
    
    void markAll(VM* vm) {
    
        for (int i = 0; i < vm->stackSize; i++) {
    
            mark(vm->stack[i]);
    
        }
    
    }  
  
---|---  
  
* * *

### Sweep

    
    
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

    
    
    //清理
    
    //仅维持一张由所有分配过(内存)的对象(组成)的链表。
    
    void sweep(VM* vm) {
    
        Object** object = &vm->firstObject;
    
        while (*object) {
    
            if (!(*object)->marked) {
    
                //没有访问过，所以从链表中移除然后释放它
    
                Object* unreached = *object;
    
                *object = unreached->next;
    
                free(unreached);
    
                vm->numObjects--;
    
            } else {
    
                //访问过，所以不标记它
    
                (*object)->marked = 0;
    
                object = &(*object)->next;
    
            }
    
        }
    
    }  
  
---|---  
  
* * *

### GC

    
    
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

|

    
    
    void gc(VM* vm) {
    
        int numObjects = vm->numObjects;
    
        markAll(vm);
    
        sweep(vm);
    
        vm->maxObjects = vm->numObjects * 2;
    
        printf("Collected %d objects, %d remaining.n", numObjects - vm->numObjects,
    
                vm->numObjects);
    
    }  
  
---|---  
  
* * *

### 对象操作函数

    
    
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

|

    
    
    Object* newObject(VM* vm, ObjectType type) {
    
        if (vm->numObjects == vm->maxObjects) gc(vm);
    
        Object* object = malloc(sizeof(Object));
    
        object->type = type;
    
        //无论何时创建对象，都添加到链表中
    
        object->next = vm->firstObject;
    
        vm->firstObject = object;
    
        object->marked = 0;
    
        vm->numObjects++;
    
        return object;
    
    }
    
    //编写方法将每种类型的对象压到虚拟机的栈上
    
    void pushInt(VM* vm, int intValue) {
    
        Object* object = newObject(vm, OBJ_INT);
    
        object->value = intValue;
    
        
    
        push(vm, object);
    
    }
    
    Object* pushPair(VM* vm) {
    
        Object* object = newObject(vm, OBJ_PAIR);
    
        object->tail = pop(vm);
    
        object->head = pop(vm);
    
        push(vm, object);
    
        return object;
    
    }
    
    void objectPrint(Object* object) {
    
        switch (object->type) {
    
            case OBJ_INT:
    
            printf("%d", object->value);
    
            break;
    
            case OBJ_PAIR:
    
            printf("(");
    
            objectPrint(object->head);
    
            printf(", ");
    
            objectPrint(object->tail);
    
            printf(")");
    
            break;
    
        }
    
    }  
  
---|---  
  
* * *

### free、test、main

    
    
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

|

    
    
    void freeVM(VM  *vm) {
    
        vm->stackSize = 0;
    
        gc(vm);
    
        free(vm);
    
    }
    
    void test1() {
    
        printf("Test1: Objects on stack are preserved.n");
    
        VM* vm = newVM();
    
        pushInt(vm, 1);
    
        pushInt(vm, 2);
    
        gc(vm);
    
        assert(vm->numObjects == 2, "Should have preserved objects.");
    
        freeVM(vm);
    
    }
    
    void test2() {
    
        printf("Test2: Objects on stack are collected.n");
    
        VM* vm = newVM();
    
        pushInt(vm, 1);
    
        pushInt(vm, 2);
    
        pop(vm);
    
        pop(vm);
    
        gc(vm);
    
        assert(vm->numObjects == 0, "Should have collected objects.");
    
        freeVM(vm);
    
    }
    
    void test3() {
    
        printf("Test3: Reach nested objects.n");
    
        VM* vm = newVM();
    
        pushInt(vm, 1);
    
        pushInt(vm, 2);
    
        pushPair(vm);
    
        pushInt(vm, 3);
    
        pushInt(vm, 4);
    
        pushPair(vm);
    
        pushPair(vm);
    
        gc(vm);
    
        assert(vm->numObjects == 7, "Should have reached objects.");
    
        freeVM(vm);
    
    }
    
    void test4() {
    
        printf("Test4: Handle cycles.n");
    
        VM* vm = newVM();
    
        pushInt(vm, 1);
    
        pushInt(vm, 2);
    
        Object* a = pushPair(vm);
    
        pushInt(vm, 3);
    
        pushInt(vm, 4);
    
        Object* b = pushPair(vm);
    
        //建立一个循环， 并且让2 和 4 不可到达和收集
    
        a->tail = b;
    
        b->tail = a;
    
        gc(vm);
    
        assert(vm->numObjects == 4, "Should have collected objects.");
    
        freeVM(vm);
    
    }
    
    void perfTest() {
    
        printf("Performance Tsetn");
    
        VM* vm = newVM();
    
        for (int i = 0; i < 1000; i++) {
    
            for (int j = 0; j < 20; j++) {
    
                pushInt(vm, i);
    
            }
    
            for (int k = 0; k < 20; k++) {
    
                pop(vm);
    
            }
    
        }
    
        freeVM(vm);
    
    }
    
    int main(int argc, const char * argv[]) {
    
        test1();
    
        test2();
    
        test3();
    
        test4();
    
        perfTest();
    
        return 0;
    
    }  
  
---|---  
  
* * *

### 测试结果

### 总结

这个简单的收集器与目前 **Ruby** 和 **Lua** 中的收集器非常的相似，所以下一次我们将深入了解 **Lua** 的 **GC** 机制。