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
    - virtual function机制支持有效率的“执行期绑定”
    - virtual base class（shared）实现“多次出现在继承体系中的base class拥有单一且被共享的实例”
- 对象的虚表指针由构造、析构、赋值函数操作，类的type_info（RTTI）通常放置在虚表的第一个slot
- 派生类存储基类的模型
    - 基类对象直接放在派生类对象中，提供了最紧凑、最高效的存取
    - 派生类对象的1个slot包含指向基类对象的指针(引入间接层导致空间和存取时间上的负担)
    - 派生类对象内含一个基类表指针，指向其继承的基类表，与虚函数机制很像(引入间接层导致空间和存取时间上的负担)
- STL - ADT(abstract data type) - Object-based
- 指向不同类型的指针的差异，不在指针的表示法不同，也不在其内容(地址)不同，而是在其寻址出来的对象类型不同
    - void*只能持有一个地址，不能通过它操作对象
    - cast是一种编译器指令，大部分情况下它不会改变指针的真正地址，而是影响指针指向内存的大小与对象的解释方式