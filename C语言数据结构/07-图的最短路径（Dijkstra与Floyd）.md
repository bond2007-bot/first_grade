# 图的最短路径（Dijkstra与Floyd）

## 📖 内容来源
- [[Djikstra.c]]
- [[Floyd.c]]

---

## 一、图的邻接矩阵存储

```c
#include<stdio.h>

typedef char VertexType;
typedef int EdgeType;

#define MAXSIZE 100
#define MAX 0x10000  // 表示无穷大

typedef struct
{
    VertexType vertex[MAXSIZE];
    EdgeType arc[MAXSIZE][MAXSIZE];
    int vertex_num;
    int edge_num;
}Mat_Graph;
```

### 创建带权图

```c
void create_graph(Mat_Graph* G)
{
    G->vertex_num = 9;
    G->edge_num = 16;

    for(int i = 0; i < G->vertex_num; i++)
        G->vertex[i] = i;  // 顶点用数字0-8表示

    // 初始化：对角线为0，其他为MAX
    for(int i = 0; i < G->vertex_num; i++)
        for(int j = 0; j < G->vertex_num; j++)
            G->arc[i][j] = (i == j) ? 0 : MAX;

    // 设置边的权值
    G->arc[0][1] = 1;  G->arc[0][2] = 5;
    G->arc[1][2] = 3;  G->arc[1][3] = 7;  G->arc[1][4] = 5;
    G->arc[2][4] = 1;  G->arc[2][5] = 7;
    G->arc[3][4] = 2;  G->arc[3][6] = 3;
    G->arc[4][5] = 3;  G->arc[4][6] = 6;  G->arc[4][7] = 9;
    G->arc[5][7] = 5;
    G->arc[6][7] = 2;  G->arc[6][8] = 7;
    G->arc[7][8] = 4;

    // 补全对称矩阵（无向图）
    for(int i = 0; i < G->vertex_num; i++)
        for(int j = 0; j < G->vertex_num; j++)
            G->arc[j][i] = G->arc[i][j];
}
```

```
图示结构（带权无向图）：
       ①───1───②
      / │ \     │ \
     5  3  \    │  7
    /   │   \   │   \
   ③───1────④──5───⑤
    \   │   /│\     │
     7  3  2 │ 6   3 9
      \ │ /  │  \   │
       ⑥───3─⑦───2─⑧───4─⑨
```

---

## 二、Dijkstra算法（单源最短路径）

**思想**：按路径长度递增的次序逐步确定最短路径。

**时间复杂度**：O(n²)
**特性**：**不能处理负权边**

```c
// 返回下一个要观察的顶点
int choose(int distance[], int found[], int vertex_num)
{
    int min = MAX;
    int minpos = -1;
    for(int i = 0; i < vertex_num; i++)
    {
        if(distance[i] < min && found[i] == 0)
        {
            min = distance[i];
            minpos = i;
        }
    }
    return minpos;
}

void dij(Mat_Graph G, int begin)  // Dijkstra算法
{
    int found[MAXSIZE];    // 记录顶点是否被访问过
    int path[MAXSIZE];     // 记录路径（到达该顶点的上一个顶点）
    int distance[MAXSIZE]; // begin到每个顶点的最短距离

    // 初始化
    for(int i = 0; i < G.vertex_num; i++)
    {
        found[i] = 0;
        path[i] = -1;
        distance[i] = G.arc[begin][i];
    }
    found[begin] = 1;
    distance[begin] = 0;

    int next;
    for(int i = 0; i < G.vertex_num; i++)
    {
        next = choose(distance, found, G.vertex_num);
        if(next == -1) break;  // 剩余顶点不可达
        found[next] = 1;

        // 更新距离
        for(int j = 0; j < G.vertex_num; j++)
        {
            if(found[j] == 0)
            {
                if(distance[next] + G.arc[next][j] < distance[j])
                {
                    distance[j] = distance[next] + G.arc[next][j];
                    path[j] = next;
                }
            }
        }
    }

    // 输出结果
    for(int i = 1; i < G.vertex_num; i++)
    {
        printf("V%d -> V%d : %d\n", begin, i, distance[i]);
        printf("V%d <- ", i);
        int j = i;
        while(path[j] != -1)
        {
            printf("V%d <- ", path[j]);
            j = path[j];
        }
        printf("V%d\n", begin);
    }
}

int main()
{
    Mat_Graph G;
    create_graph(&G);
    dij(G, 0);
    return 0;
}
```

### Dijkstra手动模拟表

