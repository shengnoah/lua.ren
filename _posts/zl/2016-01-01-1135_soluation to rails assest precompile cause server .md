---
layout: post
title: soluation to rails assest precompile cause server cpu usage too high 
tags: [lua文章]
categories: [topic]
---
I was using Mina to deploy Rails project to Digital Ocean VPS (1GB memory)

  

in mina `confir/deploy.rb` file  
there are one line:

    
    
    invoke :'rails:assets_precompile'
    

it mean precompile assest file on server

  

this it take way too long

    
    
    I, [2016-08-03T01:58:08.085687 #26033]  INFO -- : Writing /var/www/FS/tmp/build-147020368219820/public/assets/bootstrap.min-a624ed6e3c01894e8daa1456e852c26ce1ab4e8d52dcfd9ee4055395c9d39e5c.js
    I, [2016-08-03T01:58:08.086095 #26033]  INFO -- : Writing /var/www/FS/tmp/build-147020368219820/public/assets/bootstrap.min-a624ed6e3c01894e8daa1456e852c26ce1ab4e8d52dcfd9ee4055395c9d39e5c.js.gz
    -----> Cleaning up old releases (keeping 5)
    -----> Deploy finished
    -----> Building
    -----> Moving build to releases/12
    -----> Build finished
    -----> Launching
    -----> Updating the current symlink
    -----> Launching
    Command phased-restart sent success
    -----> Done. Deployed v12
    Connection to 107.170.237.232 closed.
           Elapsed time: 215.61 seconds
    

  

And most importantly when I run `ps aux`  
Node.js take 97~100% CPU  
WTF

  

### Soluation:

  1. Upgrade you VPS memory (we choose this, upgrade from 1GB -> 2GB)

  2. Precompile localy push to Github and then deploy ( tried, work fine )

  3. Create SWAP Space on Ubuntu ( tried, but not working on me )

I saw the third soluation from here:  
<http://stackoverflow.com/questions/22272339/rake-assetsprecompile-gets-
killed-when-there-is-a-console-session-open-in-produ>

  

if you got better soluation, please comment let me know, thanks