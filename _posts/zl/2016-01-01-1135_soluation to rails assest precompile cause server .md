---
layout: post
title: soluation to rails assest precompile cause server cpu usage too high 
tags: [lua文章]
categories: [topic]
---
<p>I was using Mina to deploy Rails project to Digital Ocean VPS (1GB memory)</p>

<p><br/></p>

<p>in mina <code class="highlighter-rouge">confir/deploy.rb</code> file<br/>
there are one line:</p>

<div class="highlighter-rouge"><pre class="highlight"><code>invoke :&#39;rails:assets_precompile&#39;
</code></pre>
</div>
<p>it mean precompile assest file on server</p>

<p><br/></p>

<p>this it take way too long</p>

<div class="highlighter-rouge"><pre class="highlight"><code>I, [2016-08-03T01:58:08.085687 #26033]  INFO -- : Writing /var/www/FS/tmp/build-147020368219820/public/assets/bootstrap.min-a624ed6e3c01894e8daa1456e852c26ce1ab4e8d52dcfd9ee4055395c9d39e5c.js
I, [2016-08-03T01:58:08.086095 #26033]  INFO -- : Writing /var/www/FS/tmp/build-147020368219820/public/assets/bootstrap.min-a624ed6e3c01894e8daa1456e852c26ce1ab4e8d52dcfd9ee4055395c9d39e5c.js.gz
-----&gt; Cleaning up old releases (keeping 5)
-----&gt; Deploy finished
-----&gt; Building
-----&gt; Moving build to releases/12
-----&gt; Build finished
-----&gt; Launching
-----&gt; Updating the current symlink
-----&gt; Launching
Command phased-restart sent success
-----&gt; Done. Deployed v12
Connection to 107.170.237.232 closed.
       Elapsed time: 215.61 seconds
</code></pre>
</div>
<p><br/></p>

<p>And most importantly when I run <code class="highlighter-rouge">ps aux</code> <br/>
Node.js take 97~100% CPU <br/>
WTF</p>

<p><br/></p>

<h3 id="soluation">Soluation:</h3>
<ol>
  <li>
    <p>Upgrade you VPS memory (we choose this, upgrade from 1GB -&gt; 2GB)</p>
  </li>
  <li>
    <p>Precompile localy push to Github and then deploy ( tried, work fine )</p>
  </li>
  <li>
    <p>Create SWAP Space on Ubuntu ( tried, but not working on me )</p>
  </li>
</ol>

<p>I saw the third soluation from here: <br/>
<a href="http://stackoverflow.com/questions/22272339/rake-assetsprecompile-gets-killed-when-there-is-a-console-session-open-in-produ">http://stackoverflow.com/questions/22272339/rake-assetsprecompile-gets-killed-when-there-is-a-console-session-open-in-produ</a></p>

<p><br/></p>

<p>if you got better soluation, please comment let me know, thanks</p>