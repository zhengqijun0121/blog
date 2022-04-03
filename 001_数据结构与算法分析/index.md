# 数据结构与算法分析


## 一、栈（Stack）、队列（Queue）和向量（Vector）

### 链表

#### 单链表

#### 双向链表

#### 环形链表

#### 带哨兵节点的链表

### 栈

#### 栈的基本概念

栈(stack)是限制插入和删除只能在一个位置上进行的表，该位置是表的末端，叫作栈的顶(top)．栈也叫做LIFO(后进先出)表．

#### 栈的性质

#### 栈的ADT

```c
int IsEmpty(Stack S);
Stack CreateStack(void);
void DisposeStack(Stack S);
void MakeEmpty(Stack S);
void Push(ElementType X, Stack S);
ElementType Top(Stack S);
void Pop(Stack S);
```

#### 栈的实现

由于栈是一个表，因此任何实现表的方式都能实现栈．一般有两种实现方法，一种使用链式结构，另一种使用数组．

##### 栈的数组实现

使用数组来实现栈，通过在表顶端???来实现Push，通过删除表顶端元素实现Pop．Top操作只是考察表顶端元素并返回它的值．

数组实现的特点
- 需要提前声明数组大小
- 连续的内存空间
- 在表头插入和删除操作效率低
- 支持随机访问元素

SqStack.h

```c
#ifndef SQSTACK_H_
#define SQSTACK_H_

// Type Definition
struct StackRecord;
typedef int ElementType;
typedef struct StackRecord *SqStack;

// Function Lists
int IsEmpty(SqStack S);
int IsFull(SqStack S);
SqStack CreateStack(int MaxElements);
void DisposeStack(SqStack S);
void MakeEmpty(SqStack S);
void Push(ElementType X, SqStack S);
ElementType Top(SqStack S);
void Pop(SqStack S);
ElementType TopAndPop(SqStack S);

#endif  /* SQSTACK_H_ */
```

SqStack.c

```c
#include "SqStack.h"
#include <stdlib.h>  // for malloc
#include "FatalError.h"

/* Stack implementation is a dynamically allocated array */
#define EmptyTOS     (-1)
#define MinStackSize (5)

struct StackRecord {
    int Capacity;
    int TopOfStack;
    ElementType *Array;
};

int IsEmpty(SqStack S) {
    return S->TopOfStack == EmptyTOS;
}

int IsFull(SqStack S) {
    return S->TopOfStack == S->Capacity - 1;
}

SqStack CreateStack(int MaxElements) {
    SqStack S;

    if (MaxElements < MinStackSize) {
        Error("Stack size is too small");
    }

    S = (SqStack)malloc(sizeof(struct StackRecord));
    if (S == NULL) {
        FatalError("Out of space!!!");
    }

    S->Array = (ElementType *)malloc(sizeof(ElementType) * MaxElements);
    if (S->Array == NULL) {
        FatalError("Out of space!!!");
    }

    S->Capacity = MaxElements;
    MakeEmpty(S);

    return S;
}

void DisposeStack(SqStack S) {
    if (S != NULL) {
        free(S->Array);
        free(S);
    }
}

void MakeEmpty(SqStack S) {
    S->TopOfStack = EmptyTOS;
}

void Push(ElementType X, SqStack S) {
    if (IsFull(S)) {
        Error("Full stack");
    } else {
        S->Array[++S->TopOfStack] = X;
    }
}

ElementType Top(SqStack S) {
    if (!IsEmpty(S)) {
        return S->Array[S->TopOfStack];
    } else {
        Error("Empty stack");
        return 0;  /* Return value used to avoid warning */
    }
}

void Pop(SqStack S) {
    if (IsEmpty(S)) {
        Error("Empty stack");
    } else {
        --S->TopOfStack;
    }
}

ElementType TopAndPop(SqStack S) {
    if (!IsEmpty(S)) {
        return S->Array[S->TopOfStack--];
    } else {
        Error("Empty stack");
        return 0;  /* Return value used to avoid warning */
    }
}
```

##### 栈的链表实现

使用单链表来实现栈，通过在表顶端插入来实现Push，通过删除表顶端元素实现Pop．Top操作只是考查表顶端元素并返回它的值．

链表实现的特点
- 不需要提前声明分配空间
- 分散的内存空间
- 在表头插入和删除操作效率高
- 不支持随机访问元素

LinkedStack.h

