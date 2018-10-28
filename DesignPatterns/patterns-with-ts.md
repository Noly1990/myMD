# Head First 设计模式-TypeScript-（一）

> 通过TypeScript学习设计模式，重要的是思考的过程

[TOC]

##  初识设计模式

> 全书第一章，通过一个鸭子游戏的方式，带领读者进入设计模式的世界，通过设计一个情景，模拟现实世界的需求，我们开始设想如何设计并实现模拟各种鸭子的行为和展示。

书中开始设想了最普通的OO模式，通过父类派生继承实现对各种鸭子的模拟，但是，随着鸭子的种类增多，所需要处理的细节越来越多，比如

- 鸭子能不能飞
- 鸭子如何叫
- 鸭子是否是活物
- 鸭子是否有其他特性

如果只通过继承以及实现各种子类，则我们需要对新增加的属性添加非常多的描述和判定，由此每新增一个变量，我们可能需要对所有已经实现的子类进行对应的修改，所以这种单纯的OO模式，弊端非常明显。

### 接口代理

我们根据需求划分出多变的需求，实现了两个代理鸭子飞行行为`FlyBehavior`和鸣叫的行为`QuackBehavior`，如此我们针对鸭子各种飞行的能力以及叫声的变化都可以通过这两个接口去具体化。

```typescript
// 叫声行为 
interface QuackBehavior {
      quack():void;
 }
// 飞行行为
 interface FlyBehavior {
      fly():void;
 }
```

通过 叫声接口 我们实现如下三种叫声行为

```typescript
// 普通鸭子的叫声gua gua gua 
class NormalQuack implements QuackBehavior{
    quack(){
      console.log('Gua Gua Gua')
    }
 }
// 橡皮鸭的叫法 zhi zhi zhi
 class ZhiZhiQuack implements QuackBehavior {
   quack(){
     console.log('Zhi Zhi Zhi')
   }
 }
// 木头鸭不会叫
 class MuteQuack implements QuackBehavior {
   quack(){
     console.log('...........')
   }
 }
```

通过 飞行接口 我们实现两种飞行行为

```typescript
 class CanFly implements FlyBehavior {
    fly(){
      console.log('this duck can fly')
    }
 }

 class CannotFly implements FlyBehavior {
   fly(){
      console.log('this duck can not fly')
   }
 }
```

接下来，假设父类不需要实例化，用抽象类实现```Duck```基类

```typescript
abstract class Duck {
    abstract flyBehavior:FlyBehavior;
    abstract quackBehavior:QuackBehavior;
    public name:string;
    constructor(name:string){
      this.name = name;
    }
    fly():void {
      this.flyBehavior.fly()
    }
    quack():void {
      this.quackBehavior.quack()
    }
    display():void {
      console.log(`------This is a duck called ${this.name}.--------`)
      this.quack()
      this.fly()
      console.log(`-------------------------------------------------`)
    }
}
```

然后我们来实现一个橡皮鸭子，橡皮鸭不会飞，会zhizhi叫

```typescript
class RubberDuck extends Duck {
  public flyBehavior:FlyBehavior;
  public quackBehavior:QuackBehavior;
  constructor(name:string){
    super(name)
    this.flyBehavior = new CannotFly()
    this.quackBehavior = new ZhiZhiQuack()
  }
}
```

再然后我们实现一个普通红头鸭子,会gua gua 叫，会飞

```typescript
class RedheadDuck extends Duck {
  public flyBehavior:FlyBehavior;
  public quackBehavior:QuackBehavior;
  constructor(name:string){
    super(name)
    this.flyBehavior = new CanFly()
    this.quackBehavior = new NormalQuack()
  }
}
```

最后，我们将他们实例化，进行测试

```typescript
let rubberDuck = new RubberDuck('Rubber-Duck')
rubberDuck.display()

let redheadDuck = new RedheadDuck('Red-head-Duck')
redheadDuck.display()

```

我们也可以实现一个木头鸭子，不会叫，不会飞

```typescript
class WoodenDuck extends Duck {
  public flyBehavior:FlyBehavior;
  public quackBehavior:QuackBehavior;
  constructor(name:string){
    super(name)
    this.flyBehavior = new CannotFly()
    this.quackBehavior = new MuteQuack()
  }
}

let woodenDuck = new WoodenDuck('Wooden-Duck')
woodenDuck.display()

```

再来看这种方式的优点，我们假设来实现一个火箭鸭，火箭飞行模式，huhu叫声

```typescript
// 我们只要新增一种飞行模式 和叫声模式就行 
class RocketFly implements FlyBehavior {
   fly(){
      console.log('this duck can fly by rocket')
   }
 }
class HuhuQuack implements QuackBehavior {
   quack(){
     console.log('Hu Hu Hu')
   }
 }
class RocketDuck extends Duck {
  public flyBehavior:FlyBehavior;
  public quackBehavior:QuackBehavior;
  constructor(name:string){
    super(name)
    this.flyBehavior = new RocketFly()
    this.quackBehavior = new HuhuQuack()
  }
}

let rocketDuck = new RocketDuck('Rocket-Duck');
rocketDuck.display()

```





##  观察者模式

> 本章的实际情景是一个气象监测预报显示程序，通过几个感应装置获得气象数据，然后需要在一些布告板将信息统计及显示出来

​	开始我们会有最基本的设想，在气象中心获取数据的时候，分别对各个布告板进行update就行了，但随着布告板的增减，每次必须对相对应的代码进行增删

- 布告板的数量是可变的，可以随心所欲的增删
- 有新的气象信息的时候，每块布告板都要能及时更新信息
- 各功能部分要充分解耦



## 装饰者模式

> 本章的应用场景是一家咖啡店的点单程序，通过组合不同的饮料和配料，最后得出饮料及其售价

​	最初的设计是由一个```Beverage```基类，继承出所有其他的饮料种类，产生子类的时候，根据添加的不同的配料去编写所有其他的饮料子类，这样子，当一种配料发生变化或者价格涨跌，都会导致所有涉及的饮料的内容的修改，显然这种方式是不合适的。

​	对这种模式再进行一定的修改之后，我们在```Beverage```中加上所有配料的实现，并对其```cost()```进行实现，根据不同的配料进行价格的计算。但这种模式，当添加减少的配料时，都需要在基类中修改实现，而且不利于实现多倍配料的饮料，比如双倍奶油，双倍摩卡这类的。

​	大略的阅读完本章以后，发觉，书中以java的方式，实现的是装饰类，而TS中提供的装饰器是函数类型，而书中的实现更像是链式调用，先模拟书中实现装饰类的模式，然后再实现一种以TS装饰器为基数的装饰者模式。

