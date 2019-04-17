## 参考文章

1. [Flutter中文网](https://flutterchina.club/)
2. [flutter-io.cn](https://flutter-io.cn/)
3. [Flutter Github源码](https://github.com/flutter/flutter/tree/stable)
4. [Flutter原理与实践-美团](https://tech.meituan.com/waimai_flutter_practice.html)
5. [Stack Overflow来处理“HOWTO”类型的问题](https://stackoverflow.com/tags/flutter)
6. [Android各种场景如何在Flutter中处理](https://flutterchina.club/flutter-for-android/)
7. [阿里出品的各种widget演示](https://github.com/alibaba/flutter-go)
8. [flutter packages查找网站](https://pub.dartlang.org/flutter/packages/)

## 大厂们的解决方案
1. [美团Flutter原理与实践](https://tech.meituan.com/2018/08/09/waimai-flutter-practice.html)
2. [闲鱼技术](https://www.yuque.com/xytech/flutter/)
3. [头条 Flutter 混合工程实践](https://mp.weixin.qq.com/s/wdbVVzZJFseX2GmEbuAdfA)


## 注意点

1. 进入flutter目录，执行flutter doctor命令找不到命令错误，需要先设置全局环境变量才行。
2. Dart是单线程执行模型，支持Isolates（在另一个线程上运行Dart代码的方式）、事件循环和异步编程。 除非您启动一个Isolate，否则您的Dart代码将在主UI线程中运行，并由事件循环驱动（译者语：和JavaScript一样）。
3. Dart SDK已经在捆绑在Flutter里了，没有必要单独安装Dart。 flutter/bin/cache/dart-sdk
4. 第一次运行一个flutter命令（如flutter doctor）时，它会下载它自己的依赖项并自行编译。以后再运行就会快得多。
5. 从android studio打开已存在的flutter项目，右击“android"文件夹，flutter菜单不可以用时，可能是android文件夹下缺少”项目名称_android.iml“文件，如fluttertouchstone_android.iml，从其他项目copy一个来重命名下就行。
6. 什么flutter中的android的build产物不再android目录下，而在flutter项目根目录下，是因为在android项目的build.gradle中设置了  
```
rootProject.buildDir = '../build'
subprojects {
    project.buildDir = "${rootProject.buildDir}/${project.name}"
}
```
7. 如果在pubspec.yaml中引入的package是一个插件，会自动在项目根目录下生成.flutter-plugins文件并把插件项目注册进去，settings.gradle脚本会读取.flutter-plugins中的项目并include进来，其他module如果需要依赖添加如下代码即可  
```
  dependencies {
          // 这里的xxx是.flutter-plugins中注册的名字
        compileOnly rootProject.findProject(":xxx")
    }
```
8. 运行时反射，这在Flutter中是禁用的。运行时反射会干扰Dart的_tree shaking_。

## 安装
怎样使用IDE看这里：https://flutterchina.club/using-ide/

1. 去github上clone稳定版本flutter
2. 设置全局环境变量到fluter/bin目录
```
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
export PATH=$PATH:~/android/flutter/bin
```
3. 运行 flutter doctor 命令，该命令检查您的环境并在终端窗口中显示报告，如是否安装Android SDK等
4. 运行 flutter devices 命令以验证Flutter识别您连接的Android设备
5. flutter create myapp命令创建一个最简单的flutter程序。要使用Kotlin或Swift，请使用-i和/或-a标志:flutter create -i swift -a kotlin myapp
6. 在项目根目录下运行 flutter run 命令来运行应用程序
7. 用android studio开发需要安装Flutter插件和Dart插件，File-New Flutter Project或者File Open打开已存在的flutter工程
8. 在flutter工程中右击android目录-flutter-open android module in android studio可以在新的窗口中打开android部分代码

## 常用flutter命令
通过flutter help可以查看有哪些命令
1. flutter channel 查看flutter当前使用的分支master还是stable
2. 要切换分支，请使用flutter channel beta 或 flutter channel master
3. [升级 Flutter相关命令](https://flutterchina.club/upgrading/)
4. flutter devices，查看Flutter识别您连接的Android设备
5. flutter install 安装release包

## flutter框架开发语言
Google 把 Flutter 作为 Fuchsia 的用户界面，Dart 作为主要的编程语言，从颜色和展示效果上看，使用的是 Material Design UI 理念。
![dart语言比较](../images/dart_lang_compare.png)

## [Flutter Widget框架概述](https://flutterchina.club/widgets-intro/)
1. Flutter Widget采用现代响应式框架构建，这是从 React 中获得的灵感，中心思想是用widget构建你的UI。
2. 框架强制根widget覆盖整个屏幕。
3. Flutter框架将依次构建这些widget，直到构建到最底层的子widget时，这些最低层的widget通常为RenderObject，它会计算并描述widget的几何形状。
4. 无状态的widget，它将通过构造函数从父widget中接收到的值存储在**final**成员变量中，然后在build函数中使用它们。
5. 如果要添加填充，边距，边框或背景色，请使用Container等容器类型的widget来设置（译者语：只有容器有这些属性）
6. 在Flutter中，一个自定义widget通常是通过组合其它widget来实现的，而不是继承。

## flutter零散知识点
1. 当你在添加一个包后首次运行（IntelliJ中的’Packages Get’）flutter packages get，Flutter将找到包的版本保存在pubspec.lock。

## [写UI的思路](https://flutterchina.club/tutorials/layout/)
1. 整理页面需要哪些widget以及他们的嵌套关系
2. 分析哪些widget需要把逻辑封装起来，widget一般不会继承，都是继承StatelessWidget或者StatefulWidget来组装各种基础widget。
3. 需要封装起来widget是有状态的还是无状态的，一般来说外层的widget保持状态，内部的widget不需要保持状态，需要的数据通过构造函数传进来，然后再通过事件告诉父widget，原则是事件流是“向上”传递的，而状态流是“向下”传递的。
4. 状态数据一般是保存在widget对应的State类中的变量上。
5. 最后是具体些widget的时候了，怎么写查文档或者google。


## flutter插件
在Flutter中，依赖包由[Pub](https://pub.dartlang.org/)仓库管理，项目依赖配置在项目根目录下pubspec.yaml文件中声明即可（类似于NPM的版本声明 Pub Versioning Philosophy），对于未发布在Pub仓库的插件可以使用git仓库地址或文件路径
```dependencies:
url_launcher: ">=0.1.2 <0.2.0"
collection: "^0.1.2"
plugin1:
    git:
    url: "git://github.com/flutter/plugin1.git"
plugin2:
    path: ../plugin2/
```

## 一切皆控件
在Flutter中，所有功能都可以通过组合多个Widget来实现，包括对齐方式、按行排列、按列排列、网格排列甚至事件处理等等。在Flutter中“一切皆是控件”，通过组合、嵌套不同类型的控件，就可以构建出任意功能、任意复杂度的界面


## [Android发布版本](https://flutterchina.club/android-release/)
默认情况下，flutter run命令会使用调试版本配置。
编译release包: flutter build apk
安装：flutter install
注意：这三个命令都要在项目根目录下执行

## Widget不可变特性理解
在Flutter中Widget是不可变的，不会直接更新，而必须使用Widget的状态才能更新。所以Widget仅支持一帧，并且在每一帧上，Flutter的框架都会创建一个Widget实例树(译者语：相当于一次性绘制整个界面)。
这里要注意的重要一点是无状态和有状态widget的核心特性是相同的，每一帧它们都会重新构建，不同之处在于StatefulWidget有一个State对象，它可以跨帧存储状态数据并恢复它。
所以Flutter中不存在添加或删除组件，您可以传入一个函数，该函数返回一个widget给父项，并通过布尔值控制该widget的创建，可以理解成重新动态创建。
```
body: new Center(
    child: _getToggleChild(),
)

_getToggleChild() {
    if (toggle) {
        return new Text('Toggle One');
    } else {
    return new MaterialButton(onPressed: () {}, child: new Text('Toggle Two'));
    }
}

```

## 扩展和自定义widget
Flutter拥抱组合：widget由较小的widget构建而成，您可以以各种方式组合，以制作自定义widget。例如，RaisedButton将一个Material widget与一个GestureDetector widget组合在一起，而不是对一个通用按钮widget进行子类化。Material widget提供了可视化设计，而GestureDetector widget提供了交互设计。

## “热重载”与“完全重新启动”有何不同？
Hot Reload通过将更新的源代码文件注入正在运行的Dart VM（虚拟机）中工作。这不仅包括添加新类，还包括向现有类添加方法和字段，以及更改现有函数。尽管有几种类型的代码更改无法热重新：
1. 全局变量初始化器。
2. 静态字段初始化程序。
3. main()应用程序的方法。

## 我可以在Flutter应用程序的后台运行Dart代码吗？
由于在Android和iOS平台上支持后台执行的基本差异，在后台运行代码具有特定于平台的API。
在Android上，android_alarm_manager 即使您的Flutter应用程序不在前台，该插件也可让您在后台运行Dart代码。
在iOS上，我们目前不支持此功能。请留意Bug 6192的更新。
## 动态更新ListView
1. [ListView的builder工厂构造函数允许您按需建立一个懒加载的列表视图，不需要调用setState方法也可以更新列表](https://flutterchina.club/get-started/codelab/)

```
Widget _buildSuggestions() {
    return new ListView.builder(
        // 该方法至少执行一次，哪怕列表是空的
        itemBuilder: (context, i) {
            final index = i ~/ 2;
            // 如果是建议列表中最后一个单词对
            if (index >= _suggestions.length) {
            // ...接着再生成10个单词对，然后添加到建议列表，不需要调用setState方法
            _suggestions.addAll(generateWordPairs().take(10));
        }
        return _buildRow(_suggestions[index]);
    }
    );
}

```
2. [不需要懒加载，直接new ListView，需要调用setState更新状态才能刷新列表](https://flutterchina.club/flutter-for-android/)

```
Widget getRow(int i) {
    return new GestureDetector(
        child: new Padding(
        padding: new EdgeInsets.all(10.0),
        child: new Text("Row $i")),
            onTap: () {
                // 必须调用setState
                setState(() {
                widgets = new List.from(widgets);
                widgets.add(getRow(widgets.length + 1));
                print('row $i');
            });
        },
    );
}
```

## [事件流是“向上”传递的，而状态流是“向下”传递的](https://flutterchina.club/widgets-intro/)
在Flutter中，事件流是“向上”传递的，而状态流是“向下”传递的（译者语：这类似于React/Vue中父子组件通信的方式：子widget到父widget是通过事件通信，而父到子是通过状态），重定向这一流程的共同父元素是State。更形象的理解是，父元素State是一个桥梁，子widget出发事件->State中的变量->更新其他Widget，这样可以达到*责任分离允许将复杂性逻辑封装在各个widget中，同时保持父项的简单性。*

## Scaffold
Scaffold 是 Material library 中提供的一个widget, 它提供了默认的导航栏、标题和包含主屏幕widget树的body属性

## 路由
和Android相似，您可以在AndroidManifest.xml中声明您的Activities，在Flutter中，您可以将具有指定Route的Map传递到顶层MaterialApp实例

```
void main() {
  runApp(new MaterialApp(
    home: new MyAppHome(), // becomes the route named '/'
    routes: <String, WidgetBuilder> {
      '/a': (BuildContext context) => new MyPage(title: 'page A'),
      '/b': (BuildContext context) => new MyPage(title: 'page B'),
      '/c': (BuildContext context) => new MyPage(title: 'page C'),
    },
  ));
}
```
然后，您可以通过Navigator来切换到命名路由的页面。
```
Navigator.of(context).pushNamed('/b');
```

## 写flutter 插件 https://flutterchina.club/developing-packages/
插件工程结构和普通的flutter工程结构一样，也可以直接运行，只是多了个example目录，example目录就是一个引用了插件的普通工程，实际上运行的就是这个工程。

注意点：
1. 在Android Studio中编辑Android平台代码之前，首先确保代码至少已经构建过一次（例如，从IntelliJ运行示例应用程序或在终端执行cd hello/example; flutter build apk）。
2. 由于android studio打开的插件工程是flutter工程，不是android工程，所以android/ios目录下的代码无法识别，显示是错误的。所以编辑插件中平台部分的代码需要用android studio重新打开（右击项目根目录 Flutter-open android module in android studio）,实际上打开的是example/android目录。
3. 插件或者package可以像普通的flutter工程一样，在pubspec.yam中添加对其他插件或者package的依赖。如果插件需要调用依赖的插件的特定于平台的API，那么需要在插件的特定平台的build.gradle中添加依赖  
```
android {
    // url_launcher是依赖的插件的名称
    dependencies {
        provided rootProject.findProject(":url_launcher")
    }
}
```
4. 如果项目依赖了插件，执行flutter get packages命令后，插件的本地代码会自动作为library注册到.flutter-plugins文件中，并且插件入口类会自动注册到io.flutter.plugins.GeneratedPluginRegistrant.java文件中。

添加文档
建议将以下文档添加到所有软件包：
1. README.md:介绍包的文件
2. CHANGELOG.md 记录每个版本中的更改
3. LICENSE 包含软件包许可条款的文件
4. 所有公共API的API文档 (详情见下文)
在发布软件包时，API文档会自动生成并发布到dartdocs.org。如果您希望在本地生成API文档，进入插件项目根目录，请使用以下命令：dartdoc


## flutter module
创建方式有两种：
1. 按照官网用命令创建：https://github.com/flutter/flutter/wiki/Add-Flutter-to-existing-apps
2. 用android studio -> New Flutter Project -> 选择flutter module

flutter module和普通flutter工程目录结构相同，也可以直接用android studio打开并且运行(.android目录也可以直接用android studio打开)。区别：
1. android和ios目录变成隐藏的了
2. android目中的app拆分成app和Flutter两个module

注意点：
1. .android目录下的app不能删除，不然执行./gradlew assembleRelease或者./gradlew assembleDebug命令生成的aar中没有assets。
2. ./gradlew flutter:assembleDebug命令生成的aar中没有assets，需要去掉flutter。

## flutter.gradle文件
Flutter工程的Android打包，其实只是在Android的Gradle任务中插入了一个flutter.gradle的任务，而这个flutter.gradle主要做了三件事：（这个文件可以在Flutter库中的[flutter/packages/flutter_tools/gradle]目录下能找到。） 
1. 增加flutter.jar的依赖。 
2. 插入Flutter Plugin的编译依赖。 
3. 插入Flutter工程的编译任务，最终将产物（两个isolaate_snapshot文件、两个vm_snapshot文件和flutter_assets文件夹）拷贝到mergeAssets.outputDir，最终merge到APK的assets目录下。

## dart依赖包
1. 依赖包在pubspec.yaml中管理
2. 下载的包存储在~/.pub-cache/hosted/pub.flutter-io.cn/english\_words-3.1.5/lib/
3. 每个工程通过根目录下的.packages文件映射，如
   english\_words:file:///Users/bruce/.pub-cache/hosted/pub.flutter-io.cn/english\_words-3.1.5/lib/
4. 在pubspec.yaml文件中写好依赖后，通过flutter packages get命令安装，或者IDE上的工具安装。
5. 依赖版本号写成any，表示可以使用任何版本的包。
6. 依赖git仓库时，可以使用ref参数将依赖关系固定到特定的git commit，branch或tag。

## [Flutter Engine线程管理与Dart Isolate机制](https://www.yuque.com/xytech/flutter/kwoww1)
一个进程里面最多只会初始化一个Dart VM，然而一个进程可以有多个Flutter Engine，
多个Engine实例共享同一个Dart VM，只是不同Engine实例加载的代码跑在各自独立的Isolate。
Flutter Engine自己不创建管理线程，Flutter Engine线程的创建和管理是由embedder负责的。
Mobile平台上面每一个Engine实例启动的时候会为UI，GPU，IO Runner各自创建一个新的线程。所有Engine实例共享同一个Platform Runner和线程。