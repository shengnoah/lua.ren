---
layout: post
title: evaluate division 
tags: [lua文章]
categories: [topic]
---
## 思路

巧妙的方法，化为双向图，例如等式a / b = 2.0，可以转化为两条边：，，其长度分别为2.0，0.5。  
对queries的每一项，先判断2个点是否都出现，然后判断是不是自己到自己，最后用dfs进行搜索，然后用递归进行每条边权重的相乘，如果最后找不到路径，则返回-1.0。  

    
    
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

    
    
    class  {
    
    public:
    
        double dfs(string from, string to, map<string, map<string, double>> &mp, set<string> & visited, double val)
    
    {
    
        if (mp[from].count(to)) {
    
            return mp[from][to] * val;
    
        }
    
        for (pair<string, double>p : mp[from]) {
    
            string str = p.first;
    
            if (visited.count(str) == 0) {
    
                visited.insert(str);
    
                double cur = dfs(str, to, mp, visited, p.second * val);
    
                if (cur != -1.0) return cur;
    
            }
    
        }
    
        return -1.0;
    
    }
    
    vector<double> calcEquation(vector<pair<string, string>> equations, vector<double>& values, vector<pair<string, string>> queries) {
    
        vector<double> result;
    
        map<string, map<string, double>> mp;
    
        int i = 0;
    
        for (pair<string, string> p : equations) {
    
            mp[p.first][p.second] = values[i];
    
            mp[p.second][p.first] = 1.0 / values[i];
    
            i++;
    
        }
    
        for (pair<string, string> p : queries) {
    
            if (mp.count(p.first) && mp.count(p.second)) {
    
                if (p.first == p.second) {
    
                    result.push_back(1.0);
    
                    continue;
    
                }
    
                else {
    
                    set<string> visited;
    
                    result.push_back(dfs(p.first, p.second, mp, visited, 1.0));
    
                }
    
            }
    
            else {
    
                result.push_back(-1.0);
    
            }
    
        }
    
        return result;
    
    }
    
    };  
  
---|---