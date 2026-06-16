# 图的遍历（BFS与DFS）

## 📖 内容来源
- [[bfs.c]]
- [[dfs.c]]

---

## 一、图的邻接矩阵存储结构

```c
#include<stdio.h>

typedef char VertexType;
typedef int EdgeType;

#define MAXSIZE 100

typedef struct
{
    VertexType vertex[MAXSIZE];      // 顶点数组
    EdgeType arc[MAXSIZE][MAXSIZE];  // 邻接矩阵，1表示有边，0表示无边
    int vertex_num;                  // 顶点数
    int edge_num;                    // 边数
}Mat_Grph;

int visited[MAXSIZE];  // 记录顶点是否被访问
```

---

## 二、创建图（无向图）

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

    // 邻接矩阵全部初始化为0
    for(int i = 0; i < G->vertex_num; i++)
        for(int j = 0; j < G->vertex_num; j++)
            G->arc[i][j] = 0;

    // 建立边（上三角）
    G->arc[0][1] = 1; G->arc[0][5] = 1;   // A-B, A-F
    G->arc[1][2] = 1; G->arc[1][6] = 1; G->arc[1][8] = 1;  // B-C, B-G, B-I
    G->arc[2][3] = 1; G->arc[2][8] = 1;   // C-D, C-I
    G->arc[3][4] = 1; G->arc[3][6] = 1; G->arc[3][7] = 1; G->arc[3][8] = 1;  // D-E, D-G, D-H, D-I
    G->arc[4][5] = 1; G->arc[4][7] = 1;   // E-F, E-H
    G->arc[5][6] = 1;   // F-G
    G->arc[6][7] = 1;   // G-H

    // 补全对称矩阵（无向图）
    for(int i = 0; i < G->vertex_num; i++)
        for(int j = 0; j < G->vertex_num; j++)
            G->arc[j][i] = G->arc[i][j];
}
```

> **图示结构：**
> ```
>     A ─── B ─── C ─── D ─── E
>     │     │     │     │     │
>     F ─── G ─── H     I     │
>           │     │           │
>           └─────┴───────────┘
> ```

---

## 三、广度优先搜索（BFS）

BFS类似于树的**层序遍历**，使用**队列**辅助实现。

```c
// 顺序队列
typedef struct
{
    int data[MAXSIZE];
    int front;
    int rear;
}Queue;

void InitQueue(Queue* Q)
{
    Q->front = 0;
    Q->rear = 0;
}

int EmptyQueue(Queue* Q)
{
    return Q->front == Q->rear;
}

void EnQueue(Queue* Q, int x)
{
    Q->data[Q->rear] = x;
    Q->rear++;
}

int DeQueue(Queue* Q)
{
    int x = Q->data[Q->front];
    Q->front++;
    return x;
}

// 广度优先搜索
void bfs(Mat_Grph G, int start)
{
    Queue Q;
    InitQueue(&Q);

    visited[start] = 1;
    EnQueue(&Q, start);
    printf("%c\n", G.vertex[start]);  // 访问起始顶点

    while(!EmptyQueue(&Q))
    {
        int i = DeQueue(&Q);  // 取出队头顶点
        // 遍历所有邻接点
        for(int j = 0; j < G.vertex_num; j++)
        {
            if(G.arc[i][j] == 1 && visited[j] == 0)
            {
                printf("%c\n", G.vertex[j]);  // 访问
                visited[j] = 1;
                EnQueue(&Q, j);
            }
        }
    }
}
```

### BFS执行过程
```
从A开始：
Queue: [A] → 出队A，访问A的邻接点B、F入队
Queue: [B, F] → 出队B，访问B的邻接点C、G、I入队
Queue: [F, C, G, I] → 出队F，访问F的邻接点E入队（G已访问）
...
```

---

## 四、深度优先搜索（DFS）

DFS类似于树的**先序遍历**，使用**递归**实现。

```c
// 深度优先搜索DFS
void dfs(Mat_Grph G, int i)
{
    visited[i] = 1;  // 标记已访问
    printf("%c\n", G.vertex[i]);  // 输出当前顶点

    // 寻找所有邻接点
    for(int j = 0; j < G.vertex_num; j++)
    {
        if(G.arc[i][j] == 1 && visited[j] == 0)
        {
            dfs(G, j);  // 递归访问
        }
    }
}
```

### DFS执行过程
```
从A开始：
A → B → C → D → E → F → G → H → I
     ↓
回溯过程：I无未访问邻接→回溯H→回溯G→回溯F→回溯E→D还有未访问...G已访→H已访→I已访→结束
```

---

## 五、主函数

```c
int main()
{
    Mat_Grph G;
    create_graph(&G);

    for(int i = 0; i < G.vertex_num; i++)
        visited[i] = 0;

    // BFS
    bfs(G, 0);

    // DFS
    // dfs(G, 0);

    return 0;
}
```

---

## 六、BFS与DFS对比

| 特性 | BFS | DFS |
|------|-----|-----|
| **数据结构** | 队列 | 栈（递归本质是栈） |
| **空间复杂度** | O(V) | O(V)（递归栈） |
| **最短路径** | 能找到无权图的最短路径 ✅ | 不能 ❌ |
| **遍历顺序** | 按层扩展（辐射状） | 沿纵深探索 |
| **应用场景** | 最短路径、层次遍历 | 连通分量、拓扑排序 |

---

## 七、期末考试考点

### 📌 BFS考点
1. **BFS算法过程**（手动模拟）
2. **BFS借助的数据结构**（队列）
3. **BFS可解决的经典问题**：无权图最短路径
4. **邻接矩阵/邻接表BFS的时间复杂度**：O(V²)/O(V+E)

### 📌 DFS考点
1. **DFS算法过程**（手动模拟）
2. **DFS借助的数据结构**（栈/递归）
3. **DFS可解决的问题**：连通分量、拓扑排序、图的二分性
4. **DFS生成树/森林**

### 📌 常见题型
- **选择题**：BFS/DFS使用的数据结构、时间复杂度
- **填空题**：给定图写出BFS/DFS遍历序列
- **简答题**：BFS和DFS的区别及适用场景
- **算法题**：基于图的遍历解决具体问题（如判断是否有环、连通分量）

### 📌 核心概念

| 概念 | 说明 |
|------|------|
| **时间复杂度（邻接矩阵）** | BFS = O(V²)，DFS = O(V²) |
| **时间复杂度（邻接表）** | BFS = O(V+E)，DFS = O(V+E) |
| **空间复杂度** | 均为 O(V)，visited数组 |
| **连通分量** | 每个连通分量调用一次BFS/DFS |
