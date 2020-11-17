
---
layout: post
title: fblualib在ubuntu 16.04上的安装 
tags: [lua文章]
categories: [lua文章]
---
最近需要复现一下[CRNN](https://github.com/bgshih/crnn)的结果，因为自己实现的版本点数不够高，所以需要跑一下人家的代码。一共有两个依赖，一个是TORCH，这个好装，另一个是fblualib，这个项目已经停止维护了，真的是跳了无数个坑，这里做一个记录。最后也只能跑inference不能train
BTW. 但是对于其他跳坑的人可能也是有帮助的.  
  
  

  * 概览
  * torch
  * cudnn.torch
  * googletest
  * folly
  * mstch
  * zstd
  * googletest
  * wangle
  * fbthrift
  * thpp
  * fblualib
  * CRNN
  * 各种报错的解决办法

### 概览

首先是需要安装的包  
[fblualib](https://github.com/facebookarchive/fblualib/blob/master/install_all.sh),[folly](https://github.com/facebook/folly/),[mstch](https://github.com/no1msd/mstch),[zstd](https://github.com/facebook/zstd),[wangle](https://github.com/facebook/wangle),[thpp](https://github.com/facebook/thpp),[fbthrift](https://github.com/facebook/fbthrift/)  
其次：  
system：ubuntu 16.04，相比来说14.04应该更好装  
python:2.7，之前试了python3，但是在import frontend的时候遇到了一些问题，没有解决好，就放弃了，其实是有希望的。  

### torch

commit 0219027e6c4644a0ba5c5bf137c989a0a8c9e01b

### cudnn.torch

    
    
    1  
    

|

    
    
    git clone https://github.com/soumith/cudnn.torch  
      
  
---|---  
  
### googletest

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
    
    wget https://github.com/google/googletest/archive/release-1.8.0.tar.gz &&   
    tar zxf release-1.8.0.tar.gz &&   
    rm -f release-1.8.0.tar.gz &&   
    cd googletest-release-1.8.0 &&   
    cmake configure . &&   
    make &&   
    make install  
      
  
---|---  
  
### folly

commit ID:`5817bad`  
依赖：  

    
    
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
    

|

    
    
    sudo apt-get install   
        g++   
        cmake   
        libboost-all-dev   
        libevent-dev   
        libdouble-conversion-dev   
        libgoogle-glog-dev   
        libgflags-dev   
        libiberty-dev   
        liblz4-dev   
        liblzma-dev   
        libsnappy-dev   
        make   
        zlib1g-dev   
        binutils-dev   
        libjemalloc-dev   
        libssl-dev   
        pkg-config  
        libunwind8-dev   
        libelf-dev   
        libdwarf-dev  
      
  
---|---  
      
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
    
    git clone https://github.com/facebook/folly/  
    cd folly/folly  
    autoreconf -ivf  
    ./configure &&  
    make &&  
    sudo make install &&  
    sudo ldconfig &&  
      
  
---|---  
  
### mstch

commit ID:`0fde1cf`  

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
    
    git clone https://github.com/no1msd/mstch  
    cd mstch  
    mkdir build  
    cd build  
    cmake ..  
    make -j 4  
    sudo make install  
      
  
---|---  
  
### zstd

commit ID:`87125c2` dev branch  

    
    
    1  
    2  
    3  
    

|

    
    
    git clone https://github.com/facebook/zstd  
    cd zstd  
    sudo make install  
      
  
---|---  
  
### googletest

    
    
    1  
    2  
    

|

    
    
    git clone https://github.com/google/googletest  
    cmake make install  
      
  
---|---  
  
### wangle

commit ID:`31fbaba  

    
    
    1  
    2  
    3  
    4  
    5  
    

|

    
    
    git clone https://github.com/facebook/wangle  
    cmake .  
    make  
    ctest  
    sudo make install  
      
  
---|---  
  
### fbthrift

commit ID:`94a399a`  
更改的部分  

    
    
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

    
    
    diff --git a/CMakeLists.txt b/CMakeLists.txt  
    index 51271e3..7a15eae 100644  
    --- a/CMakeLists.txt  
    +++ b/CMakeLists.txt  
    @@ -8,6 +8,9 @@ set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)  
     set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)  
     set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)  
      
    +set(CMAKE_CXX_FLAGS "-fPIC")  
    +set(CMAKE_C_FLAGS "-fPIC")  
    +  
     set(TEMPLATES_INSTALL_DIR include/thrift/templates CACHE STRING  
         "The subdirectory where compiler template files should be installed")  
     set(BIN_INSTALL_DIR bin CACHE STRING  
    主要为了解决python包 thrift-compiler无法生成的问题  
      
  
---|---  
      
    
    1  
    2  
    3  
    

|

    
    
    cd build  
    cmake ..  
    make -j 32  
      
  
---|---  
  
### thpp

commit ID:`0a4c745`  

    
    
    1  
    2  
    3  
    

|

    
    
    git clone https://github.com/facebook/thpp  
    cd thpp/thpp  
    ./build.sh  
      
  
---|---  
  
更改的部分  

    
    
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
    

|

    
    
    diff --git a/thpp/CMakeLists.txt b/thpp/CMakeLists.txt  
    index 4ae3683..603702b 100644  
    --- a/thpp/CMakeLists.txt  
    +++ b/thpp/CMakeLists.txt  
    @@ -42,7 +42,7 @@ ELSE()  
       ADD_DEFINITIONS(-DNO_THRIFT)  
     ENDIF()  
      
    -SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=gnu++11")  
    +SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=gnu++14")  
      
     SET(src  
       Storage.cpp  
    ------------------------------------------------------  
       diff --git a/thpp/build.sh b/thpp/build.sh  
       index 79af5d9..7c807e7 100755  
       --- a/thpp/build.sh  
       +++ b/thpp/build.sh  
       @@ -52,4 +52,4 @@ make  
        ctest  
      
        # Install  
       -make install  
       +sudo make install  
    ------------------------------------------------------  
    diff --git a/thpp/detail/TensorGeneric.h b/thpp/detail/TensorGeneric.h  
    index a8837f6..6b67c72 100644  
    --- a/thpp/detail/TensorGeneric.h  
    +++ b/thpp/detail/TensorGeneric.h  
    @@ -188,17 +188,17 @@ template <> struct TensorOps<Tensor<real>> {  
       }  
       static void _max(THTensor* values, THLongTensor* indices,  
                        THTensor* t, int dim) {  
    -    return THTensor_(max)(values, indices, t, dim);  
    +    return THTensor_(max)(values, indices, t, dim, 1);  
       }  
       static void _min(THTensor* values, THLongTensor* indices,  
                        THTensor* t, int dim) {  
    -    return THTensor_(min)(values, indices, t, dim);  
    +    return THTensor_(min)(values, indices, t, dim, 1);  
       }  
       static void _sum(THTensor* r, THTensor* t, int dim) {  
    -    return THTensor_(sum)(r, t, dim);  
    +    return THTensor_(sum)(r, t, dim, 1);  
       }  
       static void _prod(THTensor* r, THTensor* t, int dim) {  
    -    return THTensor_(prod)(r, t, dim);  
    +    return THTensor_(prod)(r, t, dim, 1);  
       }  
       static void _cumsum(THTensor* r, THTensor* t, int dim) {  
         return THTensor_(cumsum)(r, t, dim);  
      
  
---|---  
  
### fblualib

commit ID:`4812779`  
更改的部分，这个改的太多了  

    
    
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
    

|

    
    
    modified:   fblualib/build.sh  
    modified:   fblualib/ffivector/CMakeLists.txt  
    modified:   fblualib/ffivector/FFIVector.cpp  
    modified:   fblualib/mattorch/CMakeLists.txt  
    modified:   fblualib/python/CMakeLists.txt  
    modified:   fblualib/thrift/ChunkedCompression.h  
    modified:   fblualib/thrift/Encoding.h  
    modified:   fblualib/thrift/LuaObject.h  
    modified:   fblualib/thrift/LuaSerialization.cpp  
    modified:   fblualib/torch/CMakeLists.txt  
    modified:   fblualib/util/Reactor.cpp  
      
  
---|---  
  
### CRNN

ptr的问题

### 各种报错的解决办法

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    

|

    
    
    Could not find a package configuration file provided by "Torch" with any of  
      the following names:  
      
        TorchConfig.cmake  
        torch-config.cmake  
      
    export CMAKE_PREFIX_PATH=/path/to/torch  
      
  
---|---

