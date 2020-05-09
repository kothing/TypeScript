# 高级类型

本节将对TypeScript中类型的高级特性做详细讲解，包括交叉类型、类型别名、类型保护等。

## 交叉类型

交叉类型（Intersection Type）是将多个类型通过“&”符号合并成一个新类型，新类型将包含所有类型的特性。例如有Person和Programmer两个类（如下代码所示），当man变量的类型声明为Person&Programmer时，它就能使用两个类的成员：name属性和work()方法。
```typescript
class Person {
  name: string;
}
class Programmer {
  work() { }
}
let man: Person&Programmer;
man.name;
man.work();
```
交叉类型常用于混入（mixin）或其它不适合典型面向对象模型的场景，例如在下面的示例中，通过交叉类型让新对象obj同时包含a和b两个属性。
```typescript
function extend<T, U>(first: T, second: U): T & U {
  const result = <T & U>{};
  for (let prop in first) {
    (<T>result)[prop] = first[prop];
  }
  for (let prop in second) {
    if (!result.hasOwnProperty(prop)) {
      (<U>result)[prop] = second[prop];
    }
  }
  return result;
}
let obj = extend({ a: 1 }, { b: 2 });
```

## 类型别名

TypeScript提供了type关键字，用于创建类型别名，可作用于基本类型、联合类型、交叉类型和泛型等任意类型，如下所示。
```typescript
type Name = string;                //基本类型
type Func = () => string;          //函数
type Union = Name | Func;          //联合类型
type Tuple = [number, number];     //元组
type Generic<T> = { value: T };    //泛型
```
注意，起别名不是新建一个类型，而是提供一个可读性更高的名称。类型别名可在属性里引用自身，但不能出现在声明的右侧，如下所示。
```typescript
type Tree<T> = {
  value: T;
  left: Tree<T>;
  right: Tree<T>;
}
type Arrs = Array<Arrs>;            //错误
```

## 类型保护

当使用联合类型时，只能访问它们的公共成员。假设有一个func()函数，它的参数是由Person和Programmer两个类组成的联合类型，如下代码所示。
```typescript
function func(man: Person | Programmer) {
  if((<Person>man).run) {
    (<Person>man).run();
  }else {
    (<Programmer>man).work();
  }
}
```
虽然利用类型断言可以确定参数类型，在编译阶段避免了报错，但是多次调用类型断言未免过于繁琐。于是TypeScript就引入了类型保护机制，替代类型断言。类型保护（Type Guard）是一些表达式，允许在运行时检查类型，缩小类型范围。

**typeof**
TypeScript可将typeof运算符识别成类型保护，从而就能直接在代码里检查类型（如下所示），其计算结果是个字符串，包括“number”、“string”、“boolean”或“symbol”等关键字。
```typescript
function send(data: number | string) {
  if (typeof data === "number") {
    //...
  } else if(typeof data === "string") {
    //...
  }
}
```

**instanceof**
TypeScript也可将instanceof运算符识别成类型保护，通过构造函数来细化类型，检测实例和类是否有关联，如下所示。
```typescript
function work(man: Person | Programmer) {
  if (man instanceof Person) {
    //...
  } else if(man instanceof Programmer) {
    //...
  }
}
```

**自定义**
TypeScript还允许自定义类型保护，其形式和函数声明类似，只是返回类型需要改成类型谓词，如下所示。
```typescript
function isPerson(man: Person | Programmer): man is Person {
  return !!(<Person>man).run;
}
```
类型谓词由当前函数的参数名称、is关键字和指定的类型名称所组成。

## 字面量类型

TypeScript可将字符串字面量作为一个类型，用于指定一个字符串类型的固定值。当该类型与联合类型、类型别名等特性配合使用时，可以模拟出枚举的效果，如下所示。
```typescript
type Direction = "Up" | "Down" | "Left";
function move(data: Direction) {
  return data;
}
move("Up");           //正确
move("Right");        //错误
```
move()函数只能接收Direction类型的三个固定值，传入其它值都会产生错误。

