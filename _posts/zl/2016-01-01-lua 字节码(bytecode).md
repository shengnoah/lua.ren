---
layout: post
title: lua 字节码(bytecode) 
tags: [lua文章]
categories: [lua文章]
---
  * lua字节码
  * 字节码格式
    * 源码案例：
    * 对应字节码：
    * 指令解释：
  * 指令分类
  * 所有指令

## lua字节码

lua虚拟机最终执行的是经过lua编译器编译的字节码，这里暂不关系Chunk生成字节码过程， 只关系字节码本身，字节码的格式到底是什么样？具体的含义是什么？

## 字节码格式

lua字节码指令是由4个字节32位组成, 这时32是如何规划的，简单说那些位代表这个指令是
什么，那些位是操作数据，这里通过一个案例来看看bytecode结构，然后再解释bytecode具 体的结构。

### 源码案例：

    
    
    --test function 1
    function max(num1, num2)
       if (num1 > num2) then
    	  r = num1
       else
    	  r = num2
       end
       return r
    end
    --test function 2
    function add(num1, num2)
    	return num1 + num2
    end
    print("hello world")
    print("max ", max(10,8))
    

定义了两个函数，max和add函数，函数实现比较简单，暂且不关心，然后调用系统函数print 输出hello
world，接着调用max函数，add函数没有调用。看看这段代码被lua编译器编译后
的结果到底是什么？使用lua5.2.1版本lauc编译器，指令为luac -l test.lua进行编译。

### 对应字节码：

    
    
    main <test.lua:0,0> (15 instructions at 0000000000180370)
    0+ params, 5 slots, 1 upvalue, 0 locals, 7 constants, 2 functions
    	1	[9]	CLOSURE  	0 0	; 0000000000180640
    	2	[2]	SETTABUP 	0 -1 0	; _ENV "max"
    	3	[14]	CLOSURE  	0 1	; 00000000001809C0
    	4	[12]	SETTABUP 	0 -2 0	; _ENV "add"
    	5	[16]	GETTABUP 	0 0 -3	; _ENV "console"
    	6	[16]	LOADK    	1 -4	; "hello world"
    	7	[16]	CALL     	0 2 1
    	8	[17]	GETTABUP 	0 0 -3	; _ENV "console"
    	9	[17]	LOADK    	1 -5	; "max "
    	10	[17]	GETTABUP 	2 0 -1	; _ENV "max"
    	11	[17]	LOADK    	3 -6	; 10
    	12	[17]	LOADK    	4 -7	; 8
    	13	[17]	CALL     	2 3 0
    	14	[17]	CALL     	0 0 1
    	15	[17]	RETURN   	0 1
    function <test.lua:2,9> (8 instructions at 0000000000180640)
    2 params, 3 slots, 1 upvalue, 2 locals, 1 constant, 0 functions
    	1	[3]	LT       	0 1 0
    	2	[3]	JMP      	0 2	; to 5
    	3	[4]	SETTABUP 	0 -1 0	; _ENV "r"
    	4	[4]	JMP      	0 1	; to 6
    	5	[6]	SETTABUP 	0 -1 1	; _ENV "r"
    	6	[8]	GETTABUP 	2 0 -1	; _ENV "r"
    	7	[8]	RETURN   	2 2
    	8	[9]	RETURN   	0 1
    function <test.lua:12,14> (3 instructions at 00000000001809C0)
    2 params, 3 slots, 0 upvalues, 2 locals, 0 constants, 0 functions
    	1	[13]	ADD      	2 0 1
    	2	[13]	RETURN   	2 2
    	3	[14]	RETURN   	0 1
    

### 指令解释：

上面的源码生成指令可以看出来，每一行是一个指令，每一行指令由5部分组成，分别为： 指令行号 源码行号 指令 操作数 指令描述
通过上面的结果我们可以看出来，每一个lua函数，lua都会生成一段指令块，该指令块包含该 函数的内容指令。值得注意是lua源码会默认生成一个main
function，该指令块主要包含lua的 执行过程。

## 指令分类

四种指令：iABC iABx iAsBx iAx,代码中定义：enum OpMode {iABC, iABx, iAsBx, iAx};
lua所有指令前6位是操作码opcode,剩下组成部分如下：

    
    
    Instructions can have the following fields:
    	`A' : 8 bits
    	`B' : 9 bits
    	`C' : 9 bits
    	'Ax' : 26 bits ('A', 'B', and 'C' together)
    	`Bx' : 18 bits (`B' and `C' together)
    	`sBx' : signed Bx
    

