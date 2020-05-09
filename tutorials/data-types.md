TypeScript不仅支持JavaScript所包含的数据类型，还额外扩展了许多实用的数据类型，例如枚举、空值、任意值等。

## 一、JavaScript的数据类型
　　JavaScript的数据类型包括6种基本类型：undefined、null、布尔值、数字、字符串以及ES6新增的Symbol，还有1种复杂类型：object。由于TypeScript提供了可选的静态类型声明（即在变量后跟一个冒号和类型声明），因此同样的变量声明，在TypeScript中将更能传播代码的意图，并且在编译时还能验证代码的正确性。在下面的代码中，声明了6种类型（不包括Symbol），并且引入了ES6新增的二进制、八进制和模板字面量。

```typescript
let isDone: boolean = false;

let decimal: number = 10;　　　　　　 //十进制
let hex: number = 0xa;　　　　　　　　//十六进制
let binary: number = 0b1010;　　　　 //二进制
let octal: number = 0o12;　　　　　　//八进制

let name: string = "strick";
let template: string = `my name is ${name}`;    //模板字面量

let u: undefined = undefined;
let n: null = null;

let obj: object = {};
```

## 二、TypeScript扩展的类型

1）任意值

　　当变量声明为任意值（any类型）时，它在编译阶段会跨过类型检查，并且能被赋为任意类型的值，还允许访问任意属性、调用任意方法。在下面的示例中，虽然能编译通过，但是在使用时却会抛出错误，因为字符串类型的变量没有toFixed()方法。
```typescript
let data: any = 10.5;
data = "strick";
data.toFixed();    //错误
```
　　注意，没有显式声明类型的变量，默认都是any类型的。

2）空值

　　在JavaScript中，没有空值（void类型）的概念。TypeScript中的void不表示任意类型，与any类型相悖。当一个函数没有返回值时，通常会将其返回值的类型声明成void，如下所示。
```typescript
function send(): void { }
```
　　如果声明一个void类型的变量，那么它的值只能是undefined或null，如下所示。
```typescript
let u: void = undefined;
let n: void = null;
```

3）Never

　　never类型表示那些永不存在的值的类型，例如包含死循环不会有返回值的函数或抛出错误的函数，如下所示。
```typescript
function loop(): never {
  while (true) {}
}
function error(message: string): never {
  throw new Error(message);
}
```
　　注意，当一个函数没有返回值时，它返回一个void类型；而当函数被意外中断时，它返回一个never类型。

　　由于never类型是所有类型的子类型，因此可被赋给任意类型。但是除了其自身之外，其它类型（包括any）都不能赋给它，如下所示。
```typescript
let none: never;
let digit: number = none;        //正确
let figure: never = 10;          //错误
```

4）数组

　　在TypeScript中，有两种常见的数组声明方式。第一种通过类型和方括号组合来表示数组，第二种是使用数组泛型，如下所示。
```typescript
let arr1: number[] = [1, 2, 3];
let arr2: Array<number> = [1, 2, 3];
```
　　在指定了元素类型之后，就不能添加其它类型的元素，例如为数组的push()方法传入一个字符串数字，如下所示，在编译时会报错。
```typescript
arr1.push("4");
　　如果数组需要包含各种类型的元素，那么可以将其声明成any类型，如下所示。

let arr3: any[] = [1, "2", true];
```

5）元组

　　在TypeScript中，元组（Tuple）会合并不同类型的值，例如定义一对string和number两种类型的元组，如下所示。
```typescript
let list: [string, number];
```
　　当为元组赋值时，需要指定相应类型的元素，即先传字符串，后传数字，下面代码的第二次赋值没有按照这个顺序，因此在编译时将报错。
```typescript
list = ["strick", 10];        //正确
list = [10, "strick"];        //错误
```
　　也可以通过索引为元组添加元素，但也要遵守类型限制，如下所示。
```typescript
list[0] = "strick";
list[1] = 10;
```
　　当添加越界的元素时，只要该元素是元组的联合类型（既可以是字符串，也可以是数字），就能编译成功，如下所示。
```typescript
list.push(10);            //正确
list.push(true);          //错误
```
　　在编译第二条语句时，会报“Argument of type 'true' is not assignable to parameter of type 'string | number'.”的错误。

## 三、枚举
　　枚举是一个被命名的常量集合，该类型也是对JavaScript标准数据类型的一个补充。和C#、Java等其它语言一样，使用枚举可以更清晰的表达代码意图。TypeScript支持数字的和字符串两种类型的枚举。

