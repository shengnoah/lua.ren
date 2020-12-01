---
layout: post
title: uva 11724 - evaluate the expression 
tags: [lua文章]
categories: [topic]
---
## contents

## Problem

每個變數皆為正整數，給定一個算術表達式和變數之間的大小關係，請問最少產生出來的答案為何。

## Sample Input

    
    
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

|

    
    
    3
    
    a+b*c
    
    2
    
    a>b
    
    c>b
    
    z*(x+y)
    
    3
    
    z>x
    
    x>y
    
    z>y
    
    a+b+c+a
    
    0  
  
---|---  
  
## Sample Output

    
    
    1
    
    2
    
    3

|

    
    
    Case 1: 4
    
    Case 2: 9
    
    Case 3: 4  
  
---|---  
  
## Solution

先使用 spfa 找到根據 greedy 策略分配的變數值，如果發生負環情況直接輸出 `-1`。

然後使用矩陣鏈乘積的 DP 方法進行找到最小答案。

    
    
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
    
    81
    
    82
    
    83
    
    84
    
    85
    
    86
    
    87
    
    88
    
    89
    
    90
    
    91
    
    92
    
    93
    
    94
    
    95
    
    96
    
    97
    
    98
    
    99
    
    100

|

    
    
    #include <stdio.h> 
    
    #include <vector>
    
    #include <queue>
    
    #include <string.h>
    
    #include <algorithm>
    
    using namespace std;
    
    char exp[1024], cond[1024];
    
    int val[26];
    
    int spfa(vector<int> g[]) {
    
    	queue<int> Q;
    
    	int inq[26] = {}, dist[26] = {};
    
    	int cnt[26] = {};
    
    	int u, v;
    
    	for (int i = 0; i < 26; i++) {
    
    		Q.push(i), dist[i] = 1;
    
    	}
    
    	while (!Q.empty()) {
    
    		u = Q.front(), Q.pop();
    
    		inq[u] = 0;
    
    		for (int i = 0; i < g[u].size(); i++) {
    
    			v = g[u][i];
    
    			if (dist[v] < dist[u] + 1) {
    
    				dist[v] = dist[u] + 1;
    
    				if (!inq[v]) {
    
    					inq[v] = 1, Q.push(v);
    
    					if (++cnt[v] > 26)
    
    						return 0;
    
    				}
    
    			}
    
    		}
    
    	}
    
    	for (int i = 0; i < 26; i++)
    
    		val[i] = dist[i];
    
    	return 1;
    
    }
    
    int used[512][512];
    
    long long dp[512][512];
    
    long long dfs(int l, int r) {
    
    	if (used[l][r])
    
    		return dp[l][r];
    
    	if (l == r)
    
    		return val[exp[l] - 'a'];
    
    	used[l][r] = 1;
    
    	long long &ret = dp[l][r];
    
    	ret = 1LL<<60;
    
    	int p = 0, test = 0;
    
    	for (int i = l; i <= r; i++) {
    
    		if (exp[i] == '(')	p++;
    
    		if (exp[i] == ')')	p--;
    
    		if (p == 0 && exp[i] == '+') {
    
    			ret = min(ret, dfs(l, i-1) + dfs(i+1, r));
    
    			test++;
    
    		}
    
    		if (p == 0 && exp[i] == '*') {
    
    			ret = min(ret, dfs(l, i-1) * dfs(i+1, r));
    
    			test++;
    
    		}
    
    	}
    
    	if (test == 0)
    
    		ret = min(ret, dfs(l+1, r-1));
    
    	return ret;
    
    }
    
    int main() {
    
    	int testcase, cases = 0;
    
    	int n;
    
    	scanf("%d", &testcase);
    
    	while (testcase--) {
    
    		scanf("%s", exp);
    
    		scanf("%d", &n);
    
    		vector<int> g[26];
    
    		for (int i = 0; i < n; i++) {
    
    			scanf("%s", cond);
    
    			g[cond[2]-'a'].push_back(cond[0]-'a');
    
    		}
    
    		printf("Case %d: ", ++cases);
    
    		if(!spfa(g)) {
    
    			puts("-1");
    
    		} else {
    
    			memset(used, 0, sizeof(used));
    
    			printf("%lldn", dfs(0, strlen(exp) - 1));
    
    		}
    
    	}
    
    	return 0;
    
    }
    
    3
    
    a+b*c
    
    2
    
    a>b
    
    c>b
    
    z*(x+y)
    
    3
    
    z>x
    
    x>y
    
    z>y
    
    a+b+c+a
    
    0
    
    */  
  
---|---