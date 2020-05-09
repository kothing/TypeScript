# 接口

在传统的面向对象语言中，接口（Interface）好比协议，它会列出一系列的规则（即对行为进行抽象），再由类来实现这些规则。 在 TypeScript 里，接口的作用就是为这些类型命名和为你的代码或第三方代码定义契约，换句话说，接口就是声明了实例代码必须遵守的语法规则集合。

## 声明
使用关键字interface声明接口，语法 :
```typescript
interface interface_name { 
  //属性成员
  //方法成员
  //事件成员
}
```

## 属性
　　TypeScript中的接口可通过声明属性和其类型来限制对象的结构。例如定义一个名为Person的接口，包含一个字符串类型的name属性和一个数字类型的age属性，如下所示。
```typescript
interface Person {
  name: string;
  age: number;
}
```
　　当声明一个Person类型的对象时，必须将两个属性都定义，并且类型也要与接口中的一致，如下所示。
```typescript
let worker: Person = {
  name: "strick",
  age: 28
};
```
　　一旦在worker对象中少定义某个接口中的属性或多一个在接口中未声明的属性，那么就会在编译阶段报错。注意，TypeScript的类型检查器不会比对属性在接口和对象中的定义顺序，只要名称和类型匹配，就能编译通过。

**可选属性**

　　TypeScript允许接口中的属性定义为可选的，只要在属性名后跟问号（?），就能变为可选属性，如下所示。
```typescript
interface Person {
  school?: string;
}
```
　　可选属性既能预定义可能需要的属性，也能在捕获没有的属性时给出带有启发作用的错误提示，例如在创建worker对象时，定义一个schools属性（如下所示），在编译时就会报"'schools' does not exist in type 'Person'. Did you mean to write 'school'?"的错误。
```typescript
let worker: Person = {
  schools: "university"
};
```
　　由此可知，在对象中定义一个未在接口中声明的属性仍然是不允许的。

**只读属性**

　　如果要让对象的某个属性只能在创建时被赋值，那么可以将readonly关键字作用于相应的接口属性，使其变为只读的，如下所示。
```typescript
interface Person {
  readonly gender: string;
}
```
　　由于gender是一个只读属性，因此不能在对象初始化后对其进行修改，如下所示。
```typescript
let worker: Person = {    //正确
  gender: "男"
};
worker.gender = "女";     //错误
```

**任意属性**

　　当接口需要包含任意属性时，可以通过索引的方式实现，如下所示，用方括号将索引名和索引类型包裹起来。

```typescript
interface Person {
  [prop: string]: string;
}
```
　　在使用Person类型时，可以传任意多个字符串类型的属性，如下所示。

```typescript
let worker: Person = {
  name: "strick",
  gender: "男"
};
```
　　注意，一旦声明了任意属性之后，那么必选属性和可选属性都得是其类型的子类型。在下面的示例中，由于可选的age属性的类型是number，不是string的子类型，因此在编译时会报错。

```typescript
interface Person {
  name: string;                //正确
  age?: number;                //错误
  [prop: string]: string;
}
```
　　TypeScript除了支持字符串类型的索引之外，还支持数字类型的索引，如下所示。

```typescript
interface Person {
  [prop: number]: number;
}
```
　　有一点需要注意，当在接口中同时定义字符串和数字两种类型的索引时，后者对应的值类型得是前者的子类型。因为这个原因，导致下面的代码无法在编译时通过。

```typescript
interface Person {
  [prop: string]: string;
  [prop: number]: number;            //错误
}
```
　　TypeScript之所以如此限制，是因为JavaScript会将数字自动转换成字符串后再去索引对象，例如用10和“10”两个值去索引，得到的结果是一样的，所以两种索引对应的值类型要保持一致。

## 继承

**类继承接口**

　　与C#、Java等面向对象语言一样，TypeScript中的类也能继承接口，并且接口中的成员会让类强制实现。有了接口之后，它的任何更改都有可能导致编译错误，从而就能保证相关代码的同步。下面通过一个示例来演示类继承接口，首先创建一个名为Person的接口，包含name属性和getName()方法，如下所示。

```typescript
interface Person {
  name: string;
  getName(): string;
}
```
　　然后再创建一个名为Member的类，通过implements关键字继承Person接口，如下所示。在编译时，一旦发现类中缺少接口的属性或方法，就会马上报错。

```typescript
class Member implements Person {
  name: string = "strick";
  getName() {
    return this.name;
  }
}
```
　　类能继承多个接口，只要在类中实现它的成员，就能编译成功，如下所示，Member类继承了Person和Profile两个接口，限于篇幅原因，在其内部省略了name和getName()两个成员的实现。

```typescript
interface Profile {
  school: string;
}
class Member implements Person, Profile {
  school: string = "university";
}
```
　　注意，类不能实现接口中的所有成员，例如在接口中定义一个构造器，再用一个类通过构造函数来实现这个接口，此时编译将会失败，代码如下所示。

```typescript
interface Person {
  new (name: string);
}
class Member implements Person {
  constructor(name: string) { }
}
```
　　类包含静态和实例两部分，由于编译器只会对接口的实例部分进行类型检查，而constructor()函数属于类的静态部分，因此会被忽略，从而导致无法在类中找到匹配的成员来实现接口。

　　如果要实现接口中的构造器，那么有两种方式可供选择。第一种是参数回调，如下代码所示，Member类不再直接继承Person接口，而是作为参数传递给createPerson()函数，并且其第一个参数被声明为Person类型。

```typescript
class Member {
  constructor(name: string) { }
}
function createPerson(ctor: Person, name: string) {
  return new ctor(name);
}
createPerson(Member, "strick");
```
　　第二种是类表达式，如下代码所示，将Man变量声明为Person类型，并把Member类赋给它。

```typescript
let Man: Person = class Member {
  constructor(name: string) { }
}
```

**接口继承接口**

　　接口之间也可相互继承，这样既能更细粒度的分割接口，也能最大化的重用代码。与类不同的是，只需将其它的接口成员复制过来，而不必实现它们。在下面的示例中，Square接口通过extends关键字继承了Shape接口。

```typescript
interface Shape {
  background: string;
}
interface Square extends Shape {
  width: number;
}
```
　　一个接口还可以继承多个其它接口，创建出一个合成接口，如下所示，extends后面跟了Shape和Border两个接口。

```typescript
interface Border {
  color: string;
}
interface Ellipse extends Shape, Border {
  angle: string;
}
```

**接口继承类**

　　当接口继承一个类时，它会继承类的所有成员（包括私有和受保护的成员），但不会去实现它们。以下面的TextBox接口为例，它继承了Control类。

```typescript
class Control {
  private width: number;
  protected height: number;
}
interface TextBox extends Control {
  type: string;
}
class Tel implements TextBox {
  type: string = "tel";
}
```
　　上例中的Tel类直接继承了TextBox接口，虽然实现了接口中的type属性，但仍然会报“Type 'Tel' is missing the following properties from type 'TextBox': width, height”的错误。因为Button接口继承的width和height两个属性也需要实现。为了避免出现这些错误，可以通过Control的子类来实现TextBox接口，如下所示。

```typescript
class Password extends Control implements TextBox {
  type: string = "password";
}
```
