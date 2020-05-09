# 装饰器（Decorator）

装饰器（Decorator）可声明在类及其成员（例如属性、方法等）之上，为它们提供一种标注，用于分离复杂逻辑或附加额外逻辑，其语法形式为@expression。expression是一个会在运行时被调用的函数，它的参数是被装饰的声明信息。假设有一个@sealed装饰器，那么可以像下面这样定义sealed()函数。
```typescript
function sealed(target) {
  //...
}
```
有两种方式可以开启装饰器，第一种是在输入命令时添加--experimentalDecorators参数，如下所示，其中--target参数不能省略，它的值为“ES5”。
```typescript
tsc default.ts --target ES5 --experimentalDecorators
```
第二种是在tsconfig.json配置文件中添加experimentalDecorators属性，如下所示，对应的target属性也不能省略。
```typescript
{
  compilerOptions: {
    target: "ES5",
    experimentalDecorators: true
  }
}
```

## 类装饰器

类装饰器用于监听、修改或替换类的构造函数，并将其作为类装饰器唯一可接收的参数。当装饰器返回undefined时，延用原来的构造函数；而当装饰器有返回值时，会用它来覆盖原来的构造函数。下面的示例会通过类装饰器封闭类的构造函数和原型，其中@sealed声明在类之前。
```typescript
@sealed
class Person {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}
```
在经过TypeScript编译后，将会生成一个__decorated()函数，并应用到Person类上，如下所示。
```typescript
var Person = /** @class */ (function() {
  function Person(name) {
    this.name = name;
  }
  Person = __decorate([sealed], Person);
  return Person;
})();
```
注意，类装饰器不能出现在.d.ts声明文件和外部类之中。

## 方法装饰器

方法装饰器声明在类的方法之前，作用于方法的属性描述符，比类装饰器还多一个重载限制。它能接收三个参数，如下所列：
1. 对于静态成员来说是类的构造函数，而对于实例成员则是类的原型对象。
2. 成员的名字，一个字符串或符号。
3. 成员的属性描述符，当输出版本低于ES5时，该值将会是undefined。
当方法装饰器返回一个值时，会覆盖当前方法的属性描述符。下面是一个简单的例子，方法装饰器的第一个参数是Person.prototype，第二个是“cover”，调用getName()方法得到的将是“freedom”，而不是原先的“strick”。
```typescript
class Person {
  @cover
  getName(name) {
    return name;
  }
}
function cover(target: any, key: string, descriptor: PropertyDescriptor) {
  descriptor.value = function() {
    return "freedom";
  };
  return descriptor;
}
let person = new Person();
person.getName("strick");        //"freedom"
```

## 访问器装饰器

访问器装饰器声明在类的访问器属性之前，作用于相应的属性描述符，其限制与类装饰器相同，而接收的三个参数与方法装饰器相同。并且还需要注意一点，TypeScript不允许同时装饰一个成员的get和set访问器，只能应用在第一个访问器上。

以下面的Person类为例，定义了一个访问器属性name，当访问它时，得到的将是“freedom”，而不是原先的“strick”。
```typescript
class Person {
  private _name: string;
  @access
  get name() {
    return this._name;
  }
  set name(name) {
    this._name = name;
  }
}
function access(target: any, key: string, descriptor: PropertyDescriptor) {
  descriptor.get = function() {
    return "freedom";
  };
  return descriptor;
}
let person = new Person();
person.name = "strick";
console.log(person.name);        //"freedom"
```

## 属性装饰器

属性装饰器声明在属性之前，其限制与访问器装饰器相同，但只能接收两个参数，不存在第三个属性描述符参数，并且没有返回值。仍然以下面的Person类为例，定义一个name属性，并且在@property装饰器中修改其值。
```typescript
class Person {
  @property
  name: string;
}
function property(target: any, key: string) {
  Object.defineProperty(target, key, {
    value: "freedom"
  });
}
let person = new Person();
person.name = "strick";
console.log(person.name);        //"freedom"
```

## 参数装饰器

参数装饰器声明在参数之前，它没有返回值，其限制与方法装饰器相同，并且也能接收三个参数，但第三个参数表示装饰的参数在函数的参数列表中所处的位置（即索引）。下面用一个例子来演示参数装饰器的用法，需要与方法装饰器配合。
```typescript
let params = [];
class Person {
  @func
  getName(@required name) {
    return name;
  }
}
```
在@func中调用getName()方法，并向其传入params数组中的值，@required用于修改指定位置的参数的值，如下所示。
```typescript
function func(target: any, key: string, descriptor: PropertyDescriptor) {
  const method = descriptor.value;
  descriptor.value = function () {
    return method.apply(this, params);
  };
  return descriptor;
}
function required(target: any, key: string, index: number) {
  params[index] = "freedom";
}
```
当实例化Person类，调用getName()方法，得到的将是“freedom”。
```typescript
let person = new Person();
person.getName("strick");        //"freedom"
```

## 装饰器工厂

装饰器工厂是一个能接收任意个参数的函数，用来包裹装饰器，使其更易使用，它能返回上述任意一种装饰器函数。接下来改造方法装饰器一节中的cover()函数，接收一个字符串类型的value参数，返回一个方法装饰器函数，如下所示。
```typescript
function cover(value: string) {
  return function(target: any, key: string, descriptor: PropertyDescriptor) {
    descriptor.value = function() {
      return value;
    };
    return descriptor;
  };
}
```
在将@cover作用于类中的方法时，需要传入一个字符串，如下所示。
```typescript
class Person {
  @cover("freedom")
  getName(name) {
    return name;
  }
}
```

## 装饰器组合

将多个装饰器应用到同一个声明上时，既可以写成一行，也可以写成多行，如下所示。
```typescript
/****** 一行 ******/
@first @second desc
/****** 多行 ******/
@first 
@second
desc
```
这些装饰器的求值方式与复合函数类似，先由上至下依次执行装饰器，再将求值结果作为函数，由下至上依次调用。例如定义两个装饰器工厂函数，如下代码所示，在函数体和返回的装饰器中都会打印一个数字。
```typescript
function first() {
  console.log(1);
  return function(target: any, key: string, descriptor: PropertyDescriptor) {
    console.log(2);
  };
}
function second() {
  console.log(3);
  return function(target: any, key: string, descriptor: PropertyDescriptor) {
    console.log(4);
  };
}
```
将它们先后声明到类中的同一个方法，如下代码所示。根据求值顺序可知，先打印出1和3，再打印出4和2。
```typescript
class Person {
  @first()
  @second()
  getName(name) {
    return name;
  }
}
```
