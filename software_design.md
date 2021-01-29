## Software Design
---

#### Principle
- 单一职责
    - **接口**只负责一个功能
    - 最好只有一个原因引起**类**发生变化
- 开闭原则
    - 开放扩展，关闭修改
    - 并不是绝对不修改，尽可能通过扩展实现功能
- 里氏替换
    - 所有引用基类的地方都能用派生类替换，并且不会产生任何错误或异常
    - 通过抽象实现功能的对外扩展
    - 一般不应该继承具体类，继承的是抽象类或接口
- 依赖反转原则
    - 高层模块不应该依赖底层模块，二者都应该依赖抽象接口
    - 抽象接口不应该依赖具体实现，具体实现则应该依赖抽象接口
- 迪米特法则
    - 即最少知识原则(least knowledge principle)，规定一个对象对其他对象有最少的了解
- Abstraction principle, Minimize coupling, Maximize cohesion, Avoid premature optimization, Code reuse is good.
