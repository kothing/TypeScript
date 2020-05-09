
# 类
类是对对象的抽象，描述了对象的特征和行为，而对象就是类的实例。ES6引入了类的概念（相关内容可参考ES类和ES6类的继承两节），TypeScript在此基础上，不仅根据ES7等规范完善了类的语法，还添加了许多其它语法。而在使用TypeScript的类时，不必关心兼容性问题，因为这些工作已由编译器完成。

下面是一个简单的类，包含3个成员：带private修饰符的name属性、构造函数constructor()和getName()方法，最后一句使用new运算符创建了Person类的实例，并调用了一次它的构造函数。
```typescript
class Person {
  private name: string;
  constructor(name: string) {
    this.name = name;
  }
  getName() {
    return this.name;
  }
}
let worker = new Person("strick");
```
编译后的代码如下所示，通过传统的构造函数和基于原型的继承来模拟一个类。

```typescript
var Person = /** @class */ (function() {
  function Person(name) {
    this.name = name;
  }
  Person.prototype.getName = function() {
    return this.name;
  };
  return Person;
})();
var worker = new Person("strick");
```


## 属性

在ES6中，实例属性（即自有属性）得作为this对象的属性存在，并且一般都会在构造函数中执行初始化，而TypeScript允许在类中直接定义实例属性，如下所示。
```typescript
class Person {
  constructor(name: string) {
    this.name = name;
  }
}
//TypeScript中的实例属性
class Person {
  name: string;
}
```
　　不仅如此，TypeScript还提供了存在于类本身上的静态属性，即不需要实例化就能调用的属性。在下面的示例中，为age属性添加了static关键字，使其成为静态属性，通过类的名称就能直接调用它。
```typescript
class Person {
  static age: number;
}
Person.age = 28;
```

## 修饰符

修饰符是用于限定成员或类型的一种符号，TypeScript包含三个访问修饰符：public、private和protected，以及一个成员修饰符：readonly。

**public**

在TypeScript中，成员默认都是public的，即在派生类（也叫子类）或类的外部都能被访问。在下面的示例中，Person类中的name属性是公共的，Programmer类继承了Person类。注意，当派生类包含一个构造函数时，必须调用super()方法，执行基类（即父类）的构造函数，并且该方法得在访问this对象之前调用。
```typescript
class Person {
  public name: string;
  constructor(name: string) {
    this.name = name;
  }
}
class Programmer extends Person {
  constructor(name: string) {
    super(name);
  }
}
```
在初始化Person类或Programmer类之后，就能通过创建的实例来访问name属性，如下所示。
```typescript
let person = new Person("strick");
person.name;            //"strick"
let programmer = new Programmer("freedom");
programmer.name;        //"freedom"
```

**private**

　　当成员被修饰为private时，只能在类的内部访问它，例如在基类Person中声明一个私有的age属性，在类的实例或派生类的实例中访问age属性都会在编译阶段报错，如下所示。
```typescript
class Person {
  private age: number;
}
person.age;            //错误
programmer.age;        //错误
```
当构造函数被修饰为private时（如下所示），包含它的类既不能实例化，也不能被继承。
```typescript
class Person {
  private constructor(name: string) {
    this.name = name;
  }
}
```

**protected**

此修饰符与private的行为类似，只是有一点不同，即在派生类中还是可以访问它的，例如在基类Person中声明一个受保护的school属性，在派生类中就能访问到它，如下所示（省略了基类的构造函数）。
```typescript
class Person {
  protected school: string;
}
class Programmer extends Person {
  constructor(name: string) {
    super(name);
    this.school = "university";
  }
}
```
当构造函数被修饰为protected时（如下所示），包含它的类不能实例化，但可以被继承。
```typescript
class Person {
  protected constructor(name: string) {
    this.name = name;
  }
}
```

**readonly**

当成员被修饰为readonly时，它就变成只读的，只能在声明时或构造函数里初始化，其它地方对它的修改都是禁止的，如下所示。
```typescript
class Person {
  readonly gender: string = "女";        //正确
  constructor() {
    this.gender = "男";            　　　 //正确
  }
}
let person = new Person();
person.gender = "女";            　　　　 //错误
```
当readonly与其它修饰符一起使用时，需跟在它们后面，如下所示。
```typescript
class Person {
  protected readonly gender: string;
}
```


## 参数属性

参数属性可以便捷的在构造函数中声明并初始化一个类的属性，此类参数会与三个访问修饰符或readonly组合使用，如下所示。
```typescript
class Person {
  constructor(public name: string) { }
}
```
构造函数中的name是一个参数属性，相当于在Person类中声明一个name属性，并在构造函数中为其初始化，如下所示。
```typescript
class Person {
  public name: string;
  constructor(name: string) {
    this.name = name;
  }
}
```

## 抽象类

抽象类是供其它派生类继承的基类，它与接口一样，不能被实例化，但可以包含成员的实现细节。在声明一个类时，如果包含abstract关键字，那么这就是一个抽象类，如下所示，当对其进行实例化时，会在编译时报错。
```typescript
abstract class Person { }
let person = new Person();        //错误
```
在抽象类中，会声明一个或多个带abstract类修饰符的抽象方法，它们只有名称，不包含实现细节，可与访问修饰符组合使用，如下所示。
```typescript
abstract class Person {
  protected abstract work(): void
}
```
派生类中必须实现继承的抽象方法（如下所示），否则会在编译阶段报错。
```typescript
class Programmer extends Person {
  public work(): void {
    console.log("code");
  }
}
```