1）数字枚举

　　默认情况下，定义的都是数字枚举，如下所示，其中枚举成员的值从0开始自增长，例如Up的值为0，Down的值为1，其余依次递增。

```typescript
enum Direction { Up, Down, Left, Right }
```
　　通过枚举名可以正向映射得到枚举值，而通过枚举值也可以反向映射得到枚举名，如下所示。
```typescript
Direction.Up;        //0
Direction[0];        //"Up"
```
　　由于TypeScript中的枚举会被编译成下面这样，因此才能通过两种映射方式分别得到枚举名和枚举值。
```typescript
var Direction;
(function (Direction) {
    Direction[Direction["Up"] = 0] = "Up";
    Direction[Direction["Down"] = 1] = "Down";
    Direction[Direction["Left"] = 2] = "Left";
    Direction[Direction["Right"] = 3] = "Right";
})(Direction || (Direction = {}));
```
　　数字枚举也支持手动赋值，例如将上例中的Direction从1开始递增，如下所示，Down的值为2。注意，定义的数字还可以是小数、负数等各种与数字兼容的值。
```typescript
enum Direction { Up = 1, Down, Left, Right }
```
　　或者也可以为每个枚举成员都赋值，如下所示。
```typescript
enum Direction { Up = 1, Down = 3, Left = 5, Right = 7 }
```

2）字符串枚举

　　在字符串枚举中，每个成员的值都得是字符串类型的，如下所示。
```typescript
enum Color { Red = "RED", Green = "GREEN", Blue = "BLUE" }
```
　　字符串枚举提供了有意义、可调试的字符串，常用于简单的值比较，如下所示。
```typescript
if (colorName === Color.Red) {
  console.log("success");
}
```

3）异构枚举

　　枚举还可以混合字符串和数字两种成员，如下所示。
```typescript
enum Mix { Up = 1, Red = "RED" }
```

4）枚举成员

　　每个枚举成员都会包含一个值，而根据值的来源可将成员分成常量成员（Constant Member）和计算成员（Computed Member）。

　　之前示例中的枚举成员都是常量，除此之外，当枚举成员通过常量枚举表达式初始化时，也会成为常量成员。常量枚举表达式是TypeScript表达式的子集，可在编译阶段求值。当一个表达式满足下面一个条件时（引用自官方文档），它就是一个常量枚举表达式：

　　1. 枚举表达式字面量，例如字符串字面量或数字字面量。

　　2. 一个对之前定义的常量成员的引用，可以在不同的枚举类型中。

　　3. 带括号的常量枚举表达式。

　　4. 将+、-、~三个一元运算符中的一个应用到常量枚举表达式中。

　　5. 常量枚举表达式作为二元运算符+、-、*、/、%、<<、>>、>>>、&、|或^的操作对象。
```typescript
enum Con {
  Red = 1, 
  Green = Direction.Down,
  Blue = -(1 << 1),
  Yellow = Red & Blue
}
```
　　不满足上述条件的枚举成员会被当作计算成员来使用，并且要注意，计算成员之后，都需要手动赋值，否则会在编译阶段报错。下面这个枚举就会编译失败。
```typescript
enum Computed {
  Red = "red".length, 
  Green,
  Blue
}
```

5）常量枚举

　　在声明枚举时添加const关键字就能生成常量枚举，如下所示。
```typescript
const enum Colors { Red, Green, Blue }
```
　　常量枚举只能使用常量枚举表达式，不能包含计算成员，并且会在编译阶段被删除，其成员在被引用到时才会被内联进来，例如将上例Colors中的成员组成一个数组，其编译结果如下所示。
```typescript
let colors = [Colors.Red, Colors.Green, Colors.Blue];
//编译结果
var colors = [0 /* Red */, 1 /* Green */, 2 /* Blue */];
```

6）外部枚举

　　在声明枚举时添加declare关键字就能生成外部枚举，即全局枚举，如下所示。
```typescript
declare enum Externals { Red, Green, Blue }
```
　　外部枚举用于描述已经存在的枚举类型，并且在编译结果中会移除declare声明的枚举，例如将Externals的成员组成一个数组。
```typescript
let externals = [Externals.Red, Externals.Green, Externals.Blue];
```
　　在运行时调用externals时，如果没有定义Externals对象，那么就会报错。