字符串字面量类型还可以用来区分函数重载，如下所示。
```typescript
function run(data: "Left"): string;
function run(data: "Down"): string;
function run(data: string) {
  return data;
}
```
其它常见的字面量类型还有数字和布尔值，如下所示。
```typescript
type Numbers = 1 | 2 | 3 | 4 | 5 | 6;
type Bools = true | false;
```
注意，字面量类型属于单例类型。单例类型是一种只有一个值的类型，当每个枚举成员都用字面量初始化时，枚举成员是具有类型的，叫枚举成员类型，它也属于单例类型。

## 可辨析联合

通过合并单例类型、联合类型、类型保护和类型别名可创建一种高级模式：可辨析联合（Discriminated Union），也叫做标签联合或代数数据类型。TypeScript中的可辨析联合具有3个要素：
1. 具有单例类型的属性，即可辨析的特征或标签。
2. 一个联合了多个类型的类型别名。
3. 针对第一个要素中的属性的类型保护。

在下面的示例中，首先声明了两个接口，每个接口都有字符串字面量类型的kind属性，并且其值都不同，而kind属性就是第一个要素中的可辨析的特征或标签。
```typescript
interface Rectangle {
  kind: "rectangle";
  width: number;
  height: number;
}
interface Circle {
  kind: "circle";
  radius: number;
}
```
然后将两个接口联合，并创建一个类型别名，实现第二个要素，如下所示。
```typescript
type Shape = Rectangle | Circle;
```
最后通过具有判断性的kind属性，结合switch语句，执行类型保护，缩小类型范围，如下所示。
```typescript
function caculate(s: Shape) {
  switch (s.kind) {
    case "rectangle":
      return s.height * s.width;
    case "circle":
      return Math.PI * s.radius ** 2;
  }
}
```

**完整性检查**
当未涵盖可辨析联合的所有变化时，需要能反馈到编译器中。例如新增Square接口，并将它添加到Shape类型中（如下所示），如果未更新caculate()函数，那么就不能编译通过。
```typescript
interface Square {
  kind: "square";
  size: number;
}
type Shape = Rectangle | Circle | Square;
```
有两种方法能实现这种预警，第一种是在输入编译命令时添加--strictNullChecks参数，并为caculate()函数指定返回值类型，如下所示。
```typescript
function caculate(s: Shape): number {
  switch (s.kind) {
    case "rectangle":
      return s.height * s.width;
    case "circle":
      return Math.PI * s.radius ** 2;
  }
}
```
由于switch语句没有包含所有类型，因此TypeScript会认为该函数有可能返回undefined，从而就会编译报错。注意，这种方法不太精确，有很多因素（例如函数默认返回数字）会干扰完整性检查，并且--strictNullChecks参数对旧代码有兼容问题。

第二种方法是使用never类型，如下代码所示，新增一个能引发类型错误的assertNever()函数，并在default分支中调用该函数。
```typescript
function assertNever(x: never): never {
  throw new Error("Unexpected object: " + x);
}
function caculate(s: Shape) {
  switch (s.kind) {
    case "rectangle":
      return s.height * s.width;
    case "circle":
      return Math.PI * s.radius ** 2;
    default:
      return assertNever(s);
  }
}
```
虽然额外定义了一个函数，但是检查的精确度提升了不少。

## 索引类型

