---
layout: post
title: 399 Evaluate Division 
tags: [lua文章]
categories: [topic]
---
Equations are given in the format `A / B = k`, where `A` and `B` are variables
represented as strings, and `k` is a real number (floating point number).
Given some queries, return the answers. If the answer does not exist, return
`-1.0`.

**Example:**  
Given `a / b = 2.0, b / c = 3.0.`  
queries are: `a / c = ?, b / a = ?, a / e = ?, a / a = ?, x / x = ? .`  
return `[6.0, 0.5, -1.0, 1.0, -1.0 ].`

The input is: `vector<pair<string, string>> equations, vector<double>& values,
vector<pair<string, string>> queries`, where `equations.size() ==
values.size()`, and the values are positive. This represents the equations.
Return `vector<double>`.

According to the example above:

    
    
    1
    
    2
    
    3

|

    
    
    equations = [ ["a", "b"], ["b", "c"] ],
    
    values = [2.0, 3.0],
    
    queries = [ ["a", "c"], ["b", "a"], ["a", "e"], ["a", "a"], ["x", "x"] ].  
  
---|---  
  
The input is always valid. You may assume that evaluating the queries will
result in no division by zero and there is no contradiction.

## 思路

我们要利用图的方法来解决这道问题。 我们将每一次除法操作看作有向图的一条边。如a÷b，则构造一个自a到b的边，边上的权值为2，以此类推。

构造邻接表，分配数组下标，利用BFS的方法遍历图。如果我们需要提高执行效率的话，还可以给图的邻接表增加新的项目以缓存结果。
[Shell32的博客](https://www.xiadong.info/2016/09/14/leetcode-399-evaluate-
division/)

    
    
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
    
    50
    
    51
    
    52
    
    53
    
    54
    
    55
    
    56
    
    57
    
    58
    
    59
    
    60
    
    61
    
    62
    
    63
    
    64
    
    65
    
    66
    
    67
    
    68
    
    69
    
    70
    
    71
    
    72
    
    73
    
    74
    
    75
    
    76
    
    77
    
    78
    
    79
    
    80

|

    
    
    class Solution {
    
    public:
    
        vector<double> calcEquation(vector<pair<string, string>> equations, vector<double>& values, vector<pair<string, string>> queries) {
    
            vector<double> result;
    
            int count = 1;
    
            unordered_map<string, int> index;
    
            
    
            for (int i = 0; i < equations.size(); i++){
    
                string tmp1 = equations[i].first;
    
                string tmp2 = equations[i].second;
    
                if(index.find(tmp1) == index.end()){
    
                    index[tmp1] = count;
    
                    count++;
    
                }
    
                if(index.find(tmp2) == index.end()){
    
                    index[tmp2] = count;
    
                    count++;
    
                }
    
            }
    
            vector< vector<double> > graph(index.size());
    
            for (int i = 0; i < index.size(); i++){
    
                graph[i].resize(index.size());
    
            }
    
            for (int i = 0; i < equations.size(); i++){
    
                int tmp1 = index[equations[i].first] - 1;
    
                int tmp2 = index[equations[i].second] - 1;
    
                graph[tmp1][tmp2] = values[i];
    
                graph[tmp2][tmp1] = 1.0 / values[i];
    
            }
    
            /*for(int i = 0; i < graph.size(); i++){
    
                for(int j = 0; j < graph[i].size(); j++){
    
                    cout << graph[i][j] << " ";
    
                }
    
                cout << endl;
    
            }*/
    
            for (int i = 0; i < queries.size(); i++){
    
                int start = index[queries[i].first] - 1;
    
                int end = index[queries[i].second] - 1;
    
                if (start < 0 || end < 0){
    
                    result.push_back(-1.0);
    
                }
    
                else{
    
                    if (start == end) result.push_back(1.0);
    
                    else{
    
                        vector<int> visited(index.size());
    
                        queue<int> q;
    
                        queue<double> ans;
    
                        q.push(start);
    
                        ans.push(1);
    
                        bool hasResult = false;
    
                        while(!q.empty()){
    
                            int tmp = q.front();
    
                            double value = ans.front();
    
                            visited[tmp] = 1;
    
                            for(int j = 0; j < graph.size(); j++){
    
                                if(visited[j] == 0 && graph[tmp][j] > 0){
    
                                    if (j != end) {
    
                                        q.push(j);
    
                                        ans.push(value * graph[tmp][j]);
    
                                    }
    
                                    else{
    
                                        result.push_back(value * graph[tmp][j]);
    
                                        hasResult = true;
    
                                    }
    
                                }
    
                            }
    
                            q.pop();
    
                            ans.pop();
    
                        }
    
                        if (!hasResult) result.push_back(-1.0);
    
                    }
    
                }
    
            }
    
            return result;
    
        }
    
    };  
  
---|---  
  
最后注意几个C++语言当中的小技巧。

迭代器访问，注意用it->first访问pair的第一个元素，it是一个指向迭代器的指针。

    
    
    1
    
    2
    
    3

|

    
    
    for(auto it = equations.begin(); it != equations.end(); ++it){
    
    	cout << it->first << " " << it->second << endl;
    
    }  
  
---|---  
  
C++当中，STL标准库当中的set求交集的代码。

    
    
    1
    
    2
    
    3
    
    4
    
    5
    
    6

|

    
    
    string a = queries[i].first;
    
    string b = queries[i].second;
    
    set<string> pick;
    
    set_intersection(sets[a].begin(), sets[a].end(), sets[b].begin(), sets[b].end(), inserter(pick, pick.begin()));
    
    //取到pick集合的第一个元素
    
    string element = *pick.begin();  
  
---|---  
  
有问题的一段代码，从图的观点来看，只做到了第一层DFS，并没有进一步往下走。所以失败了QAQ

    
    
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

|

    
    
    class Solution {
    
    public:
    
        vector<double> calcEquation(vector<pair<string, string>> equations, vector<double>& values, vector<pair<string, string>> queries) {
    
            unordered_map<string, unordered_map<string, double> > stats;
    
            unordered_map<string, set<string> > sets;
    
            for(int i=0; i<values.size(); i++){
    
                stats[equations[i].first][equations[i].first] = 1.0;
    
                stats[equations[i].first][equations[i].second] = values[i];
    
                stats[equations[i].second][equations[i].second] = 1.0;
    
                stats[equations[i].second][equations[i].first] = 1.0 / values[i];
    
                sets[equations[i].first].emplace(equations[i].first);
    
                sets[equations[i].first].emplace(equations[i].second);
    
                sets[equations[i].second].emplace(equations[i].first);
    
                sets[equations[i].second].emplace(equations[i].second);
    
            }
    
            vector<double> result;
    
            for(int i=0; i<queries.size(); i++){
    
                string a = queries[i].first;
    
                string b = queries[i].second;
    
                set<string> pick;
    
                set_intersection(sets[a].begin(), sets[a].end(), sets[b].begin(), sets[b].end(), inserter(pick, pick.begin()));
    
                if (pick.empty()){
    
                    result.push_back(-1.0);
    
                }
    
                else{
    
                    string element = *pick.begin();
    
                    double ans = stats[a][element] / stats[b][element];
    
                    result.push_back(ans);
    
                }
    
            }
    
            return result;
    
        }
    
    };  
  
---|---