　　declare还可以与其它关键字（例如var、function、class等）配合，声明全局变量、全局函数、全局类等。

## 四、类型断言
　　类型断言可指定一个值的类型，类似于类型转换，但它只在编译阶段起作用，并且不影响运行时的结果。类型断言包含两种语法形式，第一种是尖括号语法，第二种是as语法，如下所示。当在TypeScript中使用JSX时，只支持as语法。
```typescript
let age: number = 28;
let digit = <number>age;        　　//语法一
let figure = age as number;        //语法二
```

## 五、联合类型
　　联合类型（Union Type）可让一个变量拥有多种类型，在语法上，通过竖线（|）来分隔每个类型，例如下面的data变量，既可以是字符串，也可以是数字。
```typescript
let data: number | string;
data = 10;
data = "strick";
```
　　注意，当访问联合类型的成员时，只能访问它们共有的成员。以上面示例的data变量为例，它能成功调用toString()方法，但不能访问length属性（如下所示），因为length属性只存在于string中，而toString()方法是两者共有的。
```typescript
data.toString();    //正确
data.length;        //错误
```

## 六、函数
　　TypeScript中的函数不仅包含ES6的默认参数、剩余参数等功能，还新增了许多额外的功能，例如类型声明、重载等。

1）函数创建

　　在创建函数时可以为其参数和返回值添加类型，从而起到约束的作用。在下面的示例中，通过两种方式创建函数，第一种是函数声明，第二种是函数表达式。
```typescript
function add(x: number, y: number): number {            　　 //第一种
  return x + y;
}
let minus = function(x: number, y: number): number {        //第二种
  return x - y;
};
```
　　由于TypeScript能根据return语句推断出返回值的类型，因此可以省略该类型的声明。注意，如果在调用函数时传递多余的参数，那么在编译时就会报错。
```typescript
function add(x: number, y: number) {        //正确
  return x + y;
}
add(1, 2, 3);
```
　　TypeScript还可以为函数表达式右侧的变量添加类型，如下所示，其中“=>”符号不表示箭头函数，而是用来定义函数的返回值类型，并且返回值类型必须指定。
```typescript
let minus: (x: number, y: number) => number = 
    function(x: number, y: number): number { return x - y; };        //正确
let minus: (x: number, y: number) = 
    function(x: number, y: number): number { return x - y; };        //错误
```
　　注意，两处的参数名称可以不一致，只要类型匹配即可，如下所示。
```typescript
let minus: (left: number, right: number) => number = 
    function(x: number, y: number): number { return x - y; };        //正确
```

2）可选参数

　　在JavaScript中，函数的参数都是可选的，而在TypeScript中的参数默认都是必传的。如果要让参数可选，那么需要在其后面跟一个问号（?），如下所示。
```typescript
function sum(x: number, y?: number): number {
  return x + y;
}
```
　　注意，可选参数得位于必选参数之后，下面的写法是错误的。
```typescript
function sum(x?: number, y: number): number {
  return x + y;
}
```

3）重载

　　JavaScript里的函数可根据不同数量和类型的参数返回不同类型的值，这样虽然很便捷，但是无法精确的传达出函数的输入和输出之间的对应关系。TypeScript提供了重载功能，可有效改善JavaScript函数定义不明确的问题。以重载定义多个caculate()函数为例，如下所示。
```typescript
function caculate(x: number, y: number): number;
function caculate(x: string, y: string): string;
function caculate(x, y): any {
  return x + y;
}
```
　　编译器会从重载列表中选出最先匹配的函数定义，并进行正确的类型检查。当多个函数定义之间是包含关系时，优先把最精确的定义放在最前面。

　　在调用改变后的caculate()函数时（如下所示），传递给它的实参，其类型和组合只要能与重载列表中的一个相同，就能编译成功，否则就会报错。
```typescript
caculate(1, 2);               //正确
caculate("1", "2");           //正确
caculate(false, true);        //错误
caculate("1", 2);             //错误
```

4）this参数

　　TypeScript能在定义函数时，显式地声明一个this参数，指定this的类型（即限制其指向），从而避免错误的使用this。例如为this定义为void类型，那么在函数中一旦使用this，就无法编译成功，如下所示。
```typescript
function func(this: void) {
  this.name = "strick";        //错误
}
```
　　注意，this是个假参数，位于参数列表的最前面，只用来做静态检查，不会出现在编译后的代码中。
