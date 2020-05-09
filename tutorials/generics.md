# 泛型

泛型是程序设计语言中的一种风格或范式，相当于类型模板，允许在声明类、接口或函数等成员时忽略类型，而在未来使用时再指定类型，其主要目的是为它们提供有意义的约束，提升代码的可重用性。

## 泛型参数
当一个函数需要能处理多种类型的参数和返回值，并且还得约束它们之间的关系（例如类型要相同）时，就可以采用泛型的语法，如下所示。
```typescript
function send<T>(data: T): T {
  return data;
}
```
函数名称后面跟了<T>，其中把T称为泛型参数或泛型变量，表示某种数据类型。注意，T只是个占位符，可以命名的更含语义，例如TKey、TValue等。在使用时，既可以指定类型，也可以利用类型推论自动确定类型，如下所示。
```typescript
send<number>(10);        //指定类型
send(10);            　　//类型推论
```
当需要处理T类型的数组时，可以像下面这么写。
```typescript
function send<T>(data: T[]): T[] {
  return data;
}
send<number>([1, 2, 3]);
```
当指定一个泛型函数的类型时，需要包含泛型参数，如下所示，其中泛型参数和函数参数的名称都可与定义时的不同。
```typescript
let func: <U>(data: U) => U = send;
```
泛型参数还支持传递多个，只需在声明时增加类型占位符即可。在下面的示例中，将T和U合并成了一个元组类型，还有许多其它用法，将在后面讲解。
```typescript
function send<T, U>(data: [T, U]): [T, U] {
  return data;
}
send<number, string>([1, "a"]);
```

## 泛型接口

在接口中，可利用泛型来约束函数的结构，如下所示，接口中声明的调用签名包含泛型参数。
```typescript
interface Func {
  <T>(str: T): T;
}
function send<T>(str: T): T {
  return str;
}
let fn: Func = send;
```
泛型参数还可以作为接口的一个参数存在，即把用尖括号包裹的泛型参数移到接口名称之后，如下所示。
```typescript
interface Func<T> {
  (str: T): T;
}
function send<T>(str: T): T {
  return str;
}
let fn: Func<string> = send;
```
当把Func接口作为类型使用时，需要向其传入一个类型，例如上面赋值语句中的string。


## 泛型类

泛型类与泛型接口类似，也是在名称后添加泛型参数，如下所示，其中send属性中的“=>”符号不表示箭头函数，而是用来定义方法的返回值类型。
```typescript
class Person<T> {
  name: T;
  send: (data: T) => T;
}
```
在实例化泛型类时，需要为其指定一种类型，如下所示。
```typescript
let person = new Person<string>();
person.send = function(data) {
  return data;
}
```
注意，类的静态部分不能使用泛型参数。


## 泛型约束

在使用泛型时，由于事先不清楚参数的数据类型，因此不能随意调用它的属性或方法，甚至无法对其使用运算符。在下面的示例中，访问了data的length属性，但由于编译器无法确定它的类型，因此就会报错。
```typescript
function send<T>(data: T): T {
  console.log(data.length);
  return data;
}
```
TypeScript允许为泛型参数添加约束条件，从而就能调用相应的属性或方法了，如下所示，通过extends关键字约束T必须是string的子类型。
```typescript
function send<T extends string>(data: T): T {
  console.log(data.length);
  return data;
}
```
在添加了这个约束之后，send()函数就无法接收数字类型的参数了，如下所示。
```typescript
send("10");        //正确
send(10);          //错误
```

**创建类的实例**

在使用泛型创建类的工厂函数时，需要声明T类型拥有构造函数，如下所示。
```typescript
class Programmer { }
function create<T>(ctor: {new(): T}): T {
  return new ctor();
}
create(Programmer);
```
用“{new(): T}”替代原先的类型占位符，表示可以被new运算符实例化，并且得到的是T类型，另一种相同作用的写法如下所示。
```typescript
function create<T>(ctor: new()=>T): T {
  return new ctor();
}
```

**多个泛型参数**

在TypeScript中，多个泛型参数之间也可以相互约束，如下所示，创建了基类Person和派生类Programmer，并将create()函数中的T约束为U的子类型。
```typescript
class Person { }
class Programmer extends Person { }
function create<T extends U, U>(target: T, source: U): T {
  return target;
}
```
当传递给create()函数的参数不符合约束条件时，就会在编译阶段报错，如下所示。
```typescript
create(Programmer, Person);        //正确
create(Programmer, 10);            //错误
```
