---
title: Shortest path--Dijkstra, Bellman-ford and SPFA
date: 2021-02-28 00:11:20
tags: [""Shortest Path",Dijkstra", "SPFA"]
---
# Shortest path algorithm
## 1. Dijkstra
Given a graph ``G=(V, E)`` with positive edge and a source edge ``S``, how to find the shortest path/distance from ``S`` to any vertex? 

Dijkstra's algorithm works as follow. Given a **Known set K** and distrance function $D_S(x)$ for $x \in K$, initialized with $K=\{S\}$, $D_S(S) = 0$.

By induction, suppose we have already known shortest distance between $S$ and vertices in $K$. Suppose vertices in set $N$ are adjacent to set $K$. We can find the distance $d_S(y)$ for $y\in N$ by simply stretching from $K$ to $N$.

If $y^*=\argmin_{y \in N} d_S(y)$, the we claim $D_s(y^*)=d_s(y^*)$. The reason is fairly simple: if there is a better path s -- x -|- y -- y*, where $\{s, x\} \subseteq K$, $\{y, y^*\} \subseteq N$, obviously 

$$
\begin{aligned}
D_S(y^*) &= D_S(x) + d(x,y) + D_y(y^*) \\
    &= D_S(y) + D_y(y^*) \\
    &\ge D_S(y^*)
\end{aligned} \tag{1}
$$
By premisis of the problem, the equation holds true only when $D_S(y)=D_S(y^*)$ and $D_y(y^*)=0$. Obviously, this can take place only when $y=y^*$. Therefore we update by letting $N'=N+\{y^*\}$, $D_S(y^*)=d_S(y^*)$

```cpp
// Dijkstra with heap
#include<cstdio>
#include<iostream>
#include<vector> 
#include<queue>

#define ll long long 
#define INF 0x3f3f3f3f
using namespace std;

const int maxn = 1009;

using namespace std;
struct edge{
    int x, d;
    bool operator <(edge b){
        return b.x < x;
    }
};
// Adjacent graph
vector<edge> vec[maxn];
int dis[maxn];  // D_S(x)
bool vis[maxn]; // vis[x] = 1 if x in K

int main(){
    int n, m;
    while( scanf("%d %d",&n, &m) && n + m){
        for(int i = 0; i <=n; ++i){
            dis[i] = INF;
            vis[i] = 0;
            vec[i].clear();
        }
        int s, t, d;
        for(int i = 0; i < m; ++i){
            scanf("%d %d %d %d", &s, &t, &d, &p);
            vec[s].push_back( (edge){t, d} );
            vec[t].push_back( (edge){s, d} );
        }
        int st, ed;
        cin >> st >> ed;
 
        // start
        priority_queue<edge> que;
        que.push( (edge){st, 0} );

        while(!que.empty()){
            edge top = que.top();
            que.pop();
            // If y is not in K, it's y*
            if( !vis[top.x] ){
                vis[top.x] = 1;         // Add to K
                dis[top.x] = top.d;     // D_S(x) = d_S(x)

                // Update N and d_S(y)
                for(int i = 0; i < vec[top.x].size(); ++i){
                    edge &e = vec[top.x][i];
                    if( vis[edge.x])
                        continue;
                    if( top.d + e.d < dis[e.x] )
                        que.push( (edge){e.x, top.d + e.d} );
                }
            }
            if(vis[ed])
                break;
        }
        printf("%d %d\n", dis[ed]);
    }
}
```

**How to find shortest path ?**

Introduce an extra array ``pa[MAXN]``, when $y^*$ is added into $K$, let $pa[y^*]$ be the closet $x\in K$.
Then we can backtrack the path from ``ed`` back to the beginning.

## 2. Bellman-ford
If the graph contains negative edges, the equation (1) won't hold true any longer, since $d_S(y, y^*)$ is no longer non-negative.

Bellman-ford is proposed to solve this problem. It's more general than Dijkstra. The pseudo-code  is:

```
dis[|V|] = {INF, INF, ..., INF}
dis[S] = 0
for i = 1 to |V|-1
    for e in E
        dis[e.ed] = min( dis[e.ed], dis[e.s] + e.dis)
```

Proof: $\forall v_i \in V$, if there are shortest paths (implies $v_i$ is reachable from $S$), consider the shortest path $P_i=[\phi_1, \phi_2, ..., \phi_k]$ from $S$ to $v_i$, which means $\phi_1=S$ and $\phi_k=v_i$. 

No vertix $v_j \in V$ can appear twice in the path. Otherwise, there must be a path $P'=[\Phi_1, ..., \Phi_k]$. 

- If the length of $P'$ is positive, then obviously we chan trim the path to shorten length, leading to contradiction that $P_i$ is not the shortest path from $S$ to $v_i$. 
- If the length of $P'$ is 0, it won't hurt the fact that this is the shortest path. We can cherry-pick the simplified path without repeated vertices, the two shortest paths have equal shortest distances.
- If the length of $P'$ is negative, a negative loop is promised to exist. In that case, no shortest paths can exist.

Now we can safely claim that any shortest path from $S$ contains # of vertices less than $|V|$. In fact, there are at most |V|-1 edges between $S$ and termination. In the first round of brute relaxation for all edges, for $\Phi_1$, $D_S(\Phi_1) = d_S(\Phi_1)$ is attained when edge $e=(S,\Phi_1)$ is relaxed (Otherwise there must be a shorter path from $S$ to $\Phi_1$). By induction we know that $D_S(\Phi_2)=D_S(\Phi_1) + d(\Phi_1, \Phi_2)=d_S(\Phi_2)$, then so on forward.

If relaxation is made for all edges for |V|-1 rounds, all shortest paths will be completely updated, then $d_S(v)$ for all $v\in V$ turns out to be equal to $D_S(v)$. If $d_S(v)=\infty$, obvious it means $v$ is unreachable.

The complexity is $O(|V|\cdot|E|)$.

## 3. Floyd algorithm
Floyd algorithm is the generalization of Bellman-ford algorithm for all pais of points. We can prove the theorem by recursion. Consider the ${ n \choose 2}$ pairs. For $i\rightarrow j$, if the shortest path exists, let the path be $P_{ij}=\{v_1, v_2, ..., v_m\}$, where $v_1=i$ and $v_m=j$. Define $max(P_{ij})=\max_{l=1}^m v_l$. 

Now we prove the correctness by recursion. Given k, suppose we have already found shortest distance for all shortest paths that includes vertex with index less than $k-1$. Next, consider all pairs of $(i,j)$ with $max(P_{ij}) = k$. Let's denote $k^+$ for the vertex in the right-hand side of $k$ in path $P_{ij}$, and $k^-$ for the reverse. There are two cases:

- k is on double-sides. Then $D(i,k)=D(i, k^-) + D(k^-, k)$ or $D(k,i)=D(k, k^+) + D(k^+, j)$. $D(k^-, k)$ or $D(k, k^+)$ are edges, known in advance. The rest parts are obviously subproblems with $max(P')<k$. By recursion, we can find the shortest distance.
- k is in the middle. Then $D(i,j) = D(i,k^-) + D(k^-, k) + D(k, k^+) + D(k^+, j)$. $D(k^-, k)$ and $D(k, k^+)$ are edges known in advance, and the rest parts are subproblem with $max(P')<k$. Obviously, we can find the shortest distance.

Finally, consider the terminal condition. $k=1$ corresponds to the only path $P_{11}$. It's surely solved, as $D(1,1)=0$ (otherwise there would be negative circles.)


In practice, we can write the code by induction, from $k=1$ to $|V|$.

```
dp[|V|][|V|] = G[|V|][|V|]       
for k = 1 to |V|              // Bottom up for shortest paths max(P) <= k
    for i = 1 to |V|            
        for j = 1 to |V|        
            if dis[k][j] != INF
                dp[i][j] = min(dp[i][j], dp[i][k] + dp[k][j])
```
**Key to understanding:** Solvable paths are readily solved, don't be confused by asking those pairs not currently solvable. 