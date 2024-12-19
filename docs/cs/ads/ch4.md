# Chapter 4 | Leftist Heap & Skew Heap

## Leftist Heap

!!! quote "link"
    OI Wiki: https://oi-wiki.org/ds/leftist-tree/

    Wikipedia: https://en.wikipedia.org/wiki/Leftist_tree

---

### 概述
!!! note "Preview of Binary Heap"
    简单过一下二叉堆比较重要的几个操作（以最小堆为例）

    - **Insert**:插入到二叉堆的末尾，再Percolate Up:$O(log(n))$
    - **FindMin**:直接弹出最顶上的元素:$O(1)$
    - **DeleteMin**:删除最顶上的元素，以末尾元素替代之，再Percolate Down:$O(log(n))$
    - **BuildHeap**:可以执行连续插入:$nO(log(n))$;更好的方式是全部排列好，从倒数第二层开始Percolate Up:$O(n)$ -- from course DS
    - ~~**Merge(Meld)**~~:类似BuildHeap，可以将一个二叉堆中的元素依次插入另一个二叉堆；但更好的方式是全部打乱再BuildHeap:$O(n)$

我们不太满意Binary Heap的Merge，因为全部打乱再BuildHeap的操作消解了两个Heap的特征，虽然说$O(n)$已经不错了，但是我们还想找到更好的结构来进行快速的堆合并操作，于是左偏堆(Leftist Heap)应运而生,它的堆合并操作的复杂度只要$O(log(n))$

!!! note "@cy's ppt"
    **Leftist Heap:**
    Order Property – the same
    Structure Property – binary tree, but unbalanced

左偏堆(Leftlist Heap)也是一颗二叉树，不仅具有堆的性质，并且是 **「左偏」** 的，但是由于它并不再是一颗完全二叉树，所以不能像维护大小堆一样用数组来维护它

!!! question "Why arrays?"
    这里提一嘴为什么堆会用数组存储，使用数组来存储堆的节点信息，有一种天然的优势那就是节省内存空间。因为数组占用的是连续的内存空间，相对于散列存储的结构来说，数组可以节省连续的内存空间，不会将内存打乱。

---

