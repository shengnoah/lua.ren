---
layout: post
title: 结合Fluentd实现ngx_lua_waf页面展示 
tags: [lua文章]
categories: [lua文章]
---
### 0x00 概述

最近在看lua_waf,看看怎么搞个web界面出来，于是有了这篇笔记。

![image](http://image.3001.net/images/20170223/14878422689085.png)

上图是大佬分享的waf日志处理流程，不过我这篇文章讲的就简单多了，如下, 纯属搞着玩儿：

![](https://www.maliciouskr.cc//2016/10/20/结合Fluentd实现ngx_lua_waf页面展示/fluentd_waf1.jpg)

**ngx_lua_waf简介**

ngx_lua_waf，是一个轻量级、高性能的WAF模块。

  * 防止sql注入，本地包含，部分溢出，fuzzing测试，XSS, SSRF等web攻击
  * 防止svn/备份之类文件泄漏
  * 防止ApacheBench之类压力测试工具的攻击
  * 屏蔽常见的扫描黑客工具，扫描器
  * 屏蔽异常的网络请求
  * 屏蔽图片附件类目录php执行权限
  * 防止webshell上传

详情（安装方法）见：<https://github.com/loveshell/ngx_lua_waf>

**Fluentd简介**

Fluentd，是一个开源收集事件和日志系统，它目前提供150+扩展插件让你存储大数据用于日志搜索，数据分析和存储。这里我们用fluentd搜集lua_waf日志。

![image](http://docs.fluentd.org/images/fluentd-architecture.png)

官网：<http://www.fluentd.org/>

文档中心：<http://docs.fluentd.org/v0.12/articles/quickstart>

### 0x01 Fluentd安装

lua_waf安装略，网上有很多教程，直接开始fluentd部署。

1.通过rpm安装，执行如下命令（如果有报错，按实际报错提示处理即可）：  

    
    
    1  
    

|

    
    
    $ curl -L https://toolbelt.treasuredata.com/sh/install-redhat-td-agent2.sh | sh  
      
  
---|---  
  
2.安装好之后便可以启动：

    
    
    1  
    2  
    3  
    4  
    

|

    
    
    $ /etc/init.d/td-agent start   
    Starting td-agent: [  OK  ]  
    $ /etc/init.d/td-agent status  
    td-agent (pid  21678) is running...  
      
  
---|---  
  
3.配置，如何取日志，如何进行处理分析都在这里进行配置（td-agent.conf）：

    
    
    1  
    

|

    
    
    $ sudo vi /etc/td-agent/td-agent.conf  
      
  
---|---  
  
4.必要的插件，我们这里是要将取得的日志送到mysql做分析以及页面展示，所以需要下载fluent-plugin-
mysql（fluentd默认没有mysql插件）。

    
    
    1  
    

|

    
    
    gem地址：https://rubygems.org/gems/fluent-plugin-mysql/versions/0.1.5  
      
  
---|---  
  
安装命令：

    
    
    1  
    2  
    3  
    4  
    

|

    
    
    yum install mysql-devel（依赖包）  
    /usr/sbin/td-agent-gem install jsonpath  
    /usr/sbin/td-agent-gem install mysql2-cs-bind  
    /usr/sbin/td-agent-gem install fluent-plugin-mysql -v 0.1.5  
      
  
---|---  
  
**ngx_lua_waf日志格式**

init.lua代码片段:

    
    
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
    

|

    
    
    #ngx_lua_waf拦截日志格式，能够与fluentd正则匹配。  
      
    function log(method,url,data,ruletag)  
        if attacklog then  
            local realIp = getClientIp()  
            local ua = ngx.var.http_user_agent  
            local servername=ngx.var.server_name  
            local time=ngx.localtime()  
            if ua  then  
                line = realIp.." "..servername.." ""..time.."" ""..method.." "..servername..url.."" ""..data.."" ""..ua.."" ""..ruletag..""n"  
            else  
                line = realIp.." "..servername.." ""..time.."" ""..method.." "..servername..url.."" ""..data.."" - ""..ruletag..""n"  
            end  
            local filename = logpath..'/'..servername.."_sec.log"  
            write(filename,line)  
        end  
    end  
      
  
---|---  
  
**fluentd配置**

td-agent.conf片段：

    
    
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
    

|

    
    
    #将/usr/local/nginx/logs/hack/ngx_lua_waf_sec.log日志实时同步到mysql  
      
    <source>  
      @type tail  
      path /usr/local/nginx/logs/hack/ngx_lua_waf_sec.log  
      pos_file /var/log/td-agent/httpd-access.log.pos  
      format /^(?<ip>[^ ]*) (?<website>[^ ]*) "(?<time>[^"]*)" "(?<method>S+)(?: +(?<path>[^"]*))?" "(?<demo>[^"]*)" "(?<agent>[^"]*)" "(?<rule>[^"]*)"$/  
      time_format %Y-%m-%d %H:%M:%S  
      tag test.http  
    </source>  
      
    <match test.**>  
      @type mysql_bulk  
      host 172.20.3.64  
      database waf  
      username root  
      password qwe@123456  
      column_names id,ip,website,time,method,path,demo,agent,rule  
      table waflog  
      time_format %Y-%m-%d %H:%M:%S  
      flush_interval 3s  
    </match>  
      
  
---|---  
  
### 0x02 mysql接收waf日志

下面这张表存放waf的拦截日志，比较简单。接收上面td-agent传过来的数据。

    
    
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
    

|

    
    
    use pscan;  
      
    SET FOREIGN_KEY_CHECKS=0;  
      
    -- ----------------------------  
    -- Table structure for waflog  
    -- ----------------------------  
    DROP TABLE IF EXISTS `waflog`;  
    CREATE TABLE `waflog` (  
      `id` int(100) NOT NULL,  
      `ip` varchar(100) DEFAULT NULL,  
      `website` varchar(255) DEFAULT NULL,  
      `time` datetime DEFAULT NULL,  
      `method` varchar(255) DEFAULT NULL,  
      `path` varchar(255) DEFAULT NULL,  
      `demo` varchar(255) DEFAULT NULL,  
      `agent` varchar(255) DEFAULT NULL,  
      `rule` varchar(255) DEFAULT NULL,  
      PRIMARY KEY (`id`)  
    ) ENGINE=MyISAM DEFAULT CHARSET=utf8;  
      
    -- ----------------------------  
    -- Records of waflog  
    -- ----------------------------  
    INSERT INTO `waflog` VALUES ('1', '172.20.3.64', 'www.maliciouskr.cc', '2016-09-08 20:18:03', 'GET', '/dump.php?<img<!--+--> src=x onerror=alert(9549);//><!-- -->', '-', 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.21 (KHTML, like Gecko) Chrome/41.0.2228.0 Safari/537.21', '(?:define|eval|file_get_contents|include|require|require_once|shell_exec|phpinfo|system|passthru|preg_\\w+|execute|echo|print|print_r|var_dump|(fp)open|alert|showmodaldialog)\\(\"');  
    INSERT INTO `waflog` VALUES ('2', '172.20.3.64', 'www.maliciouskr.cc', '2016-09-08 20:18:03', 'GET', '/dump.php?<img<!--+--> src=x onerror=alert(9549);//><!-- -->', '-', 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.21 (KHTML, like Gecko) Chrome/41.0.2228.0 Safari/537.21', '(?:define|eval|file_get_contents|include|require|require_once|shell_exec|phpinfo|system|passthru|preg_\\w+|execute|echo|print|print_r|var_dump|(fp)open|alert|showmodaldialog)\\(\"');  
    INSERT INTO `waflog` VALUES ('3', '172.20.3.64', 'www.maliciouskr.cc', '2016-09-08 20:18:03', 'GET', '/dump.php?<img<!--+--> src=x onerror=alert(9549);//><!-- -->', '-', 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.21 (KHTML, like Gecko) Chrome/41.0.2228.0 Safari/537.21', '(?:define|eval|file_get_contents|include|require|require_once|shell_exec|phpinfo|system|passthru|preg_\\w+|execute|echo|print|print_r|var_dump|(fp)open|alert|showmodaldialog)\\(\"');  
    INSERT INTO `waflog` VALUES ('4', '172.20.3.64', 'www.maliciouskr.cc', '2016-09-08 20:18:03', 'GET', '/dump.php?<img<!--+--> src=x onerror=alert(9549);//><!-- -->', '-', 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.21 (KHTML, like Gecko) Chrome/41.0.2228.0 Safari/537.21', '(?:define|eval|file_get_contents|include|require|require_once|shell_exec|phpinfo|system|passthru|preg_\\w+|execute|echo|print|print_r|var_dump|(fp)open|alert|showmodaldialog)\\(\"');  
    INSERT INTO `waflog` VALUES ('5', '172.20.3.64', 'www.maliciouskr.cc', '2016-09-08 20:18:03', 'GET', '/dump.php?<img<!--+--> src=x onerror=alert(9549);//><!-- -->', '-', 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.21 (KHTML, like Gecko) Chrome/41.0.2228.0 Safari/537.21', '(?:define|eval|file_get_contents|include|require|require_once|shell_exec|phpinfo|system|passthru|preg_\\w+|execute|echo|print|print_r|var_dump|(fp)open|alert|showmodaldialog)\\(\"');  
    INSERT INTO `waflog` VALUES ('6', '172.20.3.64', 'www.maliciouskr.cc', '2016-09-08 20:18:03', 'GET', '/dump.php?<img<!--+--> src=x onerror=alert(9549);//><!-- -->', '-', 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.21 (KHTML, like Gecko) Chrome/41.0.2228.0 Safari/537.21', '(?:define|eval|file_get_contents|include|require|require_once|shell_exec|phpinfo|system|passthru|preg_\\w+|execute|echo|print|print_r|var_dump|(fp)open|alert|showmodaldialog)\\(\"');  
    INSERT INTO `waflog` VALUES ('7', '172.20.3.64', 'www.maliciouskr.cc', '2016-09-08 20:18:03', 'GET', '/dump.php?<img<!--+--> src=x onerror=alert(9549);//><!-- -->', '-', 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.21 (KHTML, like Gecko) Chrome/41.0.2228.0 Safari/537.21', '(?:define|eval|file_get_contents|include|require|require_once|shell_exec|phpinfo|system|passthru|preg_\\w+|execute|echo|print|print_r|var_dump|(fp)open|alert|showmodaldialog)\\(\"');  
    INSERT INTO `waflog` VALUES ('8', '172.20.3.64', 'www.maliciouskr.cc', '2016-09-08 20:18:03', 'GET', '/dump.php?<img<!--+--> src=x onerror=alert(9549);//><!-- -->', '-', 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.21 (KHTML, like Gecko) Chrome/41.0.2228.0 Safari/537.21', '(?:define|eval|file_get_contents|include|require|require_once|shell_exec|phpinfo|system|passthru|preg_\\w+|execute|echo|print|print_r|var_dump|(fp)open|alert|showmodaldialog)\\(\"');  
    INSERT INTO `waflog` VALUES ('9', '172.20.3.64', 'www.maliciouskr.cc', '2016-09-08 20:18:03', 'GET', '/dump.php?<img<!--+--> src=x onerror=alert(9549);//><!-- -->', '-', 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.21 (KHTML, like Gecko) Chrome/41.0.2228.0 Safari/537.21', '(?:define|eval|file_get_contents|include|require|require_once|shell_exec|phpinfo|system|passthru|preg_\\w+|execute|echo|print|print_r|var_dump|(fp)open|alert|showmodaldialog)\\(\"');  
      
  
---|---  
  
### 0x03 页面展示

![](https://www.maliciouskr.cc//2016/10/20/结合Fluentd实现ngx_lua_waf页面展示/fluentd_waf2.png)

源码（php+mysql+Bootstrap）见[github](https://github.com/maliciouskr/lua_waf_ui)，代码质量为初学者水平，见谅。