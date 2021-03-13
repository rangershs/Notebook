## The Annotated STL Sources
---

### STL Sources
> "源码之前 了无秘密"
> "天下大事 必作于细"
> "思维理论基础 实务运用能力"

#### Chapter1
- STL为了提升复用性，建立在某种标准之上，无论是语言层次的标准、数据交换的标准或通讯协议的标准
- 组件
    - 容器
    - 算法
    - 迭代器
    - 仿函数
    - 适配器
    - 配置器
- 源码开放
    - GNU GPL
    - BSD License
    - Apache License

#### Hashtable
- 可视为一种字典结构
- **hashfunction**，将某一元素映射为可接受的值，一般是将大数映射为小数
- **loading factor**，元素个数与表大小的比值
- 线性探测
    - C+1, C+2, ... , C+i
    - lazy deletion，标记删除记号，在哈希表重整(**rehash**)时删除元素
- 二次探测(平方探测)
    - C+1\*1, C+2\*2, ..., C+i*i
- 开链法
    - 每个表格元素维护一个list，即vector+list，涉及链表指针操作
    - SGI STL采用
    - 表格大小hashtable_size仍然使用质数
    - 哈希函数计算=std::hash(key) % hashtable_size

#### Associative Container
- Containment
    - heap contains a vector
    - priority-queue contains a heap
    - stack and queue contain dequeue
    - set/map contains a RB-tree
    - hashset/hashmap contains a hashtable
- 二叉树
    - 树中任何节点最多只有2个子节点
    - 树由根节点和左右子树构成
    - 编译器的表达式树(expression tree)和哈夫曼编码树(Huffman coding tree)都是二叉树
- AVL
    - 任何节点的左右子树高度相差不超过1，确保整棵树的深度不超过O(logN)
    - 调整“插入点至根节点”路径上，平衡状态被破坏最深的那一个节点，可使整棵树重新恢复平衡
    - 左左，插入点位于最深左子节点的左子树
    - 左右，插入点位于最深左子节点的右子树
    - 右左，插入点位于最深右子节点的左子树
    - 右右，插入点位于最深右子节点的右子树
    - 外侧插入(左左和右右)，采用单旋转操作可恢复平衡
    - 内侧插入(左右和右左)，采用双旋转操作可恢复平衡
- RB-tree
    - 平衡状态定义
        - 节点不是红色就是黑色，根节点是黑色
        - 父子节点不得同时为红
        - 任意节点到叶子节点NULL的任一路径，包含相同数量的黑色节点
    - 若插入的新节点(红色)导致红黑树不满足平衡条件，经过单旋转/双旋转/变色操作可使红黑树恢复平衡
    - 插入或删除元素后，红黑树会**自动排序**(即恢复平衡)

#### Algorithm
- 算法，即解决问题的方法，以有限的步骤解决逻辑或数学上的问题
- Big-O标记法的数据量要足够大，它不适用于小数据量下的情况
    - 比如时间复杂度O(log2N)，表示花费固定时间可以将问题规模降低一半(通常是1/2)
    - 多项式最高项的系数、较低次方项、对数的底数对于Big-O标记法是没有影响的
- STL算法作用在由迭代器[first, last)所表示出来的区间上
    - 质变算法(mutating)，运算过程中会更改迭代器所指元素的内容
    - 非质变算法(non-mutating)，运算过程中不会更改迭代器所指元素的内容
- 算法的泛化
    - 迭代器，行为类似指针的对象
    - 泛型迭代器具备4种操作方式，inequility, dereferencelm, prefix increment, copy
