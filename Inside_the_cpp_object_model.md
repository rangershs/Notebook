## Inside the Cplusplus object model
---

#### Chapter-1
- 类型和个数均参数化的模板类，内部使用数组实现
```
private:
    type[number];
```
- class中的non-inline member function只会产生一个函数实例，而inline function则会在每一个使用模块上产生一个实例
- virtual
    - virtual function机制支持有效率的**执行期绑定**
    - virtual base class（shared）实现“多次出现在继承体系中的base class拥有单一且被共享的实例”
- 对象的虚表指针由构造、析构、赋值函数操作，类的type_info（RTTI）通常放置在虚表的第一个slot
- 派生类存储基类的模型
    - 基类对象直接放在派生类对象中，提供了最紧凑、最高效的存取
    - 派生类对象的1个slot包含指向基类对象的指针(引入间接层导致空间和存取时间上的负担)
    - 派生类对象内含一个基类表指针，指向其继承的基类表，与虚函数机制很像(引入间接层导致空间和存取时间上的负担)
- STL - ADT(abstract data type) - Object-based
- 指向不同类型的指针的差异，不在指针的表示法不同，也不在其内容(地址)不同，而是在其寻址出来的对象类型不同
    - 指针本身所需的内存大小是固定的
    - void*只能持有一个地址，不能通过它操作对象
    - cast是一种编译器指令，大部分情况下它不会改变指针的真正地址，而是影响指针指向内存的大小与内容的解释方式
- 面向对象
    - C++通过clas的指针或引用支持多态(机制)，public继承抽象的接口之后，封装相关的类型
    - 代价是额外的间接性，在“内存的存取”或“类型的决议”上
- 基于对象
    - 具体的ADT程序风格
    - 如STL，包含数据和算法，不支持类型的扩展
- OB设计可能比对等的OO设计速度快而且空间更紧凑，但OB设计没有弹性
    - 速度快，函数调用操作都在编译时期解析完成，不需要设置virtual机制
    - 空间紧凑，每个对象不需要负担为了支持virtual机制的额外负荷


#### Chapter-3
- Empty virtual base class的大小视编译器的处理而不同，可能为1byte，可能为0byte
- C++对象模型尽量以空间优化和存取速度优化的考虑来表现data members，并且保持和C-struct的兼容性
- 由于C++的语言规则出现的一些*防御性程序设计风格*
    - 总是把"nested type声明"放在class的起始处
    - *所有的inlines定义都放在class外(C++2.0已修订)*
    - *data members在class的起始处声明(C++2.0已修订)*
- C++Std要求同一个access section中的data members符合”较晚出现的members有较高的地址“，因此members的排列不一定是连续的
    - members之间可能会填充字节满足内存对齐的要求
    - members之间可能插入vptr支持多态特性
- 通常，选择某些函数作为inline，是设计class的一个重要课题
- C++语言保证“出现在derived class中的base class subobject有其完整性”
    - 把一个class分解为两层或多层，可能会导致其空间的膨胀
```
class Point2d
{
public:
    Point2d(float x = 0.0, float y = 0.0): x_(x), y_(y) {}
    ~Point2d() {}

    float x() { return x_; }
    float y() { return y_; }
    virtual float z() { return 0.0; }

    virtual void operator+=(const Point2d& rhs)
    {
        x_ += rhs.x();
        y_ += rhs.y();
    }

protected:
    float x_;
    float y_;
};

class Point3d
{
public:
    Point3d(float x = 0.0, float y = 0.0, float z = 0.0)
        : Point2d(x, y), z_(z) {}
    ~Point3d() {}

    float z() { return z_; }

    virtual void operator+=(const Point2d& rhs)
    {
        Point2d::operator+=(rhs);
        z_ += rhs.z();
    }

protected:
    float x_;
    float y_;
    float z_;
};
```
- 多重继承下，data members的位置在编译时就确定了，存取members就像单一继承一样是一个简单的offset运算
    - derived class object类型指针(引用)转换为第一个base class类型的指针，不需要编译器修改地址
    - derived class object类型指针(引用)转换为第二或后继base class类型的指针，需要编译器介入修改地址，才能指向subobject的地址
- 虚拟继承
    - virtual base class在多数时候最有效的运用形式：一个抽象的virtual base class，没有任何data members
    - 经一个非多态的class object存取一个继承而来的virtual base class member，可以被优化为一个直接存取操作，进行简单的offset运算即可
    - derived class内部布局分为一个不变区域和一个共享区域
    - derived class共享区域的实现多数情况下在class object中插入指针，指向virtual base class members，或指向virtual base class table(table中保存virtual base class指针)