---
layout: post
title: ubuntu安装wireshark报错lua 
tags: [lua文章]
categories: [topic]
---
<p>ubuntu: 16.04</p>
<p>打开终端，安装wireshark过程如下：<br/></p><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div><div class="line">5</div><div class="line">6</div></pre></td><td class="code"><pre><div class="line"># Add the stable official PPA.</div><div class="line">sudo add-apt-repository ppa:wireshark-dev/stable</div><div class="line"># Update the repository</div><div class="line">sudo apt-get update</div><div class="line"># Install wireshark</div><div class="line">sudo apt-get install wireshark</div></pre></td></tr></tbody></table></figure><p></p>
<p>我的在安装过程中卡住了一会，提示<code>51% [正在等待报头]</code>，等了好一会，才继续了。</p>
<p>安装完成后，运行<code>sudo wireshark</code>,就可以打开界面，但是会报错：</p>
<p><img src="https://huanyouchen-1252081928.cos.ap-shanghai.myqcloud.com/2018-05-08%2020-19-00%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png?imageView2/0/q/75|watermark/2/text/aHVhbnlvdWNoZW4uZ2l0aHViLmlv/font/5qW35L2T/fontsize/320/fill/IzBBMEEwQQ==/dissolve/93/gravity/SouthEast/dx/10/dy/10|imageslim" alt="img"/></p>
<p>网上查了一下，解决方法如下：<br/></p><figure class="highlight plain"><table><tbody><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div><div class="line">5</div><div class="line">6</div><div class="line">7</div><div class="line">8</div><div class="line">9</div></pre></td><td class="code"><pre><div class="line"># fix lua error</div><div class="line">cd /usr/share/wireshark</div><div class="line">sudo gedit init.lua</div><div class="line"># 在打开的init.lua中，找到这一行，改为true-- Set disable_lua to true to disable Lua support.</div><div class="line">disable_lua = true</div><div class="line"># 然后在终端里：</div><div class="line">sudo dpkg-reconfigure wireshark-common</div><div class="line"># 选择是(yes)，再然后：</div><div class="line">sudo adduser $USER wireshark</div></pre></td></tr></tbody></table></figure><p></p>
<p>然后重启电脑，再次打开终端，运行<code>wireshark</code>，就没有报错了。</p>
<p>方法来源：<br/><a href="https://askubuntu.com/questions/700712/how-to-install-wireshark" target="_blank" rel="external noopener noreferrer">https://askubuntu.com/questions/700712/how-to-install-wireshark</a></p>