## 所有指令

这里的指令是5.2.1版本里面所有的指令都定义在lopcode.h头文件中定义，代码如下：

    
    
    /*----------------------------------------------------------------------
    name		args	description
    ------------------------------------------------------------------------*/
    OP_MOVE,/*	A B	R(A) := R(B)					*/
    OP_LOADK,/*	A Bx	R(A) := Kst(Bx)					*/
    OP_LOADKX,/*	A 	R(A) := Kst(extra arg)				*/
    OP_LOADBOOL,/*	A B C	R(A) := (Bool)B; if (C) pc++			*/
    OP_LOADNIL,/*	A B	R(A), R(A+1), ..., R(A+B) := nil		*/
    OP_GETUPVAL,/*	A B	R(A) := UpValue[B]				*/
    
    OP_GETTABUP,/*	A B C	R(A) := UpValue[B][RK(C)]			*/
    OP_GETTABLE,/*	A B C	R(A) := R(B)[RK(C)]				*/
    
    OP_SETTABUP,/*	A B C	UpValue[A][RK(B)] := RK(C)			*/
    OP_SETUPVAL,/*	A B	UpValue[B] := R(A)				*/
    OP_SETTABLE,/*	A B C	R(A)[RK(B)] := RK(C)				*/
    
    OP_NEWTABLE,/*	A B C	R(A) := {} (size = B,C)				*/
    
    OP_SELF,/*	A B C	R(A+1) := R(B); R(A) := R(B)[RK(C)]		*/
    
    OP_ADD,/*	A B C	R(A) := RK(B) + RK(C)				*/
    OP_SUB,/*	A B C	R(A) := RK(B) - RK(C)				*/
    OP_MUL,/*	A B C	R(A) := RK(B) * RK(C)				*/
    OP_DIV,/*	A B C	R(A) := RK(B) / RK(C)				*/
    OP_MOD,/*	A B C	R(A) := RK(B) % RK(C)				*/
    OP_POW,/*	A B C	R(A) := RK(B) ^ RK(C)				*/
    OP_UNM,/*	A B	R(A) := -R(B)					*/
    OP_NOT,/*	A B	R(A) := not R(B)				*/
    OP_LEN,/*	A B	R(A) := length of R(B)				*/
    
    OP_CONCAT,/*	A B C	R(A) := R(B).. ... ..R(C)			*/
    
    OP_JMP,/*	A sBx	pc+=sBx; if (A) close all upvalues >= R(A) + 1	*/
    OP_EQ,/*	A B C	if ((RK(B) == RK(C)) ~= A) then pc++		*/
    OP_LT,/*	A B C	if ((RK(B) <  RK(C)) ~= A) then pc++		*/
    OP_LE,/*	A B C	if ((RK(B) <= RK(C)) ~= A) then pc++		*/
    
    OP_TEST,/*	A C	if not (R(A) <=> C) then pc++			*/
    OP_TESTSET,/*	A B C	if (R(B) <=> C) then R(A) := R(B) else pc++	*/
    
    OP_CALL,/*	A B C	R(A), ... ,R(A+C-2) := R(A)(R(A+1), ... ,R(A+B-1)) */
    OP_TAILCALL,/*	A B C	return R(A)(R(A+1), ... ,R(A+B-1))		*/
    OP_RETURN,/*	A B	return R(A), ... ,R(A+B-2)	(see note)	*/
    
    OP_FORLOOP,/*	A sBx	R(A)+=R(A+2);
    			if R(A) <?= R(A+1) then { pc+=sBx; R(A+3)=R(A) }*/
    OP_FORPREP,/*	A sBx	R(A)-=R(A+2); pc+=sBx				*/
    
    OP_TFORCALL,/*	A C	R(A+3), ... ,R(A+2+C) := R(A)(R(A+1), R(A+2));	*/
    OP_TFORLOOP,/*	A sBx	if R(A+1) ~= nil then { R(A)=R(A+1); pc += sBx }*/
    
    OP_SETLIST,/*	A B C	R(A)[(C-1)*FPF+i] := R(A+i), 1 <= i <= B	*/
    
    OP_CLOSURE,/*	A Bx	R(A) := closure(KPROTO[Bx])			*/
    
    OP_VARARG,/*	A B	R(A), R(A+1), ..., R(A+B-2) = vararg		*/
    
    OP_EXTRAARG/*	Ax	extra (larger) argument for previous opcode	*/