```c
#ifndef LINKEDSTACK_H_
#define LINKEDSTACK_H_

// Type Definition
struct Node;
typedef int ElementType;
typedef struct Node *PtrToNode;
typedef PtrToNode LinkedStack;

// Function Lists
int IsEmpty(LinkedStack S);
LinkedStack CreateStack(void);
void DisposeStack(LinkedStack S);
void MakeEmpty(LinkedStack S);
void Push(ElementType X, LinkedStack S);
ElementType Top(LinkedStack S);
void Pop(LinkedStack S);
ElementType TopAndPop(LinkedStack S);

#endif  /* LINKEDSTACK_H_ */
```

LinkedStack.c

```c
#include "LinkedStack.h"
#include <stdlib.h>  // for malloc
#include "FatalError.h"

/* Stack implementation is a linked list with a header */
struct Node {
    ElementType Element;
    PtrToNode   Next;
};

int IsEmpty(LinkedStack S) {
    return S->Next == NULL;
}

LinkedStack CreateStack(void) {
    LinkedStack S;

    S = (LinkedStack)malloc(sizeof(struct Node));
    if (S == NULL) {
        FatalError("Out of space!!!");
    }

    S->Next = NULL;
    MakeEmpty(S);

    return S;
}

void DisposeStack(LinkedStack S) {
    MakeEmpty(S);
    free(S);
}

void MakeEmpty(LinkedStack S) {
    if (S == NULL) {
        Error("Must use CreateStack first");
    } else {
        while (!IsEmpty(S)) {
            Pop(S);
        }
    }
}

void Push(ElementType X, LinkedStack S) {
    PtrToNode TmpCell;

    TmpCell = (PtrToNode)malloc(sizeof(struct Node));
    if (TmpCell == NULL) {
        FatalError("Out of space!!!");
    } else {
        TmpCell->Element = X;
        TmpCell->Next = S->Next;
        S->Next = TmpCell;
    }
}

ElementType Top(LinkedStack S) {
    if (!IsEmpty(S)) {
        return S->Next->Element;
    } else {
        Error("Empty stack");
        return 0;  /* Return value used to avoid warning */
    }
}

void Pop(LinkedStack S) {
    PtrToNode FirstCell;

    if (IsEmpty(S)) {
        Error("Empty stack");
    } else {
        FirstCell = S->Next;
        S->Next = S->Next->Next;
        free(FirstCell);
    }
}

ElementType TopAndPop(LinkedStack S) {
    PtrToNode FirstCell;
    ElementType X = 0;

    if (IsEmpty(S)) {
        Error("Empty stack");
        return 0;  /* Return value used to avoid warning */
    } else {
        X = S->Next->Element;  /* Save the value of the first element */

        FirstCell = S->Next;
        S->Next = S->Next->Next;
        free(FirstCell);

        return X;
    }
}
```

#### 栈的应用

平衡符号

后缀表达式

函数调用

#### 栈与递归

### 队列

#### 队列的基本概念

#### 队列的性质

#### 队列的ADT

#### 队列的实现

##### 队列的顺序实现

##### 队列的链表实现

#### 队列的应用

### 向量

#### 向量的基本概念

#### 向量的性质

#### 向量的ADT

#### 向量的实现

##### 向量的数组实现

##### 向量的链表实现

----

## 二、树

### 树的基本概念和术语

### 树的遍历

#### 前序遍历

#### 中序遍历

#### 后序遍历

#### 层序遍历

### 二叉树的定义和性质

### 普通树与二叉树的转换

### 树的存储结构

#### 标准形式

#### 完全树的数组形式

### 树的应用

#### Huffman树的定义与应用

----

## 三、查找(search)

### 查找的基本概念

### 对线性关系结构的查找

#### 顺序查找

#### 二分查找

### Hash查找法

#### 常见的Hash函数

##### 直接定址法

##### 随机数法

#### Hash冲突的概念

#### 解决Hash冲突的办法

##### 开散列方法

##### 拉链法

##### 闭散列方法

##### 开址定址法

#### 二次聚集现象

### BST查找法

#### BST树的定义

#### BST树的性质

#### BST树的ADT

#### BST树的实现

查找算法

插入算法

删除算法

### AVL查找法

#### AVL树的定义

#### AVL树的性质

#### AVL树的ADT

#### AVL树的实现

查找算法

插入算法

#### 平衡因子的概念

### 优先队列查找法

#### 堆的定义

#### 堆的生成

#### 堆的调整算法

#### 范围查找

----

