---
layout: post
title: lua data struct 
tags: [lua文章]
categories: [topic]
---
<p>TValue是lua中的通用数据结构，lua中的所有数据都可以使用这个结构表示。下面看一下与TValue定义相关的数据结构：</p>

<pre><code>/*
** {======================================================
** types and prototypes
** =======================================================
*/
union Value {
  		GCObject *gc;    /* collectable objects */
  		void *p;         /* light userdata */
  		int b;           /* booleans */
  		lua_CFunction f; /* light C functions */
  		lua_Integer i;   /* integer numbers */
  		lua_Number n;    /* float numbers */
};

/*
** Union of all Lua values
*/
typedef union Value Value;

/*
** Tagged Values. This is the basic representation of values in Lua,
** an actual value plus a tag with its type.
*/
#define TValuefields	Value value_; int tt_

struct lua_TValue {
  		TValuefields;
};

typedef struct lua_TValue TValue;

/*
** Common type for all collectable objects
*/
typedef struct GCObject GCObject;


/*
** Common Header for all collectable objects (in macro form, to be
** included in other objects)
*/
#define CommonHeader	GCObject *next; lu_byte tt; lu_byte marked


/*
** Common type has only the common header
*/
struct GCObject {
 		CommonHeader;
};
</code></pre>

<p>下面把TValue结构中相关的宏展开：</p>

<pre><code>typedef struct lua_TValue {
	union {
		struct GCObject {
			GCObject *next;
			lu_byte tt;
			lu_byte marked;
		} *gc;
		void *p;
		int b;
		lua_CFunction f;
		lua_Integer i;
		lua_Number n;
	} value_;
	int tt_;
} TValue;
</code></pre>

<p>这个结构可以分为两部分：一部分tt_用来标识数据类型，而另一部分value_用来存放相关的数据。联合体gc部分用来存放可gc对象的gc相关的信息：next指针将可gc对象连成链表，tt表示数据类型，marked表示gc处理时的颜色值。从源码时可以看出各种可gc的对象的结构都会包含CommonHeader这个头信息来处理gc相关的信息。</p>

<p>lua中stack相关的信息被表示为TValue的指针，如果：</p>

<pre><code>typedef TValue *StkId;  /* index to stack elements */
</code></pre>

<p>下面是字符串结构：</p>

<pre><code>/*
** Header for string value; string bytes follow the end of this structure
** (aligned according to &#39;UTString&#39;; see next).
*/
typedef struct TString {
  		CommonHeader;
  		lu_byte extra;  /* reserved words for short strings; &#34;has hash&#34; for longs */
  		lu_byte shrlen;  /* length for short strings */
  		unsigned int hash;
  		union {
		size_t lnglen;  /* length for long strings */
		struct TString *hnext;  /* linked list for hash table */
  		} u;
} TString;

/*
** Ensures that address after this type is always fully aligned.
*/
typedef union UTString {
  		L_Umaxalign dummy;  /* ensures maximum alignment for strings */
  		TString tsv;
} UTString;
</code></pre>

<p>字符串一旦创建，则不可被改写。lua的值对象若这字符串类型，则以引用方式存在。属于被gc管理的对象。当一个字符串没有任何地方引用就可以回收它。
除了用于gc的CommonHeader外，还有extra shrlen hash和u域。extra对于短字符串来说用于记录这个字符串是否为保留字，这个标识用于词法分析器对保留字的快速判断，对于长字符串存hash值。
hash记录字符串的hash值用来加快字符串的相关操作。shrlen用于记录短字符串的长度。u为联合体其中lnglen记录长字符串的长度，hnext为链表指针。
字符串的实际值并没有分配独立的内在来保存，而是直接加在UTString后面。用下面的宏就可以取到实际的C字符串指针：</p>

<pre><code>/*
** Get the actual string (array of bytes) from a &#39;TString&#39;.
** (Access to &#39;extra&#39; ensures that value is really a &#39;TString&#39;.)
*/
#define getaddrstr(ts)	(cast(char *, (ts)) + sizeof(UTString))
#define getstr(ts)  
  		check_exp(sizeof((ts)-&gt;extra), cast(const char*, getaddrstr(ts)))
</code></pre>

<p>所有的短字符串被存放在全局状态机(global_State)的strt域中，strt的数据类型如下：</p>

<pre><code>typedef struct stringtable {
  		TString **hash;
  		int nuse;  /* number of elements */
  		int size;
} stringtable;
</code></pre>

<p>相同的短字符串在同一个lua state中存在唯一一份，这样不仅能减小内在而且还提高了比较相关的操作的效率。
长字符串则独立存在，从外部压入一个长字符串时，简单的复制一遍字符串。但不立即计算其hash值，推迟到对字符串作匹配时。</p>

<p>Userdata在lua中并没有特别之处，在储存形式上和字符串相同。可以看成是拥有独立的元表，不被内部化处理，也不需要追加