# 图的最小生成树（Prim与Kruskal）

## 📖 内容来源
- [[Prim.c]]
- [[Kruskal.c]]

---

## 一、图的邻接矩阵存储（带权图）

```c
#include<stdio.h>
#include<stdlib.h>

typedef char VertexType;
typedef int EdgeType;

#define MAXSIZE 100
#define MAX 0x7fffffff  // 表示无穷大

typedef struct
{
    VertexType vertex[MAXSIZE];
    EdgeType arc[MAXSIZE][MAXSIZE];
    int vertex_num;
    int edge_num;
}Mat_Grph;
```

### 创建带权图（无向图）

```c
void create_graph(Mat_Grph* G)
{
    G->vertex_num = 9;
    G->edge_num = 15;

    G->vertex[0] = 'A'; G->vertex[1] = 'B';
    G->vertex[2] = 'C'; G->vertex[3] = 'D';
    G->vertex[4] = 'E'; G->vertex[5] = 'F';
    G->vertex[6] = 'G'; G->vertex[7] = 'H';
    G->vertex[8] = 'I';

    // 初始化：对角线为0，其余为MAX
    for(int i = 0; i < G->vertex_num; i++)
        for(int j = 0; j < G->vertex_num; j++)
            G->arc[i][j] = (i == j) ? 0 : MAX;

    // 带权边
    G->arc[0][1] = 10; G->arc[0][5] = 11;
    G->arc[1][2] = 18; G->arc[1][6] = 16; G->arc[1][8] = 12;
    G->arc[2][3] = 22; G->arc[2][8] = 8;
    G->arc[3][4] = 20; G->arc[3][6] = 24; G->arc[3][7] = 16; G->arc[3][8] = 21;
    G->arc[4][5] = 26; G->arc[4][7] = 7;
    G->arc[5][6] = 17;
    G->arc[6][7] = 19;

    // 补全对称矩阵
    for(int i = 0; i < G->vertex_num; i++)
        for(int j = 0; j < G->vertex_num; j++)
            G->arc[j][i] = G->arc[i][j];
}
```

---

## 二、Prim算法（从点出发）

**思想**：从任意一个顶点开始，每次选择连接"已选集合U"和"未选集合V-U"的**最小权值边**，将新顶点加入U，直到所有顶点都在U中。

**时间复杂度**：O(n²)（n为顶点数）
**适用场景**：稠密图

```
算法流程：
 任选一个顶点加入集合U
     ↓
 初始化侯选边数组weight[]（U中顶点到V-U各顶点的距离）
 和vex_index[]（记录每条侯选边的出发点）
     ↓
 循环 n-1 次（每次找一条边）：
     ↓
 在weight[]中找最小值 → 对应顶点k加入U
     ↓
 将weight[k]标记为0（表示已归入U）
     ↓
 更新侯选边：检查新顶点k到V-U各顶点的边
     ↓
 ┌─ 若权值 < 当前weight[j] → 更新weight[j], vex_index[j]=k
 └─ 否则 → 保持原侯选边不变
     ↓
 重复，直到所有顶点都在U中（得到n-1条边）
```

```c
void prim(Mat_Grph* G)
{
    int i, j, k;
    int min;
    int weight[MAXSIZE];    // 记录侯选边的权值
    int vex_index[MAXSIZE]; // 值表示出发点，下标表示到达点

    // 从顶点A（下标0）开始
    weight[0] = 0;
    vex_index[0] = 0;

    // 初始化：A点到其他点的权值
    for(i = 1; i < G->vertex_num; i++)
    {
        weight[i] = G->arc[0][i];
        vex_index[i] = 0;
    }

    for(i = 0; i < G->vertex_num - 1; i++)  // n-1条边
    {
        min = MAX;
        j = 0; k = 0;

        // 在所有侯选边中找到权值最小的
        while(j < G->vertex_num)
        {
            if(weight[j] != 0 && weight[j] < min)
            {
                min = weight[j];
                k = j;  // k记录新节点
            }
            j++;
        }

        printf("(%c,%c)\n", G->vertex[vex_index[k]], G->vertex[k]);
        weight[k] = 0;  // 标记已找到

        // 更新侯选边：加入新节点后，更新与新节点相连的边
        for(j = 0; j < G->vertex_num; j++)
        {
            if(weight[j] != 0 && G->arc[k][j] < weight[j])
            {
                weight[j] = G->arc[k][j];
                vex_index[j] = k;
            }
        }
    }
}
```

### Prim算法手动模拟过程

```
初始U = {A}
┌──────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│ 顶点  │  A  │  B  │  C  │  D  │  E  │  F  │  G  │  H  │  I  │
├──────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
│weight│  0  │  10 │  ∞  │  ∞  │  ∞  │  11 │  ∞  │  ∞  │  ∞  │
│parent│  A  │  A  │  A  │  A  │  A  │  A  │  A  │  A  │  A  │
└──────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
最小：B(10) → U={A,B}
更新侯选边：B与C(18), G(16), I(12)
...

最终结果（最小生成树的边）：
(A,B)=10, (B,I)=12, (I,C)=8, (C,D)=22, (D,E)=20, (E,H)=7, (E,F)=26...实际取决于权值比较
```

