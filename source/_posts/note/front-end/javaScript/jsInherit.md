---
title: JS继承
subtitle: JS继承的实现方式
link: jslnherit
catalog: true
date: 2023-02-16 21:12:22
tags:
  - 前端
  - JS高级
  - 继承
categories:
  - [笔记, 前端, JavaScript]
---

# JS 继承的实现方式

1. 原型链式继承
   第一种是以原型链的方式来实现继承，**也就是把子类的原型赋值为父类的原型**。但是这种实现方式存在的缺点是，_在包含有引用类型的数据时，会被所有的实例对象所共享，容易造成修改的混乱_。还有就是在创建子类型的时候不能向超类型传递参数。

2. 盗用构造函数式继承
   第二种方式是使用借用构造函数的方式，也**就是在子类中调用父类的构造函数，通过`call`方法将`this`指向子类实列**。这种方式是通过在子类型的函数中调用超类型的构造函数来实现的，这一种方法解决了不能向超类型传递参数的缺点，_但是它存在的一个问题就是无法实现函数方法的复用，并且在类型原型定义的方法子类型也没有办法访问到。_
3. 组合式继承
   第三种方式是组合继承，组合继承是将原型链和借用构造函数组合起来使用的一种方式。**通过借用构造函数的方式来实现类型的属性的继承，通过将子类型的原型设置为超类型的实例来实现方法的继承**。这种方式解决了上面的两种模式单独使用时的问题，但是由于我们是以超类型的实例来作为子类型的原型，所以*调用了两次超类的构造函数，造成了子类型的原型中多了很多不必要的属性。*
4. 原型式继承
   第四种方式是原型式继承，原型式继承的主要思路就是基于已有的对象来创建新的对象，**实现的原理是，向函数中传入一个对象，然后返回一个以这个对象为原型的对象**。这种继承的思路主要不是为了实现创造一种新的类型，只是对某个对象实现一种简单继承，ES5 中定义的 Object.create() 方法就是原型式继承的实现。_缺点与原型链方式相同。_
5. 寄生式继承
   第五种方式是寄生式继承，寄生式继承的思路是创建一个用于封装继承过程的函数，**通过传入一个对象，然后复制一个对象的副本，然后对象进行扩展，最后返回这个对象**。这个扩展的过程就可以理解是一种继承。这种继承的优点就是对一个简单对象实现继承，如果这个对象不是自定义类型时。_缺点是没有办法实现函数的复用。_
6. 寄生组合式继承
   第六种方式是寄生式组合继承，组合继承的缺点就是使用超类型的实例做为子类型的原型，导致添加了不必要的原型属性。**寄生式组合继承的方式是使用超类型的原型的副本来作为子类型的原型，这样就避免了创建不必要的属性**。
   ![寄生组合式继承](https://i.328888.xyz/2023/02/16/pt8g3.png)

```

```