### 定义
一个左偏堆的结点维护了左右子堆的地址、自身的键值、和一个**距离(dist)**/**Npl(null path length)**

```cpp
struct LeftistHeapNode {
    ElementType val;
    int dist;
    LeftistHeapNode * ls, * rs;
};
```

!!! definition "@cy's ppt"
    [Definition] **The null path length**,**Npl**(x),of any node X is the length of the shortest path from X to a node without two children. Define Npl(NULL) = -1.

    [Definition] **The leftist heap property** is that for every node X in the heap, the null path length of the left child is at least as large as that of the right child.

    简单来说，每个节点都额外存储了它到小于2个孩子的节点的最短距离 dist/Npl，并且对任意节点而言，它的左孩子的 dist >= 右孩子的dist
    需要注意的是，dist 不是深度，左偏树的深度没有保证，一条向左的链也符合左偏树的定义。

---

### 性质
!!! property "properties"
    1. 结点的 $dist$ 等于 $dist_\text{right child} + 1$（假设 $dist_\text{NULL} = -1$）；或者可以说结点的 $dist$ 就是最右路径的节点数
    2. 如果 $dist_i = N$，则以 $i$ 为根的子树**至少**是一个 $N+1$ 层的完美二叉树，至少有 $2^{N+1}-1$ 个结点
    3. 最右路径有 r 个节点的左斜堆至少有$2^r - 1$个节点 / 有 N 个节点的左斜堆的最右路径最多有 $\log_2(N+1)$ 个节点 
    
    其中命题 2 和命题 3 事实上是等价的

    ??? proof "证明"
        首先我们明确，如果一棵树的最右路径有 r 个节点，那么这棵树的 dist 就等于 r（根据性质1）/ 甚至其实我们就可以将 dist 定义为最右路径的节点个数。
        我们用数学归纳法加以证明：

        1. 若 r = 1，最右路径有1个节点，命题成立；
   
        2. 假设命题对于2,3,4,5,....r均成立，考虑左斜堆具有 r+1 个节点的情况：根节点的右孩子的最右路径有 r 个节点，右孩子树至少有$2^r - 1$个节点；左孩子的最右路径也至少要有 r 个节点才能满足根节点的左倾性质，因此左孩子树也至少要有$2^r - 1$个节点；因此根节点的这棵树至少有$2*(2^r - 1) + 1 = 2^{r+1} - 1$个节点
   
        所以这个定理成立，其实也就可以证明合并的操作是 $log(n)$ 级别的
    ![](https://note.isshikih.top/cour_note/D2CX_AdvancedDataStructure/img/22.svg){width=49%}
    ![](https://note.isshikih.top/cour_note/D2CX_AdvancedDataStructure/img/23.svg){width=43%}
    > 注意，在示意图中我们省略了结点自身键值的标记，但既然作为一个堆，它就需要满足堆的性质，即结点的键值不大于（不小于）其孩子结点的键值。在实际使用过程中，键值很可能不再是单纯的数，大小关系可能转化为偏序关系。

---

### 操作
左偏堆的核心操作就是合并。而其它操作都可以看作是合并的特殊情况。因此我们首先讨论任意两个左偏堆的合并。

---

#### 合并
作为左偏堆的核心操作，合并操作自然就是要在满足性质的条件下，合并两个左偏堆。大致思路就是先维护堆的性质，在回溯时维护左偏性质，所以实际上它是一个先自上而下再自下而上的过程。

按照实现方法，左偏堆的合并可以分为递归式和迭代式两种。其中前者可能更为直觉，而后者可视化后则更为直观。

---

##### 递归式
递归式先比较当前两个待合并子树的根结点的键值，选择较小（较大）的那个作为根结点，其左子树依然为左子树，右子树更新为「右子树和另一个待合并子树的合并结果」。

当然，在递归地更新完后，我们需要检查左子树和右子树是否满足 $dist_\text{left child} \geq dist_\text{right child}$的性质，如果不满足，我们则需要交换左右子树来维持性质。

```cpp
LeftistHeapNode * merge(LeftistHeapNode * x, LeftistHeapNode * y) {
    // Recursive exit. If any is NULL, return the other as the new root of subtree.
    if (x == NULL) return y;
    if (y == NULL) return x;
    
    // If `x`'s val is smaller than `y`'s, swap them, which means we always operates on `x`.
    if (x->val > y->val) {
        swap(x, y);
    }
    
    // Merge `x`'s right subtree and `y`, and set `x`'s right subtree to the result.
    x->rs = merge(x->rs, y);
    
    // If `x`'s left subtree's dist is smaller than `x`'s right subtree's dist, swap them.
    if (x->ls->dist == NULL || x->ls->dist < x->rs->dist) {
        swap(x->ls, x->rs);
    }

    // Update x's dist.
    x->dist = x->rs->dist + 1;

    // Return x as the new root of subtree.
    return x;
}
```
!!! eg "🌰"
    现在我们模拟一下这个过程，现在我们有下面两个左偏堆，尝试合并它们。

    === "Frame 0"
        ![](https://note.isshikih.top/cour_note/D2CX_AdvancedDataStructure/img/24.svg)
    === "Frame 1"
        ![](https://note.isshikih.top/cour_note/D2CX_AdvancedDataStructure/img/25.svg)

        我们发现，经过比较，<font color=#2ECC71>**❶**</font> 更小，所以我们将 <font color=#2ECC71>**❶**</font> 作为合并后的根结点，左子树不变，右子树更新为「绿树右子树和蓝树的合并结果」。
    === "Frame 2"
        ![](https://note.isshikih.top/cour_note/D2CX_AdvancedDataStructure/img/26.svg)

        经过比较，<font color=#2E86C1>**❷**</font> 更小，所以我们将 <font color=#2E86C1>**❷**</font> 作为合并后的根结点，左子树不变，右子树更新为「蓝树右子树和绿树的合并结果」。
    === "Frame 3"
        ![](https://note.isshikih.top/cour_note/D2CX_AdvancedDataStructure/img/25.svg)

        最后还剩下两个结点啦！实际上这里直接模拟了两个步骤，首先是比较 <font color=#2ECC71>**❺**</font> 和 <font color=#2E86C1>**❻**</font>，并选择了 <font color=#2ECC71>**❺**</font> 作为新根；接下来在递归的过程中发现需要合并 `NULL` 和 <font color=#2E86C1>**❻**</font>，所以直接返回了 <font color=#2E86C1>**❻**</font>。

        然而还没有结束，我们还需要处理左右子树 dist 大小关系问题。
    === "Frame 4"
        ![](https://note.isshikih.top/cour_note/D2CX_AdvancedDataStructure/img/28.svg)

        我们发现 <font color=#2ECC71>**❺**</font> 的左孩子为 `NULL`，我们记 $dist_\text{NULL} = -1$，右孩子 <font color=#2E86C1>**❻**</font> 有 $dist_\text{right child}=0$，所以需要交换两个孩子。
    === "Frame 5"
        ![](https://note.isshikih.top/cour_note/D2CX_AdvancedDataStructure/img/29.svg)

        这里也跳过了两个步骤：

        往回走，发现 <font color=#2ECC71>**❺**</font> 的 dist 小于 <font color=#2E86C1>**❹**</font> 的 dist，满足性质，不需要改变。
        
        继续往回走，发现 <font color=#2ECC71>**❷**</font> 和 <font color=#2E86C1>**❸**</font> 的 dist 相同，满足性质，也不需要改变。
        
        从这里也可以看出来，并不是看上去更大的子树一定在左侧。
---

##### 迭代式
迭代式是根据它的实现方法来命名的，但是我认为从可视化的角度来理解迭代式可能更有意思。事实上在很多题目中我觉得这个方法做题更加方便。

迭代式维护两个额外的指针，分别指向两棵树还没被合并的子树的根，并不断选择较小的那个合并进去，直到两个指针都为空。可以发现，这个过程和归并排序的后半部分非常相似，实际上当我们从可视化的角度去看这件事以后，会发现这里做的**就是一个归并**。

```cpp
LeftistHeapNode * merge(LeftistHeapNode * x, LeftistHeapNode * y) {
    // `tx` & `ty` are the pointers to the roots of the subtrees that haven't been merged.
    LeftistHeapNode * tx = x, * ty = y;
    // `res` is the root of the merged final tree, while `cur` is the latest node that has been merged.
    LeftistHeapNode * res = NULL, * cur = NULL;

    // Begin merging.
    whie (tx != NULL && ty != NULL) {
        // If `tx`'s val is smaller than `ty`'s, swap them, which means we always operates on `tx`.
        if (tx->val > ty->val) {
            swap(tx, ty);
        }
        
        // Specially mark the root on the first merge.
        if (res == NULL) {
            res = tx;
            cur = tx;
        } else {
            cur->rs = tx;
            cur = cur->rs;
        }

        // Go on.
        tx = tx->rs;
    }

    // Merge the rest of the tree.
    while (ty != NULL) {
        // Specially mark the root on the first merge. (rarely happens but not impossible)
        if (res == NULL) {
            res = ty;
            cur = ty;
        } else {
            cur->rs = ty;
            cur = cur->rs;
        }

        // Go on.
        ty = ty->rs;
    }

    // Adjust the left and right subtrees of all the nodes according to the properties of `dist`. 
    // It does the same work as the adjust part in the recursive version. I ignore it here.
    res = adjust(res);

    return res;
}
```

依旧是以上面进行模拟的那个合并为 🌰 进行模拟。

!!! eg "🌰"
    首先，我们对图片的排版稍微做一些改变，我们不再按照之前画堆的方式去画，而是“左偏”地去画：

    ![](https://note.isshikih.top/cour_note/D2CX_AdvancedDataStructure/img/30.svg)

    === "Frame 0"
        ![](https://note.isshikih.top/cour_note/D2CX_AdvancedDataStructure/img/31.svg)

        可以发现，在调整之前,绿色和蓝色的箭头分别表示两个待合并子树尚未合并的子树的根，紫色箭头表示最近的合并发生的位置。
    === "Frame 1"
        ![](https://note.isshikih.top/cour_note/D2CX_AdvancedDataStructure/img/32.svg)

        比较 <font color=#2ECC71>**❶**</font> 和 <font color=#2E86C1>**❷**</font>，发现 <font color=#2ECC71>**❶**</font> 更小，所以我们将 <font color=#2ECC71>**❶**</font> 作为合并后的根结点，左子树随之合并进去，同时更新绿色箭头指向尚未合并进去的 <font color=#2ECC71>**❺**</font>。
    === "Frame 2"
        ![](https://note.isshikih.top/cour_note/D2CX_AdvancedDataStructure/img/33.svg)

        和上一步类似的，比较 <font color=#2ECC71>**❺**</font> 和 <font color=#2E86C1>**❷**</font>，发现 <font color=#2E86C1>**❷**</font> 更小，所以我们将 <font color=#2E86C1>**❷**</font> 作为合并后的根结点，左子树随之合并进去，同时更新蓝色箭头指向尚未合并进去的 <font color=#2E86C1>**❻**</font>。
    === "Frame 3"
        ![](https://note.isshikih.top/cour_note/D2CX_AdvancedDataStructure/img/34.svg)

        依然类似地，比较 <font color=#2ECC71>**❺**</font> 和 <font color=#2E86C1>**❻**</font>，发现 <font color=#2ECC71>**❺**</font> 更小，所以我们将 <font color=#2ECC71>**❺**</font> 作为合并后的根结点，左子树随之合并进去，同时更新绿色箭头指向 `NULL`。
    === "Frame 4"
        ![](https://note.isshikih.top/cour_note/D2CX_AdvancedDataStructure/img/35.svg)

        这时候我们发现已经有一个指针空了，也就是绿色指针已经指向了 `NULL`，这个时候我们只需要按顺序把蓝色指针指向的内容都推进去即可。
    === "Frame 5"
        ![](https://note.isshikih.top/cour_note/D2CX_AdvancedDataStructure/img/36.svg)

        接下来我们还需要维护 dist 信息并根据性质交换左右子树。这一部分和之前一样，就不再赘述了。

!!! eg "🌰/🔥"

    当然，这么来看可能还是很乱，联想我们之前发现它和归并排序很像，我们还可以用一个更加直观的方式来看这个过程：

    === "Frame 0"
        ![](https://note.isshikih.top/cour_note/D2CX_AdvancedDataStructure/img/30.svg)

        同样从这张图开始，由于我们之前提到的，它类似于一个递归排序的后半部分，具体来说是指合并两个有序数列的过程。

        也就是说，我们可以将这两个左偏堆改写成两个有序数列！
    === "Frame 1"
        ![](https://note.isshikih.top/cour_note/D2CX_AdvancedDataStructure/img/37.svg)

        由于我们在合并过程中总是找右孩子，所以我们就沿着最右沿路径把没个左偏堆拆成这种“悬吊”的带状形式，每一“条”的值取决于根的键值，也就是这一“条”的最顶部。

        在这张图中，我们得到的两个**有序**数组分别是 <font color=#2ECC71>[1, 5]</font> 和 <font color=#2E86C1>[2, 6]</font>，接下来我们将它们进行排序。
    === "Frame 2"
        ![](https://note.isshikih.top/cour_note/D2CX_AdvancedDataStructure/img/38.svg) 

        经过排序，就会发现它们刚好符合我们在上面步骤得到的结果（可以对比着上面的 Frame 4 看）。实际上，只要你回顾一下归并排序的过程，再对比着看上面的过程，就会发现一模一样。

在做左斜堆合并的题目时，个人比较喜欢迭代式的做法，只需要沿着最右路径把每个左子树写出来再排序，最后沿着最右路径连回来即可（当然最后还要沿着最右路径对 dist 进行维护）

再次提醒，这一小节讲的部分都忽略了之后调整子树左偏性质的过程，实际上这也就单纯是一个维护堆性质的过程--只需要对**最右路径 Top-Down 或者 Bottom-Up 进行维护即可**

---

#### 插入
插入可以直接看作是和只有一个根节点的树的合并

---

#### 删除
删除操作也比较简单，只需要对被删除节点的两个孩子做合并操作即可

这里贴一段代码方便理解
```C
LeftistHeapNode * del(LeftistHeapNode * cur, ElementType x) {
    if (cur->val == x) {
        // Just return the merge of the children.
        return merge(cur->l, cur->r);
    } else {
        // Not this subtree.
        if (cur->val > x) return cur;

        // Otherwise, search the `x`.
        if (cur->l != NULL) del(cur->l, x);
        if (cur->r != NULL) del(cur->r, x);

        // Adjust the dist bottom-up.
        adjust(cur);
    }
}
```

---

## Skew Heap
!!! quote "link" 
    Wikipedia: https://en.wikipedia.org/wiki/Skew_heap

---

### 概述
Skew Heap 其实是 Leftist Heap 的一种自适应版本，它们二者之间的关系就类似于 AVL Tree 和 Splay Tree 的关系（事实上，发明 Skew Heap 的人和发明 Splay Tree 的人是同一个人）

Skew Heap 是一种具有堆性质的二叉树，但是并没有像 Leftist Heap 一样的对结构上的限制，并没有诸如 Npl 这样的信息存储在各个节点上，这也就意味着一个 Skew Heap 的 Right Path 可能会非常长，这导致了所有操作的最坏时间复杂度是$O(n)$级别的，但是就像 Spaly Tree 一样，所有操作在摊还意义下的时间复杂度是$O(logn)$级别的

!!! tip "另一种视角"
    让我们回顾一下左偏堆，由于需要自下而上地维护 dist，所以我们无法进行并发操作。回顾 AVL 树，同样为了维护它比较严格的平衡性质，我们也无法进行并发操作，而红黑树则通过一个能够仅仅通过变色就能调整的黑高来规避了必须自下而上维护的问题，实现了并发。

    换句话来说，要想将左偏堆改变地能够进行自上而下维护，就需要改变甚至放弃它的左偏性质的严格性——而这就是斜堆的由来。

---

### 合并
斜堆也需要满足大根堆（小根堆）的性质，而它的合并和左偏堆的合并也十分类似，只不过我们这次**无条件的交换左右子树**，换句话来说，不管左偏性质如何变化，我们都会选择交换参与合并的左右子树，这样我们就不需要在回溯的时候才进行左右子树的交换，于是就实现了完全的自上而下。

让我们来看看 wiki 里给出的 🌰：

!!! eg "🌰 from wikipedia"
    === "Frame 0"
        ![](https://upload.wikimedia.org/wikipedia/commons/thumb/1/19/SkewHeapMerge1.svg/540px-SkewHeapMerge1.svg.png)

        这是我们需要合并的两个堆。
    === "Frame 1"
        ![](https://upload.wikimedia.org/wikipedia/commons/thumb/4/4d/SkewHeapMerge7.svg/1280px-SkewHeapMerge7.svg.png){width=40%}

        省略了中间的步骤，可以尝试模拟一下，每一次合并操作结束之后都交换左右子树。

大概的操作类似是每次先比较两个子树的根节点大小，把大的子树往小的子树合并，具体合并过程是，将小的子树的左右子树交换，把新的左子树砍掉，让被砍掉的左子树和待合并的树合并后作为新的左子树(貌似讲的有点糊)

当然，它也是支持迭代的写法的，和是和之前的做法类似，我们可以排序每一“条”，然后再合并。Wikipedia 上提供了一系列的过程图，但是那个过程图有点自下而上的意思，但是实际上自上而下做也是一样的。如果有兴趣可以去看看 Wiki 上的过程：[🔗](https://en.wikipedia.org/wiki/Skew_heap#Non-recursive_merging)。

---

### 摊还分析

!!! quote "link"
    Halifuda:https://www.cnblogs.com/halifuda/p/14364632.html

这里我们采用势能法分析摊还复杂度

分析 skew heap 的均摊复杂度，主要就是分析**合并**操作的复杂度，因为其他操作都可以转化为合并操作。

接下来我们需要定义势能函数：

!!! definition "势能函数"
    我们定义 $\Phi(Heap) = \text{number of heavy node in } Heap$。

其中，额外需要定义 heavy node 和 light node：

!!! definition "heavy node & light node"
    对于一个子堆 $H$，如果 $size(H.\text{right.descendant}) \geq \frac{1}{2}size(H)$，则 $H$ 是 heavy node，否则是 light node。 

    ??? extra "\@ cy'ppt"
        A node p is heavy if the number of descendants of p’s right subtree is at least half of the number of descendants of p, and light otherwise.  Note that the number of descendants of a node includes the node itself.

显然，对于 heavy node 和 light node，以及合并操作，有这么一些性质：

!!! property "properties"
    1. 如果一个节点是 heavy node，并且在其右子树发生了合并（包括翻转），那么它**一定**变为一个 light node；
    2. 如果一个节点是 light node，并且在其右子树发生了合并（包括翻转），那么它**可能**变为一个 heavy node；
    3. 合并过程中，如果一个节点的 heavy/light 发生变化，那么它**原先**一定在堆的最右侧路径上；

列出公式：

$$
\hat{c} = c + \Phi(H_{merged}) - \Phi(H_x) - \Phi(H_y)
$$

其中，$c$ 为合并操作的（最坏）复杂度，$H_{merged}$ 为合并后的堆的势能，$H_x$ 和 $H_y$ 分别为合并前的两个堆的势能。

根据 property 3，在合并过程中并非所有节点都收到影响。我们可以单独记录 $l_{x}$ 为 $H_x$ 最右侧路径上的 light node 数量，$h_{x}$ 为 $H_x$ 最右侧路径上的 heavy node 数量，$h^0_{x}$ 为 $H_x$ 所有不在最右侧路径上的 heavy node 数量（即 $\text{count of heavy nodes of } H_x = H_x + H^0_x$）。

于是，我们可以将上式写开：

$$
\left\{
    \begin{aligned}
        c &= l_x + h_x + l_y + h_y &(1)\\
        \Phi(H_{merged}) &\leq l_x + h^0_x + l_y + h^0_y &(2)\\
        \Phi(H_x) &= h_x + h^0_{x} &(3)\\
        \Phi(H_y) &= h_y + h^0_{y} &(4)
    \end{aligned}
\right.
$$

其中稍微做一些解释：

1. $(1)$：$c$ 为合并操作的（最坏）复杂度，即我们的枚举涉及了两个堆所有的右侧路径；
2. $(2)$：在合并操作以后，根据 property 1 和 property 2，可以得到这个不等式；
3. $(3)$ 和 $(4)$：根据势能函数的定义得到；

于是，将它们代入得到结果：

$$
\begin{aligned}
\hat{c} 
    &= c + \Phi(H_{merged}) - \Phi(H_x) - \Phi(H_y) \\
    &\leq (l_x + h_x + l_y + h_y)
    + (l_x + h^0_x + l_y + h^0_y)
    - (h_x + h^0_{x})
    - (h_y + h^0_{y}) \\
    &\leq 2(l_x + l_y) \\
\hat{c}
    &= O(\log{N})
\end{aligned}
$$

??? note "右路径 light node 的数量"
    这里需要补充的是，右路径 Light node 的数量是 $O(logn)$ 量级的

    一种思路是这样的：
    
    轻结点有如下特征：若一棵N个结点的树R是轻的，则它的右子树的大小小于N/2。一个斜堆的右路径指的是：从根一直通向右子树。因此在一个斜堆的右路径上：当一个结点是轻结点，那么接下来的右子树的大小至少会减半。
    
    如果右路径上每个结点都是轻结点，那么数量也不会超过logN。就算其中有一些重结点，最坏的情况莫过于在遇到下一个轻结点之前，所有点都在右子树上，右路径上右子树的大小不会变大，从而轻结点的个数不会因为这些出现在中间的重结点而变得更多。

