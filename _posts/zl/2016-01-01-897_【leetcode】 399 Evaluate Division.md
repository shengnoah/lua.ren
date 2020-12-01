---
layout: post
title: 【leetcode】 399 Evaluate Division 
tags: [lua文章]
categories: [topic]
---
给出一些等式，求给定表达式的结果

![](https://whutweihuan.github.io//2018/10/27/2018-10-27-blog/raindrop-3629132_640.jpg)  

## 描述

方程式以A/B=k格式给出，其中A和B是用字符串表示的变量，k是实数（浮点数）。给定一些查询，返回答案。如果答案不存在，返回-1。

比如给定: A / B = 2.0，B / C = 3.0。  
要求: A / C = ？ B / A = ？ A / E = ？A / A = ？，X / X = ？ …

## 样例

    
    
    1  
    2  
    

|

    
    
    输入: [ ["a", "b"], ["b", "c"] ],[2.0, 3.0],[ ["a", "c"], ["b", "a"], ["a", "e"], ["a", "a"], ["x", "x"] ]  
    输出: [6.0, 0.5, -1.0, 1.0, -1.0 ]  
      
  
---|---  
  
## 思路

初期想直接按照实体数字，比如将 a 设定成 1 ，就可以求出 b ，
递推求出其他的字母，后来发现这样做之后遇到之前两个字母都没有记录的数字会很麻烦，于是想用一个 rest
集合先保存这些两个字母都没被记录的，等第一轮结束后再进行 rest 集合的处理，后来发现这样也不行，原因在于 rest
集合中也会有两个同时没有出现的，难道再创建一个 rest 2 号吗 ?
显然这样十分的不合理。参考了一份代码，采用的是深度优先搜索，很厉害。注意对集合map<string,<string,map> >的理解。

## 代码

代码[来自](https://github.com/haoel/leetcode/blob/master/algorithms/cpp/evaluateDivision/EvaluateDivision.cpp)

    
    
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
    

|

    
    
    class  {  
    private:  
        bool dfs(unordered_map<string, unordered_map<string, double> > &m,   
            string startvalue, string endvalue, unordered_map<string,bool> &visited, double & res) {  
            if (m.find(startvalue) == m.end() || m.find(endvalue) == m.end()) { return false;}  
            if (startvalue == endvalue) { return true; }  
      
            for (auto iter = m[startvalue].begin(); iter != m[startvalue].end(); iter++) {  
                if (visited.find(iter->first) != visited.end()) { continue; }  
                visited[iter->first] = true;  
                double old = res;  
                res *= m[startvalue][iter->first];  
                if (dfs(m, iter->first, endvalue, visited, res)) { return true; }  
                visited.erase(iter->first);  
                res = old;  
            }  
      
            return false;  
        }  
      
    public:  
        vector<double> calcEquation(vector<pair<string, string>> equations, vector<double>& values, vector<pair<string, string>> queries) {  
            vector<double> result;  
            unordered_map<string, unordered_map<string, double> > m;  
            for (int i = 0; i < equations.size();i++) {  
                string first = equations[i].first;  
                string second = equations[i].second;  
                m[first][second] = values[i];  
                m[second][first] = 1.0 / values[i];  
            }  
      
            for (int i = 0; i < queries.size(); i++) {  
                string start = queries[i].first;  
                string end = queries[i].second;  
                double res = 1.0;  
                unordered_map<string, bool> visited;  
                visited[start] = true;  
      
                if (dfs(m, start, end, visited, res)) { result.push_back(res);}  
                else { result.push_back(-1.0);}  
      
            }  
            return result;  
        }  
    };  
      
  
---|---  
  
## 参考

[原题链接]()

* * *