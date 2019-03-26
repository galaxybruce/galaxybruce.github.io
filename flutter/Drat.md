## 参考文章

1. [Dart中文网](http://dart.goodev.org/)
2. [Dart](https://www.dartlang.org/)
3. [stagehand-Dart 项目生成器](https://github.com/dart-lang/stagehand)
4. [最佳实践](http://dart.goodev.org/guides/language/effective-dart/usage)

## 在线练习
[dartpad](https://dartpad.cn)

## 函数式编程

函数式编程就是一种抽象程度很高的编程范式，纯粹的函数式编程语言编写的函数没有变量，因此，任意一个函数，只要输入是确定的，输出就是确定的，这种纯函数我们称之为没有副作用。而允许使用变量的程序设计语言，由于函数内部的变量状态不确定，同样的输入，可能得到不同的输出，因此，这种函数是有副作用的。

函数式编程的一个特点就是，允许把函数本身作为参数传入另一个函数，还允许返回一个函数！,如：

```
var print = function(i){ console.log(i);};
[1,2,3].forEach(print);
```

## dart依赖包

1. 依赖包在pubspec.yaml中管理
2. 下载的包存储在~/.pub-cache/hosted/pub.flutter-io.cn/english\_words-3.1.5/lib/
3. 每个工程通过根目录下的.packages文件映射，如
   english\_words:file:///Users/bruce/.pub-cache/hosted/pub.flutter-io.cn/english\_words-3.1.5/lib/
4. 在pubspec.yaml文件中写好依赖后，通过pub get命令安装，或者IDE上的工具安装

## [哪些不需要提交到代码库的文件](http://dart.goodev.org/guides/libraries/private-files)

## vscode IDE 调试

1. vscode调试也是用dart插件
2. 调试入口在左侧面板上有，启动调试时，会自动在.vscode目录下创建launch.json文件，需要调试哪个文件都要在这里面配置
   ```
   {
    "name": "hello",        //调试任务名称
    "program": "hello.dart", //待调试文件路径
    "request": "launch",    //固定的 
    "type": "dart"          // 固定的
   }
   ```
3. 从“调试-启动调试“菜单调试的是左侧面板中当前选中的调试任务
4. 编辑窗口右上角的Run Code按钮和Start Without Debuging的区别貌似只是Run Code的的输出在”OUTPUT"tab上，Start Without Debuging的输出在”DEBUG CONSOLE“tab上

## 注意点

1. 所有能够使用变量引用的都是对象，如int a;那a的默认值为 null。
2. 虽然可以用var不指定类型声明变量的方式，建议指明类型可以更加清晰的表达你的意图。 IDE 编译器等工具有可以使用类型来更好的帮助你， 可以提供代码补全、提前发现 bug 等功能。
3. stagehandle需要设置环境变量：export PATH="$PATH":"$HOME/.pub-cache/bin"

## 重要概念
1. 和 Java 不同的是，Dart 没有 public、 protected、 和 private 关键字。如果一个标识符以 (_) 开头，则该标识符 在库内是私有的。
2. 没有指定类型的变量的类型为 dynamic。
3. Dart支持顶级方法和变量，就是在类外部定义的方法（如main())和变量

## 语法

### 变量

1. const和final都是不能修改的变量，const变量同时也是final变量。顶级的 final 变量或者类中的 final 变量在 第一次使用的时候初始化，const是编译时常量。
2. const 关键字不仅仅只用来定义常量。 有可以用来创建不变的值
   ```
   var foo = const [];   // foo is currently an EIA.
   final bar = const []; // bar will always be an EIA.
   const baz = const []; // baz is a compile-time constant EIA.
   ```
3. Dart 字符串是 UTF-16 编码的字符序列。 可以使用单引号或者双引号来创建字符串。可以在字符串中使用表达式，用法是这样的： ${expression}。
4. == 操作符判断两个对象的内容是否一样。 如果两个字符串包含一样的字符编码序列， 则他们是相等的。
5. 在 Dart 中数组就是 List 对象。
   ```
   List<int> list = const [1, 2, 3],
   ```

### 方法

1. 可选命名参数需用大括号括起来，使用 {param1, param2, …} 的形式来指定命名参数  
 ```
   enableFlags({bool bold, bool hidden}) {
   // ...
   }
   // 可选参数用=号或者:号来设置默认值，没设置就是null
   void enableFlags({bool bold = false, bool hidden = false}) {
   // ...
   }

   // 调用方式
   enableFlags(bold: true, hidden: false);
   // 或者不指定参数
   enableFlags();
   ```

2. =&gt;函数，这个 =&gt; expr 语法是 { return expr; } 形式的缩写。
3. 可以创建没有名字的方法，称之为 匿名方法，有时候也被称为 lambda 或者 closure 闭包。 
4. Dart 是一个真正的面向对象语言，方法也是对象并且具有一种 类型， Function。
5. 方法定义时，可以没有返回类型。
6. 所有的函数都返回一个值。如果没有指定返回值，则 默认把语句 return null; 作为函数的最后一个语句执行。



### 闭包

一个 闭包 是一个方法对象Function，不管该对象在何处被调用， 该对象都可以访问其作用域内 的变量。

```
Function makeAdder(num addBy) {
  return (num i) => addBy + i;
}

main() {
  // Create a function that adds 2.
  // 不管你在那里执行 makeAdder() 所返回的函数， 都可以使用 addBy 参数。
  var add2 = makeAdder(2);
  assert(add2(3) == 5);
}
```

### 操作符

1. as操作符：使用 as 操作符把对象转换为特定的类型
2. ??= 操作符用来指定 值为 null 的变量的值
   ```
   b ??= value; // 如果 b 是 null，则赋值给 b；
   ```
3. expr1 ?? expr2 &ensp;&ensp;// 如果 expr1 是 non-null，返回其值； 否则执行 expr2 并返回其结果。
4. ?.&ensp;&ensp;// 例如 foo?.bar 如果 foo 为 null 则返回 null，否则返回 bar 成员
5. 要测试两个对象代表的是否为同样的内容，使用 == 操作符。\(在某些情况下，你需要知道两个对象是否是同一个对象， 使用 identical\(\) 方法。\) 
   ```
   // 两个一样的编译时常量其实是 同一个对象：
   var a = const ImmutablePoint(1, 1);
   var b = const ImmutablePoint(1, 1);
   assert(identical(a, b)); // They are the same instance!
   ```
6. is相当于java中的instance，只有当 obj 实现了 T 的接口， obj is T 才是 true。例如 obj is Object 总是 true。
7. 级联操作符 (..) 可以在同一个对象上 连续调用多个函数以及访问成员变量。 使用级联操作符可以避免创建 临时变量， 并且写出来的代码看起来 更加流畅.
8. 'a' * length 这样的代码执行效果是将字符 'a' 重复 length 次，如'a' * 3输出aaa

### 流程控制语句

1. 和java相似
2. where可以用来过滤集合中的某些数据

   ```
   for (int i = 0; i < candidates.length; i++) {
      var candidate = candidates[i];
      if (candidate.yearsExperience < 5) {
         continue;
      }
      candidate.interview();
   }
   // 可以这样写
   candidates.where((c) => c.yearsExperience >= 5)
   .forEach((c) => c.interview());
   ```

3. switch case中可以设置标签，用continue跳转到指定的标签处

   ```
   var command = 'CLOSED';
   switch (command) {
      case 'CLOSED':
       executeClosed();
       continue nowClosed;
       // Continues executing at the nowClosed label.
   nowClosed:
      case 'NOW_CLOSED':
       // Runs for both CLOSED and NOW_CLOSED.
       executeNowClosed();
       break;
   }
   ```

4. assert断言只在检查模式下运行有效，如果在生产模式 运行，则断言不会执行。
5. Exception
   可以使用on 或者 catch 来声明捕获语句，也可以 同时使用。使用 on 来指定异常类型，使用 catch 来 捕获异常对象。使用 rethrow 关键字可以 把捕获的异常给 重新抛出。
   ```
   try {
      breedMoreLlamas();
   } on OutOfLlamasException {
      // A specific exception
      buyMoreLlamas();
   } on Exception catch (e) {
      // Anything else that is an exception
      print('Unknown exception: $e');
   } catch (e) {
      // No specified type, handles all
      print('Something really unknown: $e');
      rethrow; // Allow callers to see the exception.
   }
   ```

### 类

1. 每个实例变量都会自动生成一个 getter 方法（隐含的）,Non-final 实例变量还会自动生成一个 setter 方法
2. 构造函数赋值简化操作

   ```
   class Point {
      num x;

      // Syntactic sugar for setting x and y
      // before the constructor body runs.
      Point(this.x);
   }
   ```

3. 命名构造函数:使用命名构造函数可以为一个类实现多个构造函数，或者使用命名构造函数来更清晰的表明你的意图：

   ```
   class Point {
      num x;
      num y;

      Point(this.x, this.y);
      // Named constructor
      Point.fromJson(Map json) {
       x = json['x'];
       y = json['y'];
      }
   }
   ```

4. 在构造函数参数后使用冒号 \(:\) 可以调用 超类构造函数
   ```
   class Employee extends Person {
      // Person does not have a default constructor;
      // you must call super.fromJson(data).
      Employee.fromJson(Map data) : super.fromJson(data) {
       print('in Employee');
      }
   }
   // 由于超类构造函数的参数在构造函数执行之前执行，所以 参数可以是一个表达式或者 一个方法调用(_只能是静态方法，不能是成员方法_)：
   class Employee extends Person {
      // ...
       Employee() : super.fromJson(findDefaultData());
   }
   ```
5. 冒号 \(:\)在构造函数体执行之前除了可以调用超类构造函数之外，还可以 初始化实例参数。

   ```
   Point.fromJson(Map jsonMap)
      : x = jsonMap['x'],
        y = jsonMap['y'] {
    print('In Point.fromJson(): ($x, $y)');
   }

   Point(x, y)
      : x = x,
        y = y,
        distanceFromOrigin = sqrt(x * x + y * y);
   ```

6. 如果一个构造函数并不总是返回一个新的对象，则使用 factory 来定义 这个构造函数。类似单例。
7. 重定向构造函数，就是构造函数调动类中的其他构造函数，在构造函数声明后，使用 冒号调用其他构造函数。
8. 使用 abstract 修饰符定义一个 抽象类—一个不能被实例化的类。 抽象类通常用来定义接口， 以及部分实现。
9. 通过operator关键字在类中定义方法可覆写的操作符
10. 使用 get 和 set 关键字定义 getter 和 setter，来创建新的属性  

```
class Rectangle {
   num left;
   num top;
   num width;
   num height;

   Rectangle(this.left, this.top, this.width, this.height);

   // Define two calculated properties: right and bottom.
   num get right => left + width;
   set right(num value) => left = value - width;
   num get bottom => top + height;
   set bottom(num value) => top = value - height;
}

```     
11. Dart 并不支持构造函数的重载，用没有方法体的可选命名参数的构造方法来替代 Java 代码中的重载构造方法。  

```
class Rectangle {
  int width;
  int height;
  Point origin;

  Rectangle({this.origin = const Point(0, 0), this.width = 0, this.height = 0});

  @override
  String toString() =>
      'Origin: (${origin.x}, ${origin.y}), width: $width, height: $height';
}

main() {
  print(Rectangle(origin: const Point(10, 20), width: 100, height: 200));
  print(Rectangle(origin: const Point(10, 10)));
  print(Rectangle(width: 200));
  print(Rectangle());
}
```

### 常量构造函数
有些类提供了常量构造函数。使用常量构造函数 可以创建编译时常量，要使用常量构造函数只需要用 const 替代 new 即可：  
```
var p = const ImmutablePoint(2, 2);
```
两个一样的编译时常量其实是 同一个对象：
```
var a = const ImmutablePoint(1, 1);
var b = const ImmutablePoint(1, 1);
assert(identical(a, b)); // They are the same instance!
```
### 隐式接口

Dart是没有interface这种东西的，但并不以为着这门语言没有接口，事实上，Dart任何一个类都是接口，你可以实现任何一个类，只需要重写那个类里面的所有具体方法。每个类都隐式的定义了一个包含所有实例成员的接口， 并且这个类实现了这个接口。如果你想 创建类 A 来支持 类 B 的 api，而不想继承 B 的实现， 则类 A 应该实现 B 的接口。就是说任何class可以被当做interface被其他用implement关键字实现，与extends关键字不同的是，实现的方法中不能调用super。

### 使用库

1. 对于内置的库，URI 使用特殊的 dart: scheme; 对于其他的库，你可以使用文件系统路径或者 package: scheme
   ```
   import 'dart:io';
   import 'package:mylib/mylib.dart';
   import 'package:utils/utils.dart' as utils;
   ```
2. 导入库的一部分 用关键字show或者hide
   ```
   // Import only foo.
   import 'package:lib1/lib1.dart' show foo;

   // Import all names EXCEPT foo.
   import 'package:lib2/lib2.dart' hide foo;
   ```
3. 延迟载入库，需要先使用 deferred as 来 导入

### 可调用的类
如果 Dart 类实现了 call\(\) 函数则 可以把该类当做方法来调用。

### Mixin
[Flutter基础：理解Dart的Mixin继承机制](https://kevinwu.cn/p/ae2ce64/#%E5%9C%BA%E6%99%AF)  
mixins是一个强大的概念，允许您跨多个类层次结构重用代码。当我们想要在不共享相同类层次结构的多个类之间共享行为时，或者在超类中实现此类行为没有意义时，Mixins非常有用。  
如果我们不想让我们创建的mixin被实例化或扩展，同时使用factory关键字结合_权限符
```
abstract class Walker {
  // This class is intended to be used as a mixin, and should not be
  // extended directly.
  factory Walker._() => null;

  void walk() {
    print("I'm walking");
  }
}
```

## 异步支持
1. 用async 异步方法和 await 表达式实现异步编程
2. 如果 await 无法正常使用，请确保是在一个 async 方法中。
3. 在 await expression 中， expression 的返回值通常是一个 Future； 如果返回的值不是 Future，则 Dart 会自动把该值放到 Future 中返回。await expression 会阻塞住，直到需要的对象返回为止。

## Generators
当您需要延迟地生成一个值序列时，请考虑使用生成器函数。Dart内置支持两种生成器函数:
同步生成器：返回Iterable对象
异步生成器：返回Stream对象
要实现同步生成器函数，将函数体标记为sync*，并使用yield语句传递值;
要实现异步生成器函数，将函数体标记为async*，并使用yield语句传递值;
如果您的生成器是递归的，您可以使用yield*来改进它的性能:  
```
Iterable<int> naturalsDownFrom(int n) sync* {
  if (n > 0) {
    yield n;
    yield* naturalsDownFrom(n - 1);
  }
}
```



## Isolates
所有的 Dart 代码在 isolates 中运行而不是线程。 每个 isolate 都有自己的堆内存，并且确保每个 isolate 的状态都不能被其他 isolate 访问。

## TypeDef
在 Dart 语言中，方法也是对象。 使用 typedef, 或者 function-type alias 来为方法类型命名， 然后可以使用命名的方法。

## Metadata
定义了一个带有两个参数的 @todo 注解  
```
library todo;

class todo {
  final String who;
  final String what;

  const todo(this.who, this.what);
}
```

