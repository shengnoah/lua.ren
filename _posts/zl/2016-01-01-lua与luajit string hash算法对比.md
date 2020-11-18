---
layout: post
title: lua与luajit string hash算法对比 
tags: [lua文章]
categories: [lua文章]
---
## 说明

研究lua与luajit两种string hash算法的原理，以及性能对比

## string hash

### lua

  1. 如果string长度小于32，则会对所有字符串的字符进行遍历，然后计算hash值
  2. 如果string长度大于等于32，则把string进行等分，等分间隔为长度/32+1，然后取每等分的第一个字符进行hash值计算。所以不管字符串多长，最多会循环32次

#### 代码

    
    
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

    
    
      
    unsigned int  (const char *str, size_t l, unsigned int seed) {  
      unsigned int h = seed ^ cast(unsigned int, l);  
      size_t l1;  
      size_t step = (l >> LUAI_HASHLIMIT) + 1;  // LUAI_HASHLIMIT值为5，l是字符串长度，不是一  
      for (l1 = l; l1 >= step; l1 -= step)  
        h = h ^ ((h<<5) + (h>>2) + cast_byte(str[l1 - 1]));  
      return h;  
    }  
      
  
---|---  
  
### luajit

  1. 如果string小于4，则会取字符串的第一位，最后一位，以及中间位，进行hash值计算
  2. 如果string长度大于等于4，则会取字符串第1~4位，最后4位，len/2-2~len/2+2，len/4-1~len/4+3，进行hash计算

#### 代码

    
    
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
    

|

    
    
    // 代码我整理了封装了下，用来展示和测试  
      
    #define lj_rol(x, n)	(((x)<<(n)) | ((x)>>(-(int)(n)&(8*sizeof(x)-1))))  
      
    int luajit_hash(const char *str, size_t len)  
    {  
    	unsigned int a, b, h = len;  
    	if (len >= 4) {  /* Caveat: unaligned access! */  
    		a = lj_getu32(str);  
    		h ^= lj_getu32(str + len - 4);  
    		b = lj_getu32(str + (len >> 1) - 2);  
    		h ^= b; h -= lj_rol(b, 14);  
    		b += lj_getu32(str + (len >> 2) - 1);  
    	}  
    	else if (len > 0) {  
    		a = *(const unsigned char *)str;  
    		h ^= *(const unsigned char *)(str + len - 1);  
    		b = *(const unsigned char *)(str + (len >> 1));  
    		h ^= b; h -= lj_rol(b, 14);  
    	}  
    	a ^= h; a -= lj_rol(h, 11);  
    	b ^= a; b -= lj_rol(a, 25);  
    	h ^= b; h -= lj_rol(b, 16);  
    	return h;  
    }  
      
  
---|---  
  
### 分析

从采样随机性及次数上来看，肯定是lua原生自带的hash算法好。但也因为采样次数增加，导致原生hash算法的性能会有损失

## 性能对比

对lua以及luajit的string hash算法，进行不同字符串长度的性能对比

### 测试环境

操作系统：Debian GNU/Linux 8  
CPU：Intel(R) Xeon(R) CPU E5-2640 v2 @ 2.00GHz  
内存：64G

### 测试代码

    
    
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
    

|

    
    
    #include <string>  
    #include <time.h>  
    #include <stdlib.h>  
    #include <stdio.h>  
      
    static const int lua_hash_seed = 0;  
      
    int lua_hash(const char *str, size_t len)  
    {  
    	unsigned int h = lua_hash_seed ^ (unsigned int)len;  
    	size_t step = (len >> 5) + 1;  
    	for (; len >= step; len -= step)  
    		h ^= ((h << 5) + (h >> 2) + (unsigned char)(str[len - 1]));  
    	return h;  
    }  
      
      
    #define lj_rol(x, n)	(((x)<<(n)) | ((x)>>(-(int)(n)&(8*sizeof(x)-1))))  
      
    int luajit_hash(const char *str, size_t len)  
    {  
    	unsigned int a, b, h = len;  
    	if (len >= 4) {  /* Caveat: unaligned access! */  
    		a = lj_getu32(str);  
    		h ^= lj_getu32(str + len - 4);  
    		b = lj_getu32(str + (len >> 1) - 2);  
    		h ^= b; h -= lj_rol(b, 14);  
    		b += lj_getu32(str + (len >> 2) - 1);  
    	}  
    	else if (len > 0) {  
    		a = *(const unsigned char *)str;  
    		h ^= *(const unsigned char *)(str + len - 1);  
    		b = *(const unsigned char *)(str + (len >> 1));  
    		h ^= b; h -= lj_rol(b, 14);  
    	}  
    	a ^= h; a -= lj_rol(h, 11);  
    	b ^= a; b -= lj_rol(a, 25);  
    	h ^= b; h -= lj_rol(b, 16);  
    	return h;  
    }  
      
    void random_string(std::string &str, size_t size)  
    {  
    	for (int i = 0; i < size; ++i)  
    	{  
    		str.push_back((char)(rand() % 255));  
    	}  
    }  
      
    #define HASH_COUNT 100000  
    int a[HASH_COUNT];  
    int main()  
    {  
    	std::string str;  
      
    	srand(clock());  
      
    	int step = 1;  
    	for (int len = 1; len <= 65535; len += step)  
    	{  
    		printf("string len = %d:n", len);  
      
    		random_string(str, step);  
    		  
    		{  
    			clock_t start = clock();  
    			for (int i = 0; i < HASH_COUNT; ++i)  
    			{  
    				a[i] = luajit_hash(str.c_str(), str.size());	// 防止编译器优化;  
    			}  
    			clock_t use_time = clock() - start;  
      
    			printf("tluajit use: %lfn", use_time*1.0 / CLOCKS_PER_SEC);  
    		}  
    		  
    		{  
    			clock_t start = clock();  
    			for (int i = 0; i < HASH_COUNT; ++i)  
    			{  
    				a[i] = lua_hash(str.c_str(), str.size());  
    			}  
    			clock_t use_time = clock() - start;  
      
    			printf("tlua use: %lfn", use_time*1.0 / CLOCKS_PER_SEC);  
    		}  
    	}  
      
    	return 0;  
    }  
      
  
---|---  
  
### 测试结果

    
    
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
    

|

    
    
    string len = 1:  
        luajit use: 0.000615  
        lua use: 0.000205  
    string len = 2:  
        luajit use: 0.000410  
        lua use: 0.000352  
    string len = 3:  
        luajit use: 0.000426  
        lua use: 0.000505  
    string len = 4:  
        luajit use: 0.000485  
        lua use: 0.000552  
    string len = 5:  
        luajit use: 0.000531  
        lua use: 0.000803  
    string len = 6:  
        luajit use: 0.000455  
        lua use: 0.000910  
    string len = 7:  
        luajit use: 0.000479  
        lua use: 0.001106  
    ...  
    string len = 31:  
        luajit use: 0.000483  
        lua use: 0.005269  
    string len = 32:  
        luajit use: 0.000506  
        lua use: 0.002439  
    ...  
      
  
---|---  
  
  * 可以看到，除了1，2长度的字符串，lua原生hash快，其他都是luajit快。  
漫的速度大致可以通过lua原生hash的循环次数大致估算，如果循环满32次，则性能大致是luajit的1/10倍，如果为16次则为1/5。

## 结论

  1. lua原生的hash算法会导致性能的开销，测试最慢为0.005s/10W次，而luajit为0.0004s/10W次
  2. 性能开销带来的同时，从算法层面看到，lua原生的hash算法会跟好，不过这还得需要在实际使用当中进行测试才能得到更准确得结论