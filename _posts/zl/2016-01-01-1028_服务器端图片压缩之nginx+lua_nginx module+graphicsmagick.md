---
layout: post
title: 服务器端图片压缩之nginx+lua_nginx module+graphicsmagick 
tags: [lua文章]
categories: [topic]
---
![](https://xydcom.cn//images/banner/banner_12.jpg)

>
> 在开发电子商务网站时，同一个图片需要不同尺寸的缩略图片，一般有两种策略生成缩略图，一种在上传图片时，生成需要的缩略图，另一种是请求指定尺寸的图片时生存缩略图片，第一种方式有一定限制，就是需要提前知道所有尺寸的图片，如果前端ui设计改变了图片大小，需要重新生成。而第二种方式更加灵活，这里采用第二种方
> ，只是淘宝使用ImageMagick）。

这里主要借助lua_nginx
module调用GraphicsMagick命令生存生存缩略图片，缩略图片的尺寸包含在请求图片名称中，例如：xxxxx.jpg.80x80.jpg返回的就是xxx.jpg的80x80尺寸的图片大小。nginx配置如下：

上传图片名称使用32位随机字符替换掉，图片存放目录为图片名称前六个字母，每两个一组，构造三层目录结构存放，这样可以均匀存放图片在不同目录。避免目录存放文件数量限制。  
为了避免随意生成不同尺寸的缩略图，这里做了限制，在image_sizes中定义了需要的缩略图尺寸。

  * 初次版本
    
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
    

|

    
        location /testImg/ {    
        rewrite_by_lua '    
            local image_root = "/home/tomcat/eisp-files";    
            function file_notexists(name)    
               local f=io.open(name,"r")    
               if f~=nil then io.close(f) return false else return true end    
            end    
      
    local uri = ngx.re.sub(ngx.var.uri, "/testImg/([0-9a-zA-Z]+)/([0-9a-zA-Z]{2})([0-9a-zA-Z]{2})([0-9a-zA-Z]{2})([0-9a-zA-Z]+).([0-9a-zA-Z]+)", "/images/$1/$2/$3/$4/$2$3$4$5.$6", "o");    
            local index = string.find(uri, "([0-9]+)x([0-9]+)");    
            local originalUri = string.sub(uri, 0, index-2);    
            local area = string.sub(uri, index);    
            index = string.find(area, "([.])");    
            area = string.sub(area, 0, index-1);    
      
            if file_notexists(image_root .. uri) then    
               local image_sizes = {"80x80", "800x600", "40x40"};    
               function table.contains(table, element)    
                  for _, value in pairs(table) do    
                     if value == element then    
                        return true    
                     end    
                  end    
                  return false    
               end    
      
               if table.contains(image_sizes, area) then    
                   local command = "gm convert " .. image_root ..  originalUri  .. " -thumbnail " .. area .. " -background gray -gravity center -extent " .. area .. " " .. image_root .. uri;    
                   os.execute(command);    
                   ngx.req.set_uri(uri, true);    
               else    
                   ngx.exit(404);    
               end;    
            else    
               ngx.req.set_uri(uri, true);    
            end;    
        ';    
    }    
      
    location /images/ {    
        alias /home/tomcat/eisp-files/images/;    
        expires 7d;    
    }  
      
  
---|---  
  * 最后一次修改版本
    
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
    

|

    
        location /images/ {    
        set $image_root /home/nds/nds-files;    
      
        if ($uri ~* "/images/([0-9a-zA-Z]+)/([0-9a-zA-Z]{2})([0-9a-zA-Z]{2})([0-9a-zA-Z]{2})([0-9a-zA-Z]+).(.*)") {    
           set $filePath "$image_root/$1/$2/$3/$4/$2$3$4$5.$6";    
           set $reqPath  "/$1/$2/$3/$4/$2$3$4$5.$6";    
        }    
      
        set $file "$image_root$reqPath";    
      
        if (-f $file) {    
           rewrite "/images/(.+)" /innerImages$reqPath last;    
        }    
      
        if (!-f $file) {    
           rewrite_by_lua '    
              local index = string.find(ngx.var.filePath, "([0-9]+)x([0-9]+)");    
              local originalUri = string.sub(ngx.var.filePath, 0, index-2);    
              local area = string.sub(ngx.var.filePath, index);    
              index = string.find(area, "([.])");    
              area = string.sub(area, 0, index-1);    
      
              local image_sizes = {"155x155", "400x400","104x104", "50x50", "40x40", "56x56", "172x172","800x600"};    
              function table.contains(table, element)    
                 for _, value in pairs(table) do    
                    if value == element then    
                       return true    
                    end    
                 end    
                 return false    
              end    
      
              if table.contains(image_sizes, area) then    
                 local command = "gm convert " ..  originalUri  .. " -thumbnail " .. area .. " -background white -gravity center -extent " .. area .. " " .. ngx.var.file;    
                 os.execute(command);    
                 ngx.req.set_uri("/innerImages" .. ngx.var.reqPath, true);    
              else    
                 ngx.exit(404);    
              end;    
           ';    
        }    
    }    
      
    location /innerImages/ {    
        alias /home/nds/nds-files/;    
        expires max;     
        tcp_nodelay off;     
        tcp_nopush on;    
    }  
      
  
---|---  

本文引自<http://www.iteye.com/topic/1125126>