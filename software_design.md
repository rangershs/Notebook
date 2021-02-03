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

#### Pattern
- 建造者模式
    - 建造的过程是稳定的
- 工厂模式
- 装饰模式
    - 组合的过程是灵活的、多变的
    - 动态的给对象添加额外的职责
    - [**Warning**]通过继承扩展类功能，或者继承具体类而非抽象类，为了实现功能可能会违反“is-a”的继承法则
- 访问者模式
    - **双分派技术(double dispatch)** *访问是一种操作*
    - 作用于对象的各种元素的操作(operation)，对象与操作的多种组合M*N(if-else或switch的方式很难看)
    - 适用于对象结构相对稳定的系统，对象和相应的操作解耦，使得操作集合可以自由地演化
- 观察者模式
    - 多个观察者(Observer)订阅主题(Subject)，主题对象状态变化时通知所有观察者对象，也叫发布/订阅模式(Publish/Subscribe)
    - [**Warning**]主题对象依赖于观察者，并且观察者对象间的耦合比较强，他们要有相同的方法才能被主题对象调用
    - 采用委托的方式可以解耦观察者，比如声明相同签名的EventHandler或callable object(std::function)
- 单例模式
- 原型模式
- 适配器模式
- 桥接模式
