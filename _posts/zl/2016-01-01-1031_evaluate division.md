---
layout: post
title: evaluate division 
tags: [lua文章]
categories: [topic]
---
<ul>
<li><a href="https://leetcode.com/problems/evaluate-division/description/" title="Evaluate Division" target="_blank" rel="external noopener noreferrer">问题来源</a></li>
<li>问题简介<blockquote>
<p>给出一系列式子 a / b = k（k为非零小数），要求算出指定的算式，如:<br/>给出 a / b = 2.0, b / c = 3.0；<br/>则求 a / c。<br/>结果: 6.0<br/>若求不出，则为 -1。</p>
</blockquote>
</li>
</ul>
<hr/>

<h2 id="解题思路"><a href="#解题思路" class="headerlink" title="解题思路"></a>解题思路</h2>

<hr/>
<table>
<thead>
<tr>
<th style="text-align:center">时间复杂度</th>
<th style="text-align:center">空间复杂度</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:center">O(未知)</td>
<td style="text-align:center">O(未知)</td>
</tr>
</tbody>
</table>
<hr/>
<h2 id="Code"><a href="#Code" class="headerlink" title="Code"></a>Code</h2><pre><code>class Solution {
public:
    vector&lt;double&gt; calcEquation(vector&lt;pair&lt;string, string&gt;&gt; equations, vector&lt;double&gt;&amp; values, vector&lt;pair&lt;string, string&gt;&gt; queries) {
        unordered_map&lt;string, unordered_map&lt;string, double&gt;&gt; next;
        unordered_set&lt;string&gt; all;
        pair&lt;string, string&gt; cur;
        double val;
        for (int i = 0, len = equations.size(); i &lt; len; i++) {
            cur = equations[i];
            val = values[i];
            next[cur.first][cur.second] = val;
            next[cur.second][cur.first] = 1 / val;
            all.insert(cur.first);
            all.insert(cur.second);
        }
        vector&lt;double&gt; rs(queries.size(), -1.0);
        auto it = rs.begin();
        for (auto &amp;i : queries) {
            auto a_c = all;
            if (next.find(i.first) == next.end() || next.find(i.second) == next.end()) {
                *it++ = -1;
            } else {
                *it++ = dfs(i.first, i.second, next, a_c);
            }
        }
        return rs;
    }

    double dfs(const string &amp;cur, const string &amp;goal,
               unordered_map&lt;string, unordered_map&lt;string, double&gt;&gt; &amp;next,
              unordered_set&lt;string&gt; &amp;all) {
        if (cur == goal) {
            return next.find(cur) != next.end() ? !!(next[cur].begin() -&gt; second) : -1;
        }
        if (next[cur].find(goal) != next[cur].end()) {
            return next[cur][goal];
        }
        all.erase(cur);
        auto &amp;next_set = next[cur];
        double nextVal = -1;
        for (auto &amp;i : next_set) {
            if (all.find(i.first) == all.end()) {
                continue;
            }
            nextVal = dfs(i.first, goal, next, all);
            if (nextVal != -1) {
                return nextVal * i.second;
            }
        }
        return -1;
    }
};
</code></pre><hr/>
<h2 id="运行结果"><a href="#运行结果" class="headerlink" title="运行结果"></a>运行结果</h2><blockquote>
<p><img src="https://guoyl6.github.io//img/Evaluate-Division.png" alt="Evaluate Division" title="result"/></p>
</blockquote>
<hr/>