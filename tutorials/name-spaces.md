# 命名空间

TypeScript中的命名空间可将那些具有内在联系的接口、类或对象等代码组织在一起，既能隔离作用域，也能避免命名冲突，并且使得代码结构清晰，更易追踪。在命名空间内部，所有实体部分默认都是私有的，需要由export关键字导出之后，才能在外部访问，如下所示。
```typescript
namespace Util {
  export function log(msg) {
    console.log(msg);
  }
}
Util.log("strick");
```
TypeScript会将上面的命名空间编译成两部分：Util变量和即时函数，如下代码所示，这是一种常见的模块化封装。
```typescript
var Util;
(function(Util) {
  function log(msg) {
    console.log(msg);
  }
  Util.log = log;
})(Util || (Util = {}));
Util.log("strick");
```
注意，由于TypeScript 1.5为了与ES6的术语保持一致，已将内部模块改为命名空间，外部模块简称为模块，因此原先用于内部模块的module关键字和现在的namespace关键字在功能上是一样的。但是为了避免被ES6、CommonJS、UMD等模块概念中的相似名称所迷惑，官方推荐使用namespace。

## 分离

当命名空间膨胀的过大时，为了便于维护，有必要将其分离到各个文件中。例如将命名空间分到三个文件中，第一个util.ts文件的代码如下。
```typescript
namespace Util {
  export function log(msg) {
    console.log(msg);
  }
}
```
第二个validator.ts文件的代码如下。
```typescript
namespace Validator {
  export function isAcceptable(str) {
    return str.length > 1;
  }
}
```
第三个default.ts文件的代码如下所示，其中三斜线指令用来通知编译器在编译时需要引入的外部文件，并且它必须声明在文件顶部。
```typescript
/// <reference path="util.ts" />
/// <reference path="validator.ts" />
let str = "abc";
if(Validator.isAcceptable(str)) {
  Util.log("success");
}
```
当需要处理多个包含命名空间的文件时，有两种方式可确保编译后的代码按正确顺序加载。第一种是在输入编译命令时添加--outFile参数，如下所示，其后会跟一个输出的脚本文件以及一个或多个要编译的文件。
```typescript
tsc --outFile default.js default.ts
```
第二种是将编译后的脚本通过HTML的<script>元素在页面中按正确的顺序引入，如下所示。
```typescript
<script src="util.js"></script>
<script src="validator.js"></script>
<script src="default.js"></script>
```

## 别名
　　命名空间支持嵌套，即在内部声明另一个命名空间，如下所示。
```typescript
namespace Shape {
  export namespace Polygon {
    export class Triangle { }
    export class Square { }
  }
}
```
当引用这类命名空间时，可以通过import关键字为其取一个短的别名，如下代码所示。注意，不要与加载模块的import语法相混淆，此处import的右侧必须包含命名空间。
```typescript
import P = Shape.Polygon;
import Triangle = Shape.Polygon.Triangle;
let sq = new P.Square();
let triangle = new Triangle();
```
通过编译后的代码可以发现，由于import是var的语法糖（如下代码所示），因此改变P或Triangle变量的值不会影响命名空间的引用。
```typescript
var P = Shape.Polygon;
var Triangle = Shape.Polygon.Triangle;
var sq = new P.Square();
var triangle = new Triangle();
```