## 四、排序

### 排序基本概念

### 排序算法

#### 插入排序

基本思想

算法代码

基本的时间复杂度分析

#### 希尔排序

基本思想

算法代码

基本的时间复杂度分析

#### 选择排序

基本思想

算法代码

基本的时间复杂度分析

#### 快速排序

基本思想

算法代码

基本的时间复杂度分析

#### 合并排序

基本思想

算法代码

基本的时间复杂度分析

#### 基数排序

基本思想

算法代码

基本的时间复杂度分析

----

## 五、图

### 图的基本概念

一个**图**是由有限非空**顶点集**和**边集**组成．

每一条边都是一副点对．如果边有方向，那么就是有向边，表示为<v, w>．如果边没有方向，那么就是无向边，表示为(v, w)．

如果点对是有序的，那么图就是**有向**的．如果点对是无序的，那么图就是**无向**的．

无向图中，顶点v的**度**是指和顶点v相关联的边的数目．

有向图中，以顶点v为弧头的弧的数目称为顶点v的**入度**，以顶点v为弧尾的弧的数目称为顶点v的**出度**．

**路径**是有顶点序列组成，路径的长为该路径上的边数．

序列中顶点不重复出现的路径称为**简单路径**．

若一条路径中第一个顶点和最后一个顶点相同，则这条路径是一条**回路**．

如果图含有一条从一个顶点到它自身的边(v, w)，那么路径ｖｗ有时也称作为**环**(loop)．我们一般讨论的都是无环图！

如果在一个无向图中从每一个顶点到每个其他顶点都存在一条路径，则称该无向图是**连通**的．具有这样性质的有向图称为是**强连通**的．

无向图中的极大连通子图为其**连通分量**．有向图中的极大强连通子图称为有向图的**强连通分量**．

每条边都可以附带一个数，这种与边相关的数称为**权**，权可以表示从一个顶点到另一个顶点的距离或者花费的代价。边上带权的图称为**带权图**．

**完全图**是其每一对顶点间都存在一条边的图．

### 图的存储结构

#### 邻接矩阵

表示图的一种简单的方法是使用一个二维数组，称为邻接矩阵表示法．

#### 邻接表

表示图的另一种方法是使用数组与链表相结合的存储方式，成为邻接表表示法．

### 图的遍历

从图中某一顶点出发访问图中其余顶点，且每个顶点只访问一次，这一过程成为图的遍历．

图的遍历分为广度优先遍历和深度优先遍历．

#### 广度优先遍历

广度优先搜索算法（BFS）类似于树的层次遍历

基本思想：从图中某顶点v出发，在访问了v之后依次访问v的各个未曾访问过的邻接点，然后分别从这些邻接点出发依次访问它们的邻接点，并使得“先被访问的顶点的邻接点先于后被访问的顶点的邻接点被访问，直至图中所有已被访问的顶点的邻接点都被访问到。如果此时图中尚有顶点未被访问，则需要另选一个未曾被访问过的顶点作为新的起始点，重复上述过程，直至图中所有顶点都被访问到为止。

#### 深度优先遍历

图的深度优先搜索遍历（DFS）类似于二叉树的先序遍历。

基本思路：假设初始状态是图中所有顶点均未被访问，则从某个顶点v出发，首先访问该顶点，然后依次从它的各个未被访问的邻接点出发深度优先搜索遍历图，直至图中所有和v有路径相通的顶点都被访问到。 若此时尚有其他顶点未被访问到，则另选一个未被访问的顶点作起始点，重复上述过程，直至图中所有顶点都被访问到为止。

### 最小生成树基本概念

一个无向图的最小生成树就是由该图的那些连接无向图的所有顶点的边构成的树，且其总价值最低．

#### Prim算法

#### Kruskal算法

### 最短路径问题

#### 广度优先遍历算法

#### Dijkstra算法

#### Floyd算法

```java
public void floyd(int[][] path, int[][] dist) {
    // 初始化
    for (int i = 0; i < mVexs.length; i++) {
        for (int j = 0; j < mVexs.length; j++) {
            dist[i][j] = mMatrix[i][j];    // "顶点i"到"顶点j"的路径长度为"i到j的权值"。
            path[i][j] = j;                // "顶点i"到"顶点j"的最短路径是经过顶点j。
        }
    }

    // 计算最短路径
    for (int k = 0; k < mVexs.length; k++) {
        for (int i = 0; i < mVexs.length; i++) {
            for (int j = 0; j < mVexs.length; j++) {

                // 如果经过下标为k顶点路径比原两点间路径更短，则更新dist[i][j]和path[i][j]
                int tmp = (dist[i][k]==INF || dist[k][j]==INF) ? INF : (dist[i][k] + dist[k][j]);
                if (dist[i][j] > tmp) {
                    // "i到j最短路径"对应的值设，为更小的一个(即经过k)
                    dist[i][j] = tmp;
                    // "i到j最短路径"对应的路径，经过k
                    path[i][j] = path[i][k];
                }
            }
        }
    }
}
```

