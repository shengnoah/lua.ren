---
layout: post
title: lua串行化_序列化serialization（串行流_字节流） 
tags: [lua文章]
categories: [topic]
---
<h2 id="杂乱字符串转义显示处理"><a href="#杂乱字符串转义显示处理" class="headerlink" title="杂乱字符串转义显示处理"></a>杂乱字符串转义显示处理</h2><pre><code>-- 匹配符方式 string.format
-- [=[...]=]方式
a = &#39;a &#34;program lua [[ ]]]]&#34;&#39;
print(string.format(&#39;%q&#39;,a))
serialize(a) -- [=[a &#34;program lua [[ ]]]]&#34;]=]
</code></pre><h2 id="number、stirng、table序列化"><a href="#number、stirng、table序列化" class="headerlink" title="number、stirng、table序列化"></a>number、stirng、table序列化</h2><pre><code>function serialize( o )
    if type(o) == &#34;number&#34; then
        io.write(o)
    elseif type(o) == &#34;string&#34; then
    io.write(&#34;[=[&#34;,o,&#34;]=]&#34;)
    else
        print(&#39;b&#39;)
    end
end

-- 杂乱字符串转义显示处理
-- 匹配符方式 string.format
-- [=[...]=]方式
a = &#39;a &#34;program lua [[ ]]]]&#34;&#39;
print(string.format(&#39;%q&#39;,a))
serialize(a) -- [=[a &#34;program lua [[ ]]]]&#34;]=]

-- 保存无环table
function n_serialize( o )
    if type(o) == &#34;number&#34; then
        io.write(o)
    elseif type(o) == &#34;string&#34; then
        io.write(string.format(&#39;%q&#39;,o))    
    elseif type(o) == &#34;table&#34; then
        io.write(&#34;{n&#34;)
        for k,v in pairs(o) do
            io.write(&#34; &#34;, k, &#34;=&#34;)
            -- 递归调用
            n_serialize(v)
            io.write(&#34;,n&#34;)
        end
        io.write(&#34;}n&#34;)
    else
        print(&#39;b&#39;)
    end        
end

b = {lang = &#34;lua&#34;, content = &#39;&#34;dddd&#34;[[&#34;&#39;,3}
n_serialize(b) 
--[[
{
 1=3,
 lang=&#34;lua&#34;,
 content=&#34;&#34;dddd&#34;[[&#34;&#34;,
}
--]]
</code></pre>