---

## 三、Kruskal算法（从边出发）

**思想**：将图中所有边按权值从小到大排序，依次选择最短边，若该边的两端点不属于同一集合，则加入生成树，否则丢弃。重复直到得到n-1条边。

**时间复杂度**：O(e log e)（e为边数）
**适用场景**：稀疏图

```
算法流程：
 创建邻接矩阵
     ↓
  提取所有边
     ↓
  按权值排序
     ↓
  初始化parent数组（并查集）
     ↓
  取最短边 → find()判断两端点是否属于同一集合
     ↓
  ├─ 是 → 丢弃
  └─ 否 → 加入生成树，合并集合
     ↓
  继续下一条边，直到得到n-1条边
```

```c
// 边结构体
typedef struct
{
    int begin;   // 起点下标
    int end;     // 终点下标
    int weight;  // 权值
}Edge;

// 交换边
void swap(Edge* edges, int i, int j)
{
    int temp;
    temp = edges[i].begin; edges[i].begin = edges[j].begin; edges[j].begin = temp;
    temp = edges[i].end; edges[i].end = edges[j].end; edges[j].end = temp;
    temp = edges[i].weight; edges[i].weight = edges[j].weight; edges[j].weight = temp;
}

// 冒泡排序对边按权值排序
void sortEdges(Edge edges[], int edge_num)
{
    for(int i = 0; i < edge_num; i++)
        for(int j = i+1; j < edge_num; j++)
            if(edges[i].weight > edges[j].weight)
                swap(edges, i, j);
}

// 并查集查找（查找顶点所属集合的根）
int find(int* parent, int index)
{
    while(parent[index] > 0)
        index = parent[index];
    return index;
}

// Kruskal算法
void Kruskal(Mat_Grph G)
{
    Edge edges[MAX_EDGE];  // 假设有MAX_EDGE大小的边数组
    int k = 0;

    // 提取所有边
    for(int i = 0; i < G.vertex_num; i++)
        for(int j = i+1; j < G.vertex_num; j++)
            if(G.arc[i][j] < MAX)
            {
                edges[k].begin = i;
                edges[k].end = j;
                edges[k].weight = G.arc[i][j];
                k++;
            }

    // 按权值排序
    sortEdges(edges, G.edge_num);

    int parent[MAXSIZE] = {0};  // 并查集数组
    int n, m;

    for(int i = 0; i < G.edge_num; i++)
    {
        n = find(parent, edges[i].begin);
        m = find(parent, edges[i].end);

        if(n != m)  // 不在同一集合，不构成环
        {
            parent[n] = m;  // 合并集合
            printf("(%c,%c) %d\n",
                G.vertex[edges[i].begin],
                G.vertex[edges[i].end],
                edges[i].weight);
        }
    }
}
```

### Kruskal算法手动模拟

```
所有边按权值排序：
(E,H)=7, (I,C)=8, (A,B)=10, (A,F)=11, (B,I)=12,
(B,G)=16, (D,H)=16, (F,G)=17, (B,C)=18, (G,H)=19,
(D,E)=20, (D,I)=21, (C,D)=22, (D,G)=24, (E,F)=26

过程：
1. (E,H)=7   √   parent[4]=7
2. (I,C)=8   √   parent[8]=2
3. (A,B)=10  √   parent[0]=1
4. (A,F)=11  √   parent[0]=1→... parent[1]=5
5. (B,I)=12  √   parent[1]=5→... 找到... parent[2]=8...合并
   ...直到得到8条边
```

---

## 四、Prim vs Kruskal 对比

| 特性 | Prim算法 | Kruskal算法 |
|------|---------|------------|
| **基本思想** | 从点出发，选最小邻接边 | 从边出发，选最小权值边 |
| **数据结构** | weight数组+vex_index数组 | 边数组+并查集(parent) |
| **时间复杂度** | **O(n²)** | **O(e log e)** |
| **适用场景** | **稠密图**（边多） | **稀疏图**（边少） |
| **实现难度** | 较简单 | 较复杂（需排序+并查集） |
| **贪心策略** | 每次选择到U的最小边 | 每次选择全局最小边 |

---

## 五、期末考试考点

### 📌 核心考点
1. **手动模拟Prim算法构造过程**（必考大题）
   - 画出每一步选择的边和当前U集合
2. **手动模拟Kruskal算法构造过程**（必考大题）
   - 按权值排序边，判断是否成环
3. **生成树的边数 = n - 1**（n为顶点数）
4. **Prim和Kruskal的适用场景对比**

### 📌 常见题型
- **选择题**：时间复杂度、适用场景对比
- **简答题**：给定图，手动模拟Prim/Kruskal求最小生成树
- **填空题**：最小生成树的边数和权值总和

### 📌 易错点
- Prim算法中weight[0]=0==表示已选==，不要混淆
- Kruskal中find()函数是并查集的查找，不是简单的数组访问
- 最小生成树**不唯一**（当有权值相等的边时）
- 生成树中**不能有环**
