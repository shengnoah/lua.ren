---
layout: post
title: ubuntu安装wireshark报错lua 
tags: [lua文章]
categories: [topic]
---
ubuntu: 16.04

打开终端，安装wireshark过程如下：  

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6

|

    
    
    # Add the stable official PPA.
    
    sudo add-apt-repository ppa:wireshark-dev/stable
    
    # Update the repository
    
    sudo apt-get update
    
    # Install wireshark
    
    sudo apt-get install wireshark  
  
---|---  
  
我的在安装过程中卡住了一会，提示`51% [正在等待报头]`，等了好一会，才继续了。

安装完成后，运行`sudo wireshark`,就可以打开界面，但是会报错：

![img](https://huanyouchen-1252081928.cos.ap-
shanghai.myqcloud.com/2018-05-08%2020-19-00%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png?imageView2/0/q/75|watermark/2/text/aHVhbnlvdWNoZW4uZ2l0aHViLmlv/font/5qW35L2T/fontsize/320/fill/IzBBMEEwQQ==/dissolve/93/gravity/SouthEast/dx/10/dy/10|imageslim)

网上查了一下，解决方法如下：  

    
    
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

    
    
    # fix lua error
    
    cd /usr/share/wireshark
    
    sudo gedit init.lua
    
    # 在打开的init.lua中，找到这一行，改为true-- Set disable_lua to true to disable Lua support.
    
    disable_lua = true
    
    # 然后在终端里：
    
    sudo dpkg-reconfigure wireshark-common
    
    # 选择是(yes)，再然后：
    
    sudo adduser $USER wireshark  
  
---|---  
  
然后重启电脑，再次打开终端，运行`wireshark`，就没有报错了。

方法来源：  
<https://askubuntu.com/questions/700712/how-to-install-wireshark>