索引类型（Index Type）能让编译器检查使用动态属性的场景，例如从对象中选取属性的子集，如下所示。
```typescript
function pluck(obj, names) {
  return names.map(n => obj[n]);
}
```
如果要让pluck()函数能从obj对象中成功的选出names数组所指定的属性，那么需要在声明时设置类型约束，包括names中的元素必须是obj中存在的属性以及返回值类型得是obj属性值的类型，下面通过泛型来描述这些约束。
```typescript
function pluck<T, K extends keyof T>(obj: T, names: K[]): T[K][] {
  return names.map(n => obj[n]);
}
interface Person {
  name: string;
  age: number;
}
let person: Person = {
  name: "strick",
  age: 28
};
let attrs: string[] = pluck(person, ["name"]);
```
　　泛型函数pluck()引入了两个新的类型操作符，分别是索引类型查询操作符（keyof T）和索引访问操作符（T[K]）。前者会取T类型中由公共（public）属性名所组成的联合类型，例如“"name" | "age"”；后者会取T类型中指定属性值的类型，这意味着示例中的person["name"]和Person["name"]两者的类型都是string。

**字符串索引签名**
keyof T与T[K]同样适用于字符串索引签名，以下面的泛型接口People为例，kType的类型是string和number的联合类型，因为JavaScript里的数值索引会自动转换成字符串索引；vType的类型是number，也就是索引签名的类型。
```typescript
interface People<T> {
  [key: string]: T;
}
let kType: keyof People<number>;         //string | number
let vType: People<number>["name"];       //number
```

## 映射类型

映射类型（Mapped Type）与索引类型类似，也是从现有类型中创建出一种新类型。接下来用一个例子来演示映射类型用法，假设有一个Person接口，它有两个成员，如下所示。
```typescript
interface Person {
  name: string;
  age: number;
}
```
当需要将Person接口的每个成员都变为可选或只读的，粗糙的解决方法是一个个的修改，如下所示。
```typescript
interface PersonPartial {
  name?: string;
  age?: number;
}
interface PersonReadonly {
  readonly name: string;
  readonly age: number;
}
```
而如果采用映射类型，那么就能快速的改变接口成员，如下代码所示，其中Readonly<T>可将T类型的成员设为只读，而Partial<T>会将它们设为可选。
```typescript
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
}
type Partial<T> = {
  [P in keyof T]?: T[P];
}
type PersonPartial = Partial<Person>;
type ReadonlyPerson = Readonly<Person>;
```
[P in keyof T]的语法与索引类型的类似，但内部使用了for-in遍历语句，其中：

1. P是类型变量，会依次绑定到每个成员上，对应成员名的类型。
2. T是由字符串字面量构成的联合类型，表示一组成员名，例如“"name" | "age"”。
3. T[P]是成员值的类型。

注意，映射类型描述的是类型而非成员，如果要添加额外的成员，需要使用交叉类型的方式，如下所示，直接在类型中添加成员会无法通过编译。
```typescript
//交叉类型
type ReadonlyNew<T> = {
  readonly [P in keyof T]: T[P];
} & { data: boolean };
//编译错误
type ReadonlyNew<T> = {
  readonly [P in keyof T]: T[P];
  data: boolean;
};
```
`Readonly<T>`和`Partial<T>`是一种同态转换，即在映射时保留源类型的成员名以及其值类型，并且与目标类型相比只有修饰符有差异。而那些会创建新成员、改变成员类型或其值类型的转换都被称为非同态。由于Readonly<T>和Partial<T>很实用，因此它们已经被包含进TypeScript的标准库里，作为内置的工具类型存在。

## 条件类型

条件类型（Conditional Type）能够表示非统一的类型映射，常以条件表达式进行类型检测，语法类似于三目运算符，从两个类型中选出一个，如下所示。
```typescript
T extends U ? X : Y
```
如果T是U的子类型，那么类型将被解析成X，否则是Y。当条件的真假无法确定时，得到的结果将是由X和Y组成的联合类型，以下面的全局函数sum()为例，T是布尔值的子类型，当传入的参数是true时，得到的将是string类型；而传入false时，得到的是number类型。
```typescript
declare function sum<T extends boolean>(x: T): T extends true ? string : number;
let x = sum(true);        //string | number
```
如果T或U包含类型变量，那么就得延迟解析，即等到类型变量都有具体类型后才能计算出条件类型的结果。在下面的示例中，创建了一个Person接口，声明的全局函数add()的返回值类型会根据是否是Person的子类型而改变，并且在泛型函数func()中调用了add()函数。
```typescript
interface Person {
  name: string;
  age: number;
  getName(): string;
}
declare function add<T>(x: T): T extends Person ? string : number;
function func<U>(x: U) {
  let a = add(x);
  let b: string | number = a;
}
```
虽然a变量的类型尚不确定，但是条件类型的结果不是string就是number，因此可以成功的赋给b变量。