```
从V0出发，求到各顶点的最短路径：

┌─────────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┐
│  顶点   │  V0  │  V1  │  V2  │  V3  │  V4  │  V5  │  V6  │  V7  │  V8  │
├─────────┼──────┼──────┼──────┼──────┼──────┼──────┼──────┼──────┼──────┤
│初始化   │  0   │  1   │  5   │  ∞   │  ∞   │  ∞   │  ∞   │  ∞   │  ∞   │
│选V1     │      │      │  4*  │  8   │  6   │      │      │      │      │
│选V2     │      │      │      │  8   │  5*  │  11  │      │      │      │
│选V4     │      │      │      │  7*  │      │  8   │  11  │  14  │      │
│...      │      │      │      │      │      │      │      │      │      │
└─────────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┘
* 表示该顶点在此轮被选中（距离已确定）
```

---

## 三、Floyd算法（多源最短路径）

**思想**：动态规划，逐个顶点加入作为中转点，看是否能缩短路径。

**时间复杂度**：**O(n³)**
**特性**：可求任意两点间最短路径

```c
void floyd(Mat_Graph G)
{
    int path[MAXSIZE][MAXSIZE];
    int distance[MAXSIZE][MAXSIZE];

    // 初始化
    for(int i = 0; i < G.vertex_num; i++)
    {
        for(int j = 0; j < G.vertex_num; j++)
        {
            distance[i][j] = G.arc[i][j];
            path[i][j] = j;  // 初始：i到j直接走j
        }
    }

    // j:起始顶点  i:中转顶点  k:终止顶点
    for(int i = 0; i < G.vertex_num; i++)       // 中转顶点
    {
        for(int j = 0; j < G.vertex_num; j++)   // 起始顶点
        {
            for(int k = 0; k < G.vertex_num; k++) // 终止顶点
            {
                if(distance[j][k] > distance[j][i] + distance[i][k])
                {
                    distance[j][k] = distance[j][i] + distance[i][k];
                    path[j][k] = path[j][i];  // 更新路径
                }
            }
        }
    }

    // 输出结果
    for(int i = 0; i < G.vertex_num; i++)
    {
        for(int j = i + 1; j < G.vertex_num; j++)
        {
            printf("V%d->V%d weight:%d ", i, j, distance[i][j]);
            int k = path[i][j];
            printf("path:V%d", i);
            while(k != j)
            {
                printf("->V%d", k);
                k = path[k][j];
            }
            printf("->V%d\n", j);
        }
    }
}
```

### Floyd算法思想图解

```
考虑加入中转顶点i后，路径是否变短：

    j ────────→ k          原来的路径distance[j][k]
    
    j ─→ i ─→ k            加入i后的路径distance[j][i] + distance[i][k]
    
    如果 distance[j][k] > distance[j][i] + distance[i][k]
    则更新 distance[j][k] = distance[j][i] + distance[i][k]
    并更新路径 path[j][k] = path[j][i]
```

---

## 四、Dijkstra vs Floyd 对比

| 特性 | Dijkstra算法 | Floyd算法 |
|------|-------------|-----------|
| **解决问题** | 单源最短路径 | 多源最短路径（所有点对） |
| **时间复杂度** | O(n²) | **O(n³)** |
| **空间复杂度** | O(n) | O(n²) |
| **能否处理负权** | ❌ 不能 | 可以（无负权环） |
| **算法思想** | 贪心 | 动态规划 |
| **代码复杂度** | 中等 | 简单（三层循环） |

---

## 五、期末考试考点

### 📌 核心考点
1. **Dijkstra手动模拟**（必考大题）：
   - 填写distance数组的每一步变化
   - 写出从起点到各顶点的最短路径和距离
2. **Floyd手动模拟**（常考）：
   - 写出distance矩阵和path矩阵的每一步变化
3. **算法对比**：Dijkstra vs Floyd 适用场景

### 📌 常见题型
- **选择题**：时间复杂度、能否处理负权、算法类别
- **简答题**：
  - 给出图，用Dijkstra求单源最短路径（填表）
  - 给出图，用Floyd求任意两点最短路径
  - 写出Dijkstra算法每一步的distance和path数组
- **算法题**：Dijkstra或Floyd的代码实现（阅读/填空）

### 📌 易错点
1. **Dijkstra不能处理负权边**（某些负权边可能导致已确定的最短路径不正确）
2. **Floyd的三层循环顺序**：中转顶点在最外层
3. **无向图的邻接矩阵对称**，注意更新
4. **路径输出**：Dijkstra用path数组逆向回溯，Floyd用path矩阵正向输出