### 拓扑排序


### 历年真题总结

#### 二叉树叶子节点统计个数

递归解法：
- 如果二叉树为空，返回0
- 如果二叉树是叶子节点，返回1
- 如果二叉树不是叶子节点，二叉树的叶子节点数 = 左子树叶子节点数 + 右子树叶子节点数

```java
public static int getNodeNumLeafRec(TreeNode root) {
    if (root == null) {
        return 0;
    }
    if (root.left == null && root.right == null) {
        return 1;
    }
    return getNodeNumLeafRec(root.left) + getNodeNumLeafRec(root.right);
}
```

非递归解法：基于层次遍历进行求解，利用Queue进行。

```java
public static int getNodeNumLeaf(TreeNode root){
    if (root == null) {
        return 0;
    }

    int leaf = 0; // 叶子节点个数
    Queue<TreeNode> queue = new LinkedList<>();
    queue.add(root);
    while (!queue.isEmpty()) {
        TreeNode temp = queue.poll();
        if (temp.left == null && temp.right == null) { // 叶子节点
            leaf++;
        }

        if (temp.left != null) {
            queue.add(temp.left);
        }

        if (temp.right != null) {
            queue.add(temp.right);
        }
    }

    return leaf;
```

#### TopK问题: 选择最大的K个数
用PriorityQueue默认是自然顺序排序，要选择最大的k个数，构造小顶堆，每次取数组中剩余数与堆顶的元素进行比较，如果新数比堆顶元素大，则删除堆顶元素，并添加这个新数到堆中。

Java中的PriorityQueue来实现堆，用PriorityQueue的实现的代码如下：

```java
public class findTopK {
    // 找出前k个最大数，采用小顶堆实现
    public static int[] findKMax(int[] nums, int k) {
        PriorityQueue<Integer> pq = new PriorityQueue<>(k);  // 队列默认自然顺序排列，小顶堆，不必重写compare

        for (int num : nums) {
            if (pq.size() < k) {
                pq.offer(num);
            } else if (pq.peek() < num) {  // 如果堆顶元素 < 新数，则删除堆顶，加入新数入堆
                pq.poll();
                pq.offer(num);
            }
        }

        int[] result = new int[k];
        for (int i = 0; i < k && !pq.isEmpty(); i++) {
            result[i] = pq.poll();
        }
        return result;
    }

    public static void main(String[] args) {
        int[] arr = new int[]{1, 6, 2, 3, 5, 4, 8, 7, 9};
        System.out.println(Arrays.toString(findKMax(arr,5)));
    }
}
// 输出：[5, 6, 7, 8, 9]
```

要选择最小的K个数使用大顶堆，每次取数组中剩余数与堆顶的元素进行比较，如果新数比堆顶元素小，则删除堆顶元素，并添加这个新数到堆中。

Java中的PriorityQueue来实现堆，用PriorityQueue的实现的代码如下：

```java
public class findTopK {
    // 要找前k个最小数，则构建大顶堆，要重写compare方法
    public static int[] findKMin(int[] nums, int k) {
        PriorityQueue<Integer> pq = new PriorityQueue<>(k, new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                return o2-o1;
            }
        });

        for (int num : nums) {
            if (pq.size() < k) {
                pq.offer(num);
            } else if (pq.peek() > num) {//如果堆顶元素 > 新数，则删除堆顶，加入新数入堆
                pq.poll();
                pq.offer(num);
            }
        }

        int[] result = new int[k];
        for (int i = 0; i < k&&!pq.isEmpty(); i++) {
            result[i] = pq.poll();
        }

        return result;
    }

    public static void main(String[] args) {
        int[] arr = new int[]{1, 6, 2, 3, 5, 4, 8, 7, 9};
        System.out.println(Arrays.toString(findKMin( arr,5)));
    }

}
// 输出：[5, 4, 3, 2, 1]
```

<!-- EOF -->


