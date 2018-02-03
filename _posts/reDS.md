---
title: 重拾数据结构
date: 2017-08-02
tags:
	- Algo
	- Math
	- DataStruct
	- C/C++
---

> 在大三到大四过渡期中, 从四月到七月, 经历过几场面试, 找实习. (如果八月份拿到[真.offer]的话我也想把这段经历记录下来) 结果很悲剧, 觉得方向不是什么障碍, 基础比较重, 所以要重拾数据结构不涉及算法具体实现, 因为是重拾, 所以先前有一定基础, 只要有一点提示便能唤醒无限回忆. 这里主要是记录各种数据结构的结构体, 这是经过重重思维过程得出的精华, 十分有价值. 至于具体实现和讲解日后回会以链接形式提供, 这里只提供一个思维树, 建立一个数据机构的思维体系, 后续更新欢迎关注 [GitHub](https://github.com/mk43/Algo-Math).

<!-- more -->

### 1 线性表

##### 1.1 动态分配空间

```C
typedef struct {
	ElemType * elem;
	int length;
	int listsize;
} SqList;
```

##### 1.2 线性链表

```C
typedef struct LNode {
	ElemType data;
	struct LNode * next;
} LNode, * LinkList;
```

##### 1.3 静态链表

```C
typedef struct {
	ElemType data;
	int cur;	// 游标指向下一个元素的数组下标
} component, SlinkList[MAXSIZE];
```

##### 1.4 循环链表 & 双向链表

```C
typedef struct DuLNode{
	ElemType data;
	struct DuLNode * prior;
	struct DuLNode * next;
} DuLNode, * DuLinkList;
```

### 2 栈和队列

##### 2.1 栈

```C
typedef struct {
	SELemType * base;
	SElemType * top;##### 
	int stacksize;
} SqStack;
```

> 应用: 主要是利用先进后出的特性
> 
> - 数制转换
> 
> - 括号匹配检测
> 
> - 行编辑程序
> 
> - 迷宫非递归求解
> 
> - 表达式求值
> 
> - Hanoi塔问题


##### 2.2 队列

- 链队列
	
```C
typedef struct QNode {
	QElemType data;
	struct QNode * next;
} QNode, * QueuePtr;
typedef struct {
	QueuePtr front;
	QueuePtr rear;
} LinkQueue;
```
	
- 循环队列
	
```C
typedef struct {
	QElemType * base;
	int front;
	int rear;
} SqQueue;
```

### 3 串

##### 3.1 定长串

```C
typedef unsigned char SString[MAXSTRLEN + 1];
SString s;
```

##### 3.2 变长

- 堆分配存储

```C
typedef struct {
	char * ch;
	int length;
}HString;
```
	
- 块链存储
	
```C
typedef struct Chunk{
	char ch[CHUNKSIZE];
	struct Chunk * next;
} Chunk;
typedef struct {
	Chunk * head;
	Chunk * tail;
	int curlen;
} LString;
```

> 应用
> 
> - 子串定位 (KMP)
> 

### 4 数组和广义表

##### 4.1 数组顺序存储

```C
typedef struct {
	ElemType * base;
	int dim;    // 数组维数
	int * bounds;    // 维界基址
	int * constants;    // 印象函数常量基址
} Array;
```

##### 4.2 矩阵

> 讨论稀疏矩阵的存储
	
- 三元顺序表
	
```C
typedef struct {
	int i;
	int j;
	ElemType e;
} Triple;
typedef struct {
	Triple data[MAXSIZE + 1];
	int mu;    // 行
	int nu;    // 列
	int tu;    // 非零个数
}TSMatrix;
```
	
- 行逻辑链接
	
```C
typedef struct {
	Triple data[MAXSIZE + 1];
	int rpos[MAXRC + 1];    // 各行第一个非零元素位置表
	int mu, nu, tu;
} RLSMatrix;
```
	
- 十字链表
	
```C
typedef struct OLNode {
	int i;
	int j;
	ElemType e;
	struct OLNode * right;    // 该非零元素所在行的右链域
	struct OLNode * down;    // 该非零元素所在列的下链域
} OLNode, * OLink;
typedef struct {
	OLink * rhead;    // 行链表头指针地址
	OLink * chead;    // 列链表头指针地址
	int mu, nu, tu;
} CrossLink;
```

##### 4.3 广义表
	
> 表中有表
	
- 头尾链表存储
	
```C
typedef enum { ATOM, LIST } ElemTag;    // 0 : 原子, 1 : 子表
typedef struct GLNode {
	ElemTag tag;
	union {
		AtomType atom;
		struct {
			struct GLNode * hp;    // 表头
			struct GLNode * tp;    // 表尾
		} ptr;    // 表节点指针域
	};
} * GList;
```
	
- 扩展线性链表存储
	
```C
typedef enum { ATOM, LIST } ElemTag;    // 0 : 原子, 1 : 子表
typedef struct GLNode {
	ElemTag tag;
	union {
		AtomType atom;
		struct {
			struct GLNode * hp;    // 表头
		} ptr;    // 表节点指针域
	};
	struct GLNode * tp;    // 表尾, 相当于 next.
} * GList;
```

### 5 树和二叉树

##### 5.1 二叉树存储结构

- 顺序存储结构
	
> 数组, 利用下标寻址
	
```C
typedef TElemType SqBiTree[MAX_TREE_SIZE];
```
	
- 链式存储结构
	
```C
typedef struct BiTNode {
	TElemType data;
	struct BiTNode * lchild;
	struct BiTNode * rchild;
} BiTNode, * BiTree;
```

##### 5.2 遍历二叉树

- 先序遍历
	
> 根节点 -> 左子树 -> 右子树
	
- 中序遍历
	
> 左子树 -> 根节点 -> 右子树
	
- 后序遍历
	
> 左子树 -> 右子树 -> 根节点

> 算数表达式 a + b * (c - d) - e / f
> 
> 前缀表达式-先序遍历(逆波兰 : - + a * b - cd / ef)
> 
> 中缀表达式-中序遍历(原表达式 : a + b * (c - d) - e / f)
> 
> 后缀表达式-后续遍历(逆波兰式 : abcd - * + ef / -)


##### 5.3 线索二叉树

> 保存比遍历过程中的节点相关性结果
> 
> 前驱后继节点和左右孩子指示
> 
> | lchild | LTag | data | RTag | rchild |
> |--------|------|------|------|--------|
> 
> LTag 0 : lchild 域指示左孩子 1 : lchild 域指示前驱节点
> 
> RTag 0 : rchild 域指示右孩子 1 : rchild 域指示后继节点

```C
typedef enum PointerTag {Link, Thread};    // 0 : 指针 1 : 线索
typedef struct BiThrNode {
	TElemType data;
	struct BiThrNode * lchild;
	struct BiThrNode * rchild;
	PointerTag LTag;
	PointerTag RTag;
} BiThrNode, * BiThrTree;
```

##### 5.4 树和森林

- 双亲表示法

```C
typedef struct PTNode {
	TElemType data;
	int parent;
} PTNode;
typedef struct {
	PTNode nodes[MAX_TREE_SIZE];
	int r;    // 根的位置
	int n;    // 节点数
} PTree;
```
	
- 孩子表示法
	
```C
typedef struct CTNode {    // 孩子节点
	struct CTNode * next;
	int child;
} * ChildPtr;
typedef struct {
	TElemType data;
	ChildPtr firstchild;    // 孩子链表头指针
} CTBox;
typedef struct {
	CTBox nodes[MAX_TREE_SIZE];
	int r;    // 根的位置
	int n;    // 节点数
} CTree;
```
	
- 孩子兄弟表示法
	
```C
typedef struct CSNode {
	ElemType data;
	struct CSNode * firstchild;
	struct CSNode * nextsibling;
} CSNode, * CSTree;
```

##### 5.5 二叉树和森林互换
	
- 森林转换成二叉树
	
> 左孩子右兄弟(左右是对二叉树而言, 孩子兄弟是对森林而言, 下面同理) 
	
- 二叉树转换成森林

> 左孩子转换成孩子, 右孩子转换成兄弟

##### 5.6 树和森林遍历
	
- 先序遍历森林
	
> 1. 第一棵树的根
> 
> 2. 先序遍历第一棵树中根节点的子树森林
>
> 3. 先序遍历除第一棵树剩余的树构成的森林
	
- 中序遍历森林
	
> 1. 中序遍历第一棵树中根节点的子树森林
> 
> 2. 第一棵树的根
>
> 3. 中序遍历除第一棵树剩余的树构成的森林
	
### 7 图

##### 7.1 图的存储结构

- 数组表示法
	
```C
typedef enum {DG, DN, UDG, UDN} GraphKind;    // {有向图, 有向网, 无向图, 无向网}
typedef struct ArcCell {
	VRType adj;    // 顶点相关类型. 无权图 : 1/0 表示相邻与否 带权图 : 权值信息
	InfoType * info;    // 该弧相关的指针
} ArcCell, AdjMatrix[MAX_VERTEX_NUM][MAX_VERTEX_NUM];
typedef struct {
	VertexType vexs[MAX_VERTEX_NUM];    // 顶点向量
	AdjMatrix arcs;    // 邻接矩阵
	int vexnum;    // 顶点数
	int arcnum;    // 弧数
	GraphKind kind;    // 图的种类标志
} MGraph;
```
	
- 邻接表
	
>  表节点
> 
> | adjvex | nextarc | info |
> |--------|---------|------|
> 
> 头结点
> 
> | data | firstarc |
> |------|----------|
	
```C
typedef struct ArcNode {
	int adjvex;    // 该弧所指向顶点位置
	struct ArcNode * nextarc;    // 指向下一条弧的指针
	InfoType * info;    // 该弧相关信息的指针
} ArcNode;
typedef struct VNode {
	VertexType data;    // 顶点信息
	ArcNode * firstarc;    // 指向第一条依附该顶点的弧的指针
} VNode, AdjList[MAX_VERTEX_NUM];
typedef struct {
	AdjList vertices;
	int vexnum;    // 顶点数
	int arcnum;    // 弧数
	int kind;    // 种类标记
} ALGraph;
```	
	
- 十字链表
	
> 弧节点
> 
> | tailvex | headvex | hlink | tlink | info |
> |---------|---------|-------|-------|------|
> 
> 顶点节点
> 
> | data | firstin | firstout |
> |------|---------|----------|
>
	
```C
typedef struct ArcBox {
	int tailvex;    // 该弧的尾顶点位置
	int headvex;    // 该弧的头顶点位置
	struct ArcBox * hlink;    // 弧头相同的弧的链域
	struct ArcBox * tlink;    // 弧尾相同的弧的链域
	InfoType * info;    // 该弧相关信息指针
} ArcBox;
typedef struct VexNode {
	VertexType data;
	ArcBox * firstin;    // 指向该节点第一条入弧
	ArcBox * firstout;    // 指向该节点第一条出弧
} VexNode;
typedef struct {
	VexNode xlist[MAX_VERTEX_NUM];    // 表头向量
	int vexnum;    // 有向图的当前顶点数
	int arcnum;    // 有向图的当前弧数
} OLGraph;
```
	
- 邻接多重表
	
> 每一条边用一个节点表示
> 
> | mark | ivex | ilink | jvex | jlink | info |
> |------|------|-------|------|-------|------|
> 
> 每个顶点用一个节点表示
> 
> | data | firstedge |
> |------|-----------|
>
	
```C
typedef enum {unvisited, visited} VisitIf;
typedef struct EBox {
	VisitIf mark;    // 访问标记
	int ivex;    // 依附顶点位置
	int jvex;    // 依附顶点位置
	struct EBox * ilink;    // 依附顶点的下一边
	struct EBox * jlink;    // 依附顶点的下一边
	InfoType * info;    // 该边的信息指针
} EBox;
typedef struct VexBox {
	VertexType data;
	EBox * firstedge;    // 指向第一条依附该顶点的边
} VexBox;
typedef struct {
	VexBox adjmulist[MAX_VERTEX_NUM];
	int vexnum;    // 无向图的顶点数
	int edgenum;    // 无向图的边数
} AMLGraph
```
	
##### 7.2 图的遍历

- 深度优先搜索
	
> 以迷宫为例子(面试中被问到, 印象比较深刻). 深度优先就是一条路走到黑, 所以返回的第一条路径不保证是最优解. 
>
	
```C
Boolean visited[MAX];    // 访问标志数组
Status (* VisiteFunc) (int v);    // 函数变量
	
void DFSTraverse(Graph G, Status (* Visit)(int v)) {    // 深度优先遍历
	VisitFunc = Visit;    // 使用全局变量 VisitFunc, 使 DFS 不必设置函数指针参数
	for (v = 0; v < G.vexnum; ++v) {
		visited[v] = FALSE;    // 访问数组标志初始化
	}
	for (v = 0; v < G.vexnum; ++v) {
		if (!visited[v]) {
			DFS(G, v);    // 对未访问的顶点调用 DFS
		}
	}
}
	
void DFS(Graph G, int v) {    // 从第 v 个顶点出发递归的深度优先遍历图 G.
	visited[v] = TRUE;
	VisitFunc(v);    // 访问第 v 个顶点
	for (w = FirstAdjVex(G, v); w >= 0; w = NextAdjVex(G, v, w)) {
		if (!visited[w]) {
			DFS(G, w);    // 对 v 的尚未访问的邻接顶点 w 递归调用 DFS.
		}
	}
}
```

- 广度优先搜索

> 有点层序遍历的意思, 在遍历完所有情况下(可以进行算法优化, 对有些情况进行舍弃)可以得出最优解. 

```C
void BFSTraverse(Graph G, Status (* Visit) (int v)) {
	for (v= 0; v < G.vexnum; ++v) {
		visited[v] = FALSE;
	}
	InitQueue(Q);
	for (v = 0; v < G.vexnum; ++v) {
		if (!visited[v]) {
			visited[v] = TRUE;
			Visit(v);
			EnQueue(Q, v);
			while (!QueueEmpty(Q)) {
				DeQueue(Q, u);
				for (w = FirstAdjVex(G, u); w >= 0; w = NextAdjVex(G, u, w)) {
					// w 为 u 尚未访问的邻接顶点
					if (!visited[w]) {
						visited[w] = TRUE;
						Visit(w);
						EnQueue(Q, w);
					} // if
				} // for
			} // while
		} // if
	} // for
} // BFSTraverse
```
	

// 以下待填

##### 7.3 图的连通性问题

- 无向图的连通分量和生成树
- 有向图的强连通分量

##### 7.4 最小生成树

- [Prim 算法 O(n2)](https://en.wikipedia.org/wiki/Prim%27s_algorithm)

  ![没图戳 wikipedia 或 fitzeng.org](PrimAlgDemo.gif)
  
  > 以点为主: 主要是分为两个集合, 一个是已加入的节点, 另一个是未加入节点. 在未加入的节点集合中找到一个离已加入集合最近的节点加入. 直至所有节点被加入. 

- [Kruskal 算法 O(eloge)](https://en.wikipedia.org/wiki/Kruskal%27s_algorithm)

  ![没图戳 wikipedia 或 fitzeng.org](MST_kruskal_en.gif)
  
  > 以边为主: 初始条件是把图的所有边去除变成 V 个连通图. 然后每次找一条代价最小的边加入, 确保每加入一条边连通图个数都减少一个(也就是确保无环路)。直至成一个连通图时就是最小生成树.  

##### 7.5 最短路径

- [Dijkstra 算法 O(n3)](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm)

  ![没图戳 wikipedia 或 fitzeng.org](Dijkstra_Animation.gif)
  
  > 主要是维护一个表和一个已加入路径集合, 表记录从原点到每一个点的当前最小权值. 如果已加入路径集合中的点通过某条路径对未加入集合中的点的最小权值有影响则更新该节点权值. 最后在每次更新完成后判断目前未加入集合中的最小权值节点加入集合, 再对该节点的边所达的节点做如上判断. 最终可以求出起点到每一个点的所有最短路径. 

- [Floyd 算法 O(n3)](https://en.wikipedia.org/wiki/Floyd%E2%80%93Warshall_algorithm)

  > 主要是判断经过该点到达的临时目点的权值和该点目前权值(可以不考虑是否已经经过, 但是循环只会扫一遍所以没有什么影响)的大小来判断是否更新权值.

- 关键路径和拓扑排序 (KeyWord : 松弛)

### 8 动态存储管理


### 查找