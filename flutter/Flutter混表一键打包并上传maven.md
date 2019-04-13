## 概述
flutter工程目前有四种形式：
1. 普通的flutter工程，也就是纯flutter项目
2. flutter package，一个纯dart的组件
3. flutter plugin，一种专用的Dart包，其中包含用Dart代码编写的API，以及针对Android（使用Java或Kotlin）和/或针对iOS（使用ObjC或Swift）平台的特定实现
4. flutter module，一个可用于嵌入现有android或者ios项目的flutter组件

这篇文章的混编用的就是第四种，官方的[flutter module](https://github.com/flutter/flutter/wiki/Add-Flutter-to-existing-apps)，官方的flutter module已经很好的能把flutter代码嵌入到现有的android和ios项目。但是缺陷也很明显，就是只能源码引用，也就是说一个项目中的其他不开发flutter的研发人员也要安装flutter环境。如果是小项目影响也不是太大，如果是一个有一定规模的项目呢，那么会严重影响团队之间的写作，本来用来提高开发效率的神器却变成了累赘。那今天我和大家一起分享下我的解决方案，如果不对的对方欢迎拍砖。

## 需要解决的问题
为了不影响其他开发人员，最好的办法就是把flutter代码打成aar并上传到maven供他小伙伴使用，需要解决那些问题呢？  
1. 如何把flutter打成aar?
2. 如果flutter module依赖了我们写的flutter plugin，我们的flutter plugin又依赖了第三方的flutter plugin，这种级联依赖如何打成aar?
3. 这些aar上传到maven，他们的彼此依赖的maven版本号怎么解决？
4. 即使上面三个问题都解决了，用起来还是很繁琐，有没有只要执行一个命令就能把所有的流程串起来？
我们用shell脚本把gradle命令串起来就能达到自动化的效果，也就解决了第四个问题。下面我来和盆友们一步一步来分享我的处理方法。

### 一、如何把flutter打成aar?

要把flutter打成aar，我们得知道flutter module依赖了哪些插件，包括我们自己写的以及第三方的。只要在pubspec.ymal中配置了插件依赖，执行flutter packages get命令就会flutter module根目录下生成一个.flutter-plugins文件，里面按照key-value的形式记录所有依赖的插件（包括级联依赖的），key是插件的名称，value是插件工程所在的目录，这里要注意的是第三方的插件也是先下载的本地（mac上的目录是~/.pub-cache/hosted/pub.flutter-io.cn/）再引用该地址。

```
// 自己开发的插件
helloplugin=/Users/bruce/work/kidswant/sourcecode/Projects_Android/git/flutter/helloplugin/
// 第三方的插件
url_launcher=/Users/bruce/.pub-cache/hosted/pub.flutter-io.cn/url_launcher-5.0.2/
```
flutter也是通过在编译阶段读取该文件动态在settings.gradle中添加include这些插件工程，flutter普通工程和flutter module中都有一段这个脚本（flutter module中的脚本在.android/include_flutter.groovy）

```
gradle.include ':flutter'
gradle.project(':flutter').projectDir = new File(flutterProjectRoot, '.android/Flutter')

def plugins = new Properties()
def pluginsFile = new File(flutterProjectRoot, '.flutter-plugins')
if (pluginsFile.exists()) {
    pluginsFile.withReader('UTF-8') { reader -> plugins.load(reader) }
}

plugins.each { name, path ->
    def pluginDirectory = flutterProjectRoot.toPath().resolve(path).resolve('android').toFile()
    gradle.include ":$name"
    gradle.project(":$name").projectDir = pluginDirectory
}
```
还有一个重要的脚本/android/flutter/packages/flutter_tools/gradle/flutter.gradle，