**分布式条件类型**
当条件类型中被检查的类型是无类型参数（naked type parameter）时，它会被称为分布式条件类型（Distributive Conditional Type）。其特殊之处在于它能自动分布联合类型，举个简单的例子，假设T的类型是A | B | C，那么它会被解析成三个条件分支，如下所示。
```typescript
(A | B | C) extends U ? X : Y
//等同于
(A extends U ? X : Y) | (B extends U ? X : Y) | (C extends U ? X : Y)
```
分布式条件类型可以用来过滤联合类型，如下所示，Filter<T, U>类型可从T中移除U的子类型。
```typescript
type Filter<T, U> = T extends U ? never : T;
type T1 = Filter<"a" | "b" | "c" | "d", "a" | "c" | "f">;        // "b" | "d"
type T2 = Filter<string | number | (() => void), Function>;      // string | number
```
分布式条件类型也可与映射类型配合使用，进行针对性的类型映射，即不同源类型对应不同映射规则，例如映射接口的方法名，如下所示。
```typescript
type FunctionPropertyNames<T> = { 
       [K in keyof T]: T[K] extends Function ? K : never
    }[keyof T];
type T3 = FunctionPropertyNames<Person>;      // "getName"
```
注意，条件类型与联合类型和交叉类型相似，不允许递归地引用自身，下面这样写会在编译阶段报错。
```typescript
type Custom<T> = T extends any[] ? Custom<T[number]> : T;
```

**类型推断**
在条件类型的extends子句中，允许通过infer声明引入一个待推断的类型变量，并且可出现多个同类型的infer声明，例如用infer声明来提取函数的返回值类型，如下所示。有一点要注意，只能在true分支中使用infer声明的类型变量。
```typescript
type Func<T> = T extends (...args: any[]) => infer R ? R : any;
```
当函数具有重载时，就取最后一个函数签名进行推断，如下所示，其中ReturnType<T>是内置的条件类型，可获取函数类型T的返回值类型。
```typescript
declare function load(x: string): number;
declare function load(x: number): string;
declare function load(x: string | number): string | number;
type T4 = ReturnType<typeof load>;          // string | number
```
注意，无法在正常类型参数的约束子语句中使用infer声明，如下所示。
```typescript
type Func<T extends (...args: any[]) => infer R> = R;
```
但是可以将约束里的类型变量移除，并将其转移到条件类型中，就能达到相同的效果，如下所示。
```typescript
type AnyFunction = (...args: any[]) => any;
type Func<T extends AnyFunction> = T extends (...args: any[]) => infer R ? R : any;
```

**预定义的条件类型**
除了之前示例中用到的ReturnType<T>之外，TypeScript还预定义了4个其它功能的条件类型，如下所列。
1. Exclude<T, U>：从T中移除掉U的子类型。
2. Extract<T, U>：从T中筛选出U的子类型。
3. NonNullable<T>：从T中移除null与undefined。
4. InstanceType<T>：获取构造函数的实例类型。
```typescript
type T11 = Exclude<"a" | "b" | "c" | "d", "a" | "c">;    // "b" | "d"
type T12 = Extract<"a" | "b" | "c" | "d", "a" | "c">;    // "a" | "c"
type T13 = NonNullable<string | number | undefined>;     // string | number
type T14 = ReturnType<(s: string) => void>;              // void
class Programmer {
  name: string;
}
type T15 = InstanceType<typeof Programmer>;              //Programmer
```
