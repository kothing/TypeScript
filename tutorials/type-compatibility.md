
# 类型兼容性

TypeScript是一种基于结构类型的语言，可根据其成员来描述类型。以结构相同的Person接口和Programmer类为例，如下所示。
```typescript
interface Person {
  name: string;
}
class Programmer {
  name: string;
}
let person: Person = new Programmer();
```
由于结构类型的关系，因此当变量声明为Person类型时，可通过Programmer类实例化。由此可知，结构类型只关注类型的组成结构，而名称并不重要。

## 函数
在判断两个函数的兼容性时，需要考虑参数数量、返回值类型等多个方面。

**参数数量**

假设有两个函数add()和sum()，它们的参数数量不同，后者比前者多一个参数，而函数的返回值类型相同，如下所示。
```typescript
let add = (x: number) => 0;
let sum = (y: number, z: number) => 0;
```
当把add赋给sum时，编译能成功执行；而反之，则会报错，如下所示。
```typescript
sum = add;        //正确
add = sum;        //错误
```
由此可知，参数少的函数不能赋给参数多的；而反过来时，需要确保每个位置所对应的参数类型保持一致，不过参数名称可以不同。

**返回值类型**
假设有两个函数add()和sum()，它们没有参数，后者的返回值比前者多一个属性，如下所示。
```typescript
let add = () => ({ x: 1 });
let sum = () => ({ x: 1, y: 2});
```
　　当把sum赋给add时，编译能成功执行；而反之，则会报错，如下所示。
```typescript
add = sum;        //正确
sum = add;        //错误
```
由此可知，源函数的返回值类型得是目标函数返回值的子类型。

**参数类型**
TypeScript中的参数类型需要同时满足协变和逆变，即双向协变。协变比较好理解，是指子类型兼容父类型，而逆变正好与协变相反。在下面的示例中，定义了父类Person和子类Programmer，Programmer类覆盖了Person类中的work()方法，并且其参数类型声明的更加宽泛。
```typescript
class Person {
  work(msg: string | undefined) { }
}
class Programmer extends Person {
  work(msg: string) { }
}
```
接下来声明两个函数，它们的参数类型分别是Person和Programmer两个类，如下所示，其中person()函数是programmer()函数的子类型。
```typescript
let person = (x: Person) => 0;
let programmer = (x: Programmer) => 0;
```
由于参数类型是双向协变的，因此两个变量之间可相互赋值，如下所示。
```typescript
person = programmer;
programmer = person;
```

**可选参数和剩余参数**
在比较函数的兼容性时，不需要匹配可选参数。以下面的pls()和add()两个函数为例，pls()中的两个参数必传，而add()中的第二个参数是可选的。
```typescript
let pls = (x: number, y: number) => 0;
let add = (x: number, y?: number) => 0;
pls = add;
add = pls;
```
虽然参数不同，但是两个函数仍然是兼容的，并且可以相互赋值。剩余参数相当于无限个可选参数，也不会被匹配。下面示例中的sum()函数只声明了剩余参数，它与pls()和add()两个函数都是兼容的。
```typescript
let sum = (...args: number[]) => 0;
pls = sum;
sum = pls;

add = sum;
sum = add;
```

**函数重载**
当比较存在多个重载的函数时，其每个重载都要在目标函数上找到对应的函数签名，以此确保目标函数能在源函数所有可调用的地方调用，如下所示。
```typescript
interface add {
  (x: number, y: string): any;
  (x: number, y: number): number;
}
function sum(x: number, y: string): any;
function sum(x: number, y: number): number;
let func: add = sum;
```

## 枚举

来自于不同枚举类型的枚举值，被认为是不兼容的，如下所示，当把Direction.Up赋给color变量时，在编译阶段会报错。
```typescript
enum Color { Red, Green, Blue }
enum Direction { Up, Down, Left, Right }
let color = Color.Red;        //正确
color = Direction.Up;         //错误
```
数字枚举和数字类型相互兼容，如下所示，color变量被赋予了枚举成员，digit变量是一个数字，它们之间可以相互赋值。
```typescript
let color = Color.Red;
let digit = 1;
color = digit;
digit = color;
```
字符串枚举无法兼容字符串类型，如下所示，当把field变量赋给Color的枚举成员时，在编译阶段会报错，但反过来可以正确执行。
```typescript
enum Color { Red = "RED", Green = "GREEN", Blue = "BLUE" }
let color = Color.Red;
let field = "PURPLE";
color = field;            //错误
field = color;            //正确
```

## 类

类与对象字面量和接口类似，但类包含静态和实例两部分。在比较两个类实例时，仅匹配它们的实例成员，而静态成员和构造函数不影响兼容性，因为它们在比较时会被忽略。

在下面的示例中，创建了Person和Programmer两个类，虽然Programmer类包含了一个静态属性，并且其构造函数与Person类不同，但是它们之间可以相互兼容。
```typescript
class Person {
  name: string;
  constructor(name: string) { }
}
class Programmer {
  name: string;
  static age: number;
  constructor(name: string, age: number) { }
}

let person: Person;
let programmer: Programmer;
person = programmer;
programmer = person;
```
类的私有成员和受保护成员会影响兼容性，TypeScript要求它们必须来源于同一个类，从而既能保证父类兼容子类，也能避免与其它相同结构的类兼容。

在下面的示例中，Person和Teacher两个类都包含一个同名的私有属性，Programmer是Person的子类，三个变量的类型对应这三个类。
```typescript
class Person {
  private name: string;
}
class Teacher {
  private name: string;
}
class Programmer extends Person { }

let person: Person;
let programmer: Programmer;
let teacher: Teacher;
```
person和programmer两个变量可相互赋值，因为它们的私有成员来源于同一个类，如下所示。
```typescript
person = programmer;
programmer = person;
```
虽然person和teacher两个变量的结构相同，但是它们的私有成员来源于两个不同的类，因此无法相互赋值，如下所示。
```typescript
person = teacher;        //错误
teacher = person;        //错误
```

## 泛型

当泛型接口中的类型参数未使用时，不会影响其兼容性，如下所示，x和y两个变量可相互赋值。
```typescript
interface Person<T> { }
let x: Person<number>;
let y: Person<string>;

x = y;
y = x;
```
当泛型接口中的类型参数被一个成员使用时，就会影响其兼容性，如下所示，x和y两个变量不可相互赋值。
```typescript
interface Person<T> {
  data: T;
}
let x: Person<number>;
let y: Person<string>;

x = y;        //错误
y = x;        //错误
```
当比较未指定参数类型的泛型函数时，在检查兼容性之前会将其替换成any类型，例如下面的两个函数，相当于对“(x: any)=>any”和“(y: any)=>any”进行匹配，因此可相互赋值。
```typescript
let send = function<T>(x: T): T {
  return x;
}
let func = function<U>(y: U): U {
  return y;
}

send = func;
func = send;
```
泛型类的兼容性规则与之前所述一致。
