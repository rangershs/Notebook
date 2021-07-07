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

#### Chapter3
- 迭代器模式
    - 提供一种方法，使之能够依序遍历容器的各个元素，而又无需暴露容器的内部实现
    - STL = Container + Algorithm + Iterator(class/function template)
    - 迭代器是一种行为类似指针的对象
    - RTTI typeid()获得的是类型名称，不能用于声明变量
        - function template的参数推导机制可以获取迭代器所指对象的类型
        - ```
            template<typename I, typename T>
            void func_impl(I iter, T t)
            {
                T temp;
                ...;
            }
            template<typename I>
            inline
            void func(I iter)
            {
                func_impl(iter, *iter);
            }
            int main()
            {
                int i = 2021;
                func(&i);
                return 0;
            }
          ```
    - typename Iter::value_type
        - class type内部声明内嵌类型
        - 模板实例化前编译器不知道模板参数的意义，不知道value_type是成员函数、数据成员还是内嵌类型
        - typename显示指定value_type是类型
    - 偏特化(STL接受原生指针作为迭代器)
        - 如果class template拥有一个以上template参数，可以针对其中一个或多个参数进行特化
        - 其实并不一定是对某个或某些模板参数指定参数值，而是提供条件限制的特化版本，如下
        - ```
            template<typename T> class C {};
            template<typename T> class C<T*> {};
            template<typename T> class C<const T*> {};
          ```
    - ```
        template<class T>
        struct iterator_traits
        {
            typedef typename T::value_type          value_type;
            typedef typename T::difference_type     difference_type;
            typedef typename T::pointer             pointer;
            typedef typename T::reference           reference;
            typedef typename T::iterator_category   iterator_category;
        };
        template<class T>
        struct iterator_traits<T*>
        {
            typedef T           value_type;
            typedef ptrdiff_t   difference_type;
            typedef T*          pointer;
            typedef T&          reference;
            typedef random_access_iterator_tag iterator_category;
        };
        template<class T>
        struct iterator_traits<const T*>
        {
            typedef T           value_type;
            typedef ptrdiff_t   difference_type;
            typedef const T*    pointer;
            typedef const T&    reference;
            typedef random_access_iterator_tag iterator_category;
        };
      ```
    - Iterator Category
        - Input/Output/Forward/Bidirectional/Random Access Iterator
        - ```
            struct input_iterator_tag {};
            struct output_iterator_tag {};
            struct forward_iterator_tag : public input_iterator_tag {};
            struct bidirectional_iterator_tag : public forward_iterator_tag {};
            struct random_access_iterator_tag : public bidirectional_iterator_tag {};
          ```
    - C++函数以by reference的方式传回左值
        - mutable iterator，value_type T，传回T&
        - constant iterator，value_type T，传回const T&
    - ```
        if (n >= 0)
            while (n--) ++i;
        else
            while (n++) --i;
      ```
    - 重载机制 + 继承消除传递调用
        - ```
            struct B {};
            struct D1 : public B {};
            struct D2 : public D1 {};

            template<class I>
            inline void func(I& i, B) { cout << "B version" << endl; }
            template<class I>
            inline void func(I& i, D2) { cout << "D2 version" << endl; }

            int main()
            {
                int* p;
                func(p, B());       //  "B version"
                func(p, D1());      //  "B version"
                func(p, D2());      //  "D2 version"
                return 0;
            }
          ```
    - distance()
        - ```
            template<class InputIterator>
            inline typename iterator_traits<InputIterator>::difference_type
            __distance(InputIterator first, InputIterator last, input_iterator_tag)
            {
                typename iterator_traits<InputIterator>::difference_type n = 0;
                while (first != last)
                {
                    ++n;
                    ++first;
                }
                return n;
            }

            template<class RandomAccessIterator>
            inline typename iterator_traits<RandomAccessIterator>::difference_type
            __distance(RandomAccessIterator first, RandomAccessIterator last, random_access_iterator_tag)
            {
                return last - first;
            }

            template<class InputIterator>
            inline typename iterator_traits<InputIterator>::difference_type
            distance(InputIterator first, InputIterator last)
            {
                typedef typename iterator_traits<InputIterator>::iterator_category category;
                return __distance(first, last, category());
            }
          ```
    - iterator
        - ```
            template<class Category,
                     class T,
                     class Difference = ptrdiff_t,
                     class Pointer = T*,
                     class Reference = T&>
            struct iterator
            {
                typedef Category    iterator_category;
                typedef T           value_type;
                typedef Difference  difference_type;
                typedef Pointer     pointer;
                typedef Reference   reference;
            };
          ```
    - traits编程技法利用内嵌类型的编程技巧，与编译器的模板参数推导功能实现

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
