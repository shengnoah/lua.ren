---
layout: post
title: 莱迪思产品ICEstick Evaluation Kit 简介 
tags: [lua文章]
categories: [lua文章]
---
本文在CSDN的地址：[点这里](https://blog.csdn.net/idevede/article/details/61930997)

USB驱动、拇指大小的评估板 —— iCEstick评估套件是一款易于使用、小体积的评估板，通过使用板上莱迪思半导体公司的iCE40
FPGA系列，您可以以极低的成本快速实现系统功能的开发。

##摘自官网##

IrDA和Digilent PMOD™ 接口 —— 用户可以与一个IrDA收发器进行通信，一个Digilent
PMOD™外设连接器可用于许多传感器扩展功能，还有16个通用I/O和LED。

##产品图片##  
![ICEstick Evaluation Kit
正视图](http://img.blog.csdn.net/20170313205246231?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaWRldmVkZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

##各部件介绍##  
![ICEstick Evaluation Kit
框图](http://img.blog.csdn.net/20170313205353408?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaWRldmVkZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

###1.FT2232H###  
FT2232H是USB／RS232双向转换器，支持480 Mb/s的USB 2．0高速规范，提供2个支持USB
2．0高速规范且可配置的并行／串行接口，并且内部集成有USB协议，无须编写USB固件程序。  
引脚布局如下：  
![FT2232H
引脚布局](http://img.blog.csdn.net/20170313205719380?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaWRldmVkZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

###2.LDO–LT3030###  
LT3030是一款具备 1.8V 至 20V 的宽输入电压范围，提供低至 1.215V 和高达 19.5V 的输出电压的低压差稳压器

###3.93LC56###  
一款存储芯片。cmos serial eeprom（串行电可擦除只读存储器）  
框图如下：  
![93LC56内部电路图](http://img.blog.csdn.net/20170313211934660?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaWRldmVkZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  
管脚图如下：  
![93LC56管脚图](http://img.blog.csdn.net/20170313212009489?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaWRldmVkZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

###4.SPI–N25Q32 SPI闪存###  
串行闪存是一种尺寸和功耗都很小的采用SPI(串行外设接口)总线的 NOR 闪存芯片. SPI
又叫”四线”串行总线,用于顺序数据缓存器.当被嵌入式系统用于代码或参数存储器时,串行闪存在印制电路板上需要的连线数量比并行闪存少,因为串行闪存是把数据串
行化,所需的输入/输出引脚比较少,每个时钟周期只传送一位数据.这些特性有助于降
低电路板空间,功耗和系统总体成本,这就是串行闪存在嵌入式系统设计社区不断升温的原因.

常见用途：

  1. 代码存储  
代码存储分大两大类：  
1.1. 标准性能:从外部闪存执行代码(XIP)  
对于没有严格的时间限制的应用，控制器可以直接从串行闪存执行代码，不过存取操作的时间较长。但是，在执行代码时，如果经常出现地址跳转命令，那么最好还是使用能够同时发送地址位和数据位的并行闪存，以改进数据传输时间(>40ns)。并行通信需要的引脚数量多，因此封装尺寸也就相对较大。  
1.2. **_高性能:从RAM内存执行代码(代码映射技术)_**  
很多应用对性能要求很高，因此不能直接从闪存执行代码，只能从存取时间较短的RAM内存执行程序(>5ns)。因为RAM是易失性存储器，这些应用还需要一个非易失性存储器(闪存)在断电时保存代码，每次应用系统上电时还要把代码下载到RAM，这种方法叫做代码映射技术。  
因为数据从闪存下载到RAM是按照一定顺序的(无地址跳转)，所以从成本和紧凑性考虑，串行闪存是这种应用的最佳解决方案。代码映射技术还能压缩代码，降低对闪存的密度需求。  
串行闪存产品组合(M25Pxx系列的密度从512Kb到128Mb)，这些产品使很多利用代码映射技术的应用发生了革命性的变化，如硬盘驱动器、显卡、无线网卡、光驱、打印机、计算机(台式机或笔记本电脑的BIOS)、服务器、FPGA配置、液晶显示器、电视、数字电视、机顶盒、汽车收音机、POS机和游戏机等应用领域。无疑，串行外设接口(SPI)产品还将继续渗透到其它的代码映射技术应用领域。  
2.2.数据存储  
任何一种特定应用还需要存储器保存数据，例如调整参数、查阅表、历史日志、测量信息等，因为系统上电后立即下载代码，数据可以与代码共用存储器，两者之间不会出现任何冲突。液晶显示器(用户配置)、PC主板(BIOS配置)等应用就属于这种情况。其它应用设备使用专用存储器保存数据，如应答机(语音信息)、测量工具(数值)、医疗设备(记录)、游戏机(用户配置和分数)。因为这些数据不要求很快的读取速度，所以串行闪存仍是最佳的选择。  
###5.FPGA–ICE40HX1K###  
属
HX系列超低功耗FPGA系列，在网上找到一个中文文档：[戳这里](http://www.ic37.com/LATTICE_CN/ICE40HX4K-TQ144_datasheet_12974957/
"戳这里")  
相关英文文档传送门：  
[ICE40管脚图](http://download.csdn.net/detail/idevede/9779921 "ICE管脚图")  
[ICE40管脚及使用修改版](http://download.csdn.net/detail/idevede/9779925 "ICE40管脚详解")  
###6.PMOD###  
Pmod接口是将外设与FPGA开发板进行组合和匹配的一种方式。  
###7.IrDA###  
使用IrDA协议的红外连接设备。

![这里写图片描述](http://img.blog.csdn.net/20170313220834203?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaWRldmVkZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)