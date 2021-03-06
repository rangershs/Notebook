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
    - 高层模块不应该依赖底层模块，二者都应该依赖**抽象接口**
    - 抽象接口不应该依赖具体实现，具体实现则应该依赖抽象接口
- 迪米特法则
    - 即最少知识原则(least knowledge principle)，规定一个对象对其他对象有最少的了解
- Abstraction principle, Minimize coupling, Maximize cohesion, Avoid premature optimization, Code reuse is good.

#### Pattern
- 建造者模式
    - 建造的过程是稳定的
    - 也叫生成器模式，分离复杂对象的*构建*和*表示*，相同的构建过程可以创建不同的表示(属性)
    - 用于创建复杂的对象，对象内部的构造顺序通常是稳定的，而对象的组成属性可能变化
- 工厂模式
    - 简单工厂模式在工厂类中进行*switch-case*
    - 抽象工厂模式在客户端(调用者)进行*switch-case*
    - 根据场景灵活应用，比如在抽象工厂框架中嵌入简单工厂模式
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
    - 从一个对象创建另外一个可定制的对象，而且不需要了解任何的创建细节
    - 只知道抽象类型的场景下完成具体对象的克隆，不需要或没法获取具体对象的类型
    - *构造函数实现了多态的效果，因为构造函数不允许声明为virtual*
    - 一般情况下使用拷贝构造/赋值构造的方式
- 适配器模式
    - 复用现有的类和模块的功能，改动接口使其适配
    - 类适配器模式，多重继承实现
    - 对象适配器模式，聚合或组合实现
- 桥接模式
