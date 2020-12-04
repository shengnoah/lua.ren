---
layout: post
title: linux常用库的lua移植：libmcrypt的lua插件 
tags: [lua文章]
categories: [topic]
---
<p>openresty的lua方案效率不错，缺点是库不全，php现成的接口应用改成lua并不容易，比如libmcrypt，这个php常用的加密库（其实nginx的lua模块加密还有个不错的原生方案，openssl，在openrestylualibresty目录下，特定的加密方式需要自行修改），第三方C库在lua里使用都可以依照本文的方法来处理。<br/>        下载libmcrypt的源代码，解压，找到正常libmcrypt库的入口C文件（源代码目录下，找int main()），这个目录libmcrypt-2.5.3src里面，找到这几个文件：cipher_test.c和aes_test.c，移植就从修改这两个文件的开始：<br/>         </p>
<pre><code>int main()  
{  
MCRYPT td;//设定IV向量（需要的话）  
unsigned char IV[8]={0x12,0x34,0x56,0x78,0x90,0xab,0xcd,0xef};  
unsigned char key[8]={0x12,0x34,0x56,0x78,0x90,0xab,0xcd,0xef};  
int keysize;  
names = mcrypt_list_algorithms (ALGORITHMS_DIR, &amp;jmax);  
modes = mcrypt_list_modes (MODES_DIR, &amp;imax);//设定加密模式  
for (i=0;i&lt;imax;i++) {  
if(strcmp(modes[i],”cbc”)==0){  
td = mcrypt_module_open(names[j], ALGORITHMS_DIR, modes[i], MODES_DIR);//初始化  
ivsize = mcrypt_enc_get_iv_size(td);  
mcrypt_generic_init( td, key, keysize, IV) ;  
if (mcrypt_enc_is_block_mode(td)!=0)  
siz = (strlen(TEXT) / mcrypt_enc_get_block_size(td))*mcrypt_enc_get_block_size(td);  
else siz = strlen(TEXT);//开辟存储结果的内存  
text = calloc( 1, siz);  
memmove( text, TEXT, siz);  
//产生结果  
mdecrypt_generic( td, text, siz);  
memcmp( text, TEXT, siz);  
mcrypt_generic_deinit(td);  
mcrypt_module_close(td);  
free(text);  
free(key);  
free(IV);  
return;  
}    
</code></pre><p>保存，编译，gcc编译选项记得加lpthread之类的，运行 . 到这步，移植的逻辑被证明是正确的，接下来想办法把它做成一个弓lua调用的so文件。</p>
<pre><code>#include “../lib/mcrypt.h”   //使用libmcrypt要加的头文件
#include “lua.h”
#include “lauxlib.h”
#include “lualib.h”          //lua和C、C++交互需要的几个头文件
unsigned char *enc(unsigned char *data);
unsigned char *dec(unsigned char *data);   //这几个封装上面加密和解密的逻辑，一个加密，一个解密

接下来想办法把这两个函数抛给lua：

static int encode(lua_State* L)
{
const char *src=lua_tostring(L,-1);
unsigned char *s=(unsigned char *)malloc(lua_strlen(L,-1)+1);
strcpy(s,src);
s[lua_strlen(L,-1)]=(unsigned char)0x0;
unsigned char *rst=enc(s);   //在这里调用enc()加密函数
lua_pushstring(L,rst);
free(s);
free(rst);
return 1;
}

static int decode(lua_State* L)
{
const char *src=lua_tostring(L,-1);
unsigned char*s=(unsigned char *)malloc(lua_strlen(L,-1)+1);
strcpy(s,src);
s[lua_strlen(L,-1)]=(unsigned char)0x0;
unsigned char *rst=dec(s);      //在这里调用dec()解密函数
lua_pushstring(L,rst);
free(s);
free(rst);
return 1;
}

//让so文件注册这两个函数
static luaL_Reg mylibs[] = {
{“encode”, encode},
{“decode”, decode},
{NULL, NULL}
};

//我们库的名字叫做libdes4lua.so,最终这么写，相当于动态链接库的main
int luaopen_des4lua(lua_State* L)
{
const char* libName = “des4lua”;
luaL_register(L,libName,mylibs);  //注册，透给lua脚本
return 1;
}
</code></pre><p>编译，得到libdes4lua.so,将这个文件拷贝到openresty/lualib,openresty的lua加密库就做好了.</p>