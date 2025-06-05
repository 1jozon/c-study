好的！你想要学习如何用 C++ 实现 **Dijkstra 算法**，它是一种用于求解 **单源最短路径（Single Source Shortest Path, SSSP）** 的经典算法，适用于 **带权有向图或无向图**，且权重必须为非负数。

---

## ✅ 一、问题描述

给定一个图 `G = (V, E)`，以及一个起点 `s`，找出从 `s` 到其他所有顶点的最短路径。

---

## ✅ 二、实现方式

我们使用：

- 邻接表（Adjacency List）表示图
- 优先队列（`priority_queue`）进行贪心选择
- 距离数组记录每个节点到起点的最短距离

---

## ✅ 三、完整 C++ 示例代码

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <climits>

using namespace std;

// 定义边的结构体：目标节点 to，权重 weight
typedef pair<int, int> Edge; // {to, weight}

void dijkstra(int start, vector<vector<Edge>>& graph, vector<int>& dist) {
    int n = graph.size(); // 顶点数量

    // 使用最小堆，按距离排序 {distance, node}
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;

    // 初始化起点
    dist[start] = 0;
    pq.push({0, start});

    while (!pq.empty()) {
        int currDist = pq.top().first;
        int u = pq.top().second;
        pq.pop();

        // 如果当前节点已经找到了更短路径，跳过
        if (currDist > dist[u]) continue;

        // 遍历当前节点的所有邻接边
        for (auto& edge : graph[u]) {
            int v = edge.first;
            int weight = edge.second;

            // 如果找到更短路径，更新并加入队列
            if (dist[u] + weight < dist[v]) {
                dist[v] = dist[u] + weight;
                pq.push({dist[v], v});
            }
        }
    }
}

int main() {
    int n = 5; // 节点数编号 0 ~ 4

    // 图的邻接表表示：graph[u] 包含所有从 u 出发的边
    vector<vector<Edge>> graph(n);

    // 添加边（u -> v），权重 w
    graph[0].push_back({1, 10});
    graph[0].push_back({3, 5});
    graph[1].push_back({2, 1});
    graph[1].push_back({3, 2});
    graph[2].push_back({4, 4});
    graph[3].push_back({1, 3});
    graph[3].push_back({4, 9});
    graph[4].push_back({0, 7});
    graph[4].push_back({2, 6});

    int start = 0; // 起点
    vector<int> dist(n, INT_MAX); // 初始化为无穷大

    dijkstra(start, graph, dist);

    cout << "从节点 " << start << " 到各节点的最短距离：" << endl;
    for (int i = 0; i < n; ++i) {
        cout << "节点 " << i << ": " << (dist[i] == INT_MAX ? "不可达" : to_string(dist[i])) << endl;
    }

    return 0;
}
```

---

## ✅ 四、输出示例

运行以上程序，输出类似如下内容：

```
从节点 0 到各节点的最短距离：
节点 0: 0
节点 1: 8
节点 2: 9
节点 3: 5
节点 4: 14
```

---

## ✅ 五、复杂度分析

- 时间复杂度：`O((V + E) * log V)`，其中 `V` 是顶点数，`E` 是边数。
- 空间复杂度：`O(V + E)`

---

## ✅ 六、扩展功能建议

你可以根据需要添加以下功能：

- 打印最短路径本身（而不仅仅是距离）
- 支持输入自定义图（例如从文件或控制台读取）
- 支持检测不可达节点

---

如果你希望我继续教你如何打印具体路径，或者封装成类，请告诉我 😄