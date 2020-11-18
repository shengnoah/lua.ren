---
layout: post
title: lua函数与upavalue 
tags: [lua文章]
categories: [lua文章]
---
Lua中的所谓upvalue是一类比较特殊的值，可以理解为在某函数内引用外部的数据。

函数定义中用到了pushclosure这个函数用于将函数的定义Proto结构体指针push到父函数的数组中，在这个函数内，还建立了有关upvalue相关的逻辑：

    
    
    static void pushclosure (LexState *ls, FuncState *func, expdesc *v) {
      FuncState *fs = ls->fs;
      Proto *f = fs->f;
      int oldsize = f->sizep;
      int i;
      luaM_growvector(ls->L, f->p, fs->np, f->sizep, Proto *,
                      MAXARG_Bx, "constant table overflow");
      while (oldsize < f->sizep) f->p[oldsize++] = NULL;
      f->p[fs->np++] = func->f;
      luaC_objbarrier(ls->L, f, func->f);
      init_exp(v, VRELOCABLE, luaK_codeABx(fs, OP_CLOSURE, 0, fs->np-1));
      for (i=0; if->nups; i++) {
        OpCode o = (func->upvalues[i].k == VLOCAL) ? OP_MOVE : OP_GETUPVAL;
        luaK_codeABC(fs, o, 0, func->upvalues[i].info, 0);
      }
    }
    

注意在这个函数的最后，将遍历upvalue数组，根据该upvalue是否是局部变量，来决定紧跟着的是MOVE指令还是GETUPVAL指令。而这些是如何确定的呢？

Lua的分析器在解析到一个变量时，会调用singlevaraux函数进行查找：

    
    
    static int singlevaraux (FuncState *fs, TString *n, expdesc *var, int base) {
      if (fs == NULL) {  /* no more levels? */
        init_exp(var, VGLOBAL, NO_REG);  /* default is global variable */
        return VGLOBAL;
      }
      else {
        int v = searchvar(fs, n);  /* look up at current level */
        if (v >= 0) {
          init_exp(var, VLOCAL, v);
          if (!base)
            markupval(fs, v);  /* local will be used as an upval */
          return VLOCAL;
        }
        else {  /* not found at current level; try upper one */
          if (singlevaraux(fs->prev, n, var, 0) == VGLOBAL)
            return VGLOBAL;
          var->u.s.info = indexupvalue(fs, n, var);  /* else was LOCAL or UPVAL */
          var->k = VUPVAL;  /* upvalue in this level */
          return VUPVAL;
        }
      }
    }
    

可以看到，这个函数是一个递归函数，有以下几种情况：

  1. 在函数的当前层找到该变量，则认为一个LOCAL变量
  2. 在函数的上层找到，则认为一个UPVAL
  3. 最后，则认为是一个全局变量。

如何定义函数的“层次”？来看一个例子就知道了:

    
    
    local a = 1
    function test1()
      local b = 100
      function test2()
         print(a)
         print(b)
      end
    end
    test1()
    

在这个例子中，函数test2与变量b是同层的，所以在调用函数test2时，singlevaraux查找变量b返回的LOCAL变量；而变量a是更上一层的LOCAL变量，对于函数test2而言，它就是UPVAL。

明白了解析部分是怎么处理upvalue的，来看看在虚拟机中是如何处理的。  
对应的代码在lvm.c中的这一部分：

    
    
          case OP_CLOSURE: {
            Proto *p;
            Closure *ncl;
            int nup, j;
            p = cl->p->p[GETARG_Bx(i)];
            nup = p->nups;
            ncl = luaF_newLclosure(L, nup, cl->env);
            ncl->l.p = p;
            for (j=0; jl.upvals[j] = cl->upvals[GETARG_B(*pc)];
              else {
                lua_assert(GET_OPCODE(*pc) == OP_MOVE);
                ncl->l.upvals[j] = luaF_findupval(L, base + GETARG_B(*pc));
              }
            }
            setclvalue(L, ra, ncl);
            Protect(luaC_checkGC(L));
            continue;
          }
    

当变量是UPVAL时，此时PC指令对应的B参数是函数结构体的upval数组的索引，根据它直接从upval数组中取出值来；否则，PC指令对应的B参数是基于函数基地址base的一个偏移量，根据它得到相应的变量；再调用函数luaF_findupval：

    
    
    UpVal *luaF_findupval (lua_State *L, StkId level) {
      global_State *g = G(L);
      GCObject **pp = &L->openupval;
      UpVal *p;
      UpVal *uv;
      while (*pp != NULL && (p = ngcotouv(*pp))->v >= level) {
        lua_assert(p->v != &p->u.value);
        if (p->v == level) {  /* found a corresponding upvalue? */
          if (isdead(g, obj2gco(p)))  /* is it dead? */
            changewhite(obj2gco(p));  /* ressurect it */
          return p;
        }
        pp = &p->next;
      }
      uv = luaM_new(L, UpVal);  /* not found: create a new one */
      uv->tt = LUA_TUPVAL;
      uv->marked = luaC_white(g);
      uv->v = level;  /* current value lives in the stack */
      uv->next = *pp;  /* chain it in the proper position */
      *pp = obj2gco(uv);
      uv->u.l.prev = &g->uvhead;  /* double link it in `uvhead' list */
      uv->u.l.next = g->uvhead.u.l.next;
      uv->u.l.next->u.l.prev = uv;
      g->uvhead.u.l.next = uv;
      lua_assert(uv->u.l.next->u.l.prev == uv && uv->u.l.prev->u.l.next == uv);
      return uv;
    }
    

注意到，传入这个函数的参数level，其实是前面已经根据base基址定位到的变量。这个函数分为两个部分：

  1. 首先，遍历当前的openupval数组，查找这个变量。由于这个变量肯定是前面已经定义过的，所以查找的条件就是（(p = ngcotouv(*pp))->v >= level）。当查找到这个变量时，如果是准备释放的变量，则将它重新置为不可释放。
  2. 如果在openval数组中没有找到，说明之前没有别的地方引用过这个upval。如此则重新分配一个upvalue指向待引用的值。
  3. 最后，当函数调用完毕时，有相应的close指令，将upvalue的引用关系去除。具体见函数luaF_close。