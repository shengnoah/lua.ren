---
layout: post
title: evaluate division 
tags: [lua文章]
categories: [topic]
---
  * [问题来源](https://leetcode.com/problems/evaluate-division/description/ "Evaluate Division")
  * 问题简介

> 给出一系列式子 a / b = k（k为非零小数），要求算出指定的算式，如:  
> 给出 a / b = 2.0, b / c = 3.0；  
> 则求 a / c。  
> 结果: 6.0  
> 若求不出，则为 -1。

* * *

## 解题思路

* * *

时间复杂度 | 空间复杂度  
---|---  
O(未知) | O(未知)  
  
* * *

## Code

    
    
    class Solution {
    public:
        vector<double> calcEquation(vector<pair<string, string>> equations, vector<double>& values, vector<pair<string, string>> queries) {
            unordered_map<string, unordered_map<string, double>> next;
            unordered_set<string> all;
            pair<string, string> cur;
            double val;
            for (int i = 0, len = equations.size(); i < len; i++) {
                cur = equations[i];
                val = values[i];
                next[cur.first][cur.second] = val;
                next[cur.second][cur.first] = 1 / val;
                all.insert(cur.first);
                all.insert(cur.second);
            }
            vector<double> rs(queries.size(), -1.0);
            auto it = rs.begin();
            for (auto &i : queries) {
                auto a_c = all;
                if (next.find(i.first) == next.end() || next.find(i.second) == next.end()) {
                    *it++ = -1;
                } else {
                    *it++ = dfs(i.first, i.second, next, a_c);
                }
            }
            return rs;
        }
    
        double dfs(const string &cur, const string &goal,
                   unordered_map<string, unordered_map<string, double>> &next,
                  unordered_set<string> &all) {
            if (cur == goal) {
                return next.find(cur) != next.end() ? !!(next[cur].begin() -> second) : -1;
            }
            if (next[cur].find(goal) != next[cur].end()) {
                return next[cur][goal];
            }
            all.erase(cur);
            auto &next_set = next[cur];
            double nextVal = -1;
            for (auto &i : next_set) {
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
    

* * *

## 运行结果

> ![Evaluate Division](https://guoyl6.github.io//img/Evaluate-Division.png)

* * *