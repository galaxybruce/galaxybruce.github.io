## 参考文章

1. [Dart中文网](http://dart.goodev.org/)
2. [Dart](https://www.dartlang.org/)
3. [stagehand-dart project generator](https://github.com/dart-lang/stagehand)

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

3. 命名构造函数:使用命名构造函数可以为一个类实现多个构造函数， 或者使用命名构造函数来更清晰的表明你的意图：

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
   // 由于超类构造函数的参数在构造函数执行之前执行，所以 参数可以是一个表达式或者 一个方法调用：
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

### 隐式接口

每个类都隐式的定义了一个包含所有实例成员的接口， 并且这个类实现了这个接口。如果你想 创建类 A 来支持 类 B 的 api，而不想继承 B 的实现， 则类 A 应该实现 B 的接口。  
就是说任何class可以被当做interface被其他用implement关键字实现，与extends关键字不同的是，实现的方法中不能调用super。

### 使用库

1. 对于内置的库，URI 使用特殊的 dart: scheme; 对于其他的库，你可以使用文件系统路径或者 package: scheme
   ```
   import 'dart:io';
   import 'package:mylib/mylib.dart';
   import 'package:utils/utils.dart' as utils;
   ```
2. 导入库的一部分
3. 延迟载入库

### 可调用的类

如果 Dart 类实现了 call\(\) 函数则 可以把该类当做方法来调用。

