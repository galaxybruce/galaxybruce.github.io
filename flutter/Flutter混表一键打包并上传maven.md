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

### 一、有哪些代码要打成aar？

要把flutter module打成aar，我们得知道flutter module依赖了哪些插件，包括我们自己写的以及第三方的。只要在pubspec.ymal中配置了插件依赖，执行flutter packages get命令就会flutter module根目录下生成一个.flutter-plugins文件，里面按照key-value的形式记录所有依赖的插件（包括级联依赖的），key是插件的名称，value是插件工程所在的目录，这里要注意的是第三方的插件也是先下载的本地（mac上的目录是~/.pub-cache/hosted/pub.flutter-io.cn/）再引用该地址。

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
这些工程是如何添加到flutter module依赖中的呢？还有一个重要的脚本/android/flutter/packages/flutter_tools/gradle/flutter.gradle，代码如下

```
File pluginsFile = new File(project.projectDir.parentFile.parentFile, '.flutter-plugins')
Properties plugins = readPropertiesIfExist(pluginsFile)

plugins.each { name, _ ->
    def pluginProject = project.rootProject.findProject(":$name")
    if (pluginProject != null) {
        project.dependencies {
            if (project.getConfigurations().findByName("implementation")) {
                implementation pluginProject
            } else {
                compile pluginProject
            }
        }
        pluginProject.afterEvaluate {
            pluginProject.android.buildTypes {
            profile {
                initWith debug
            }
        }
    }
    pluginProject.afterEvaluate this.&addFlutterJarCompileOnlyDependency
    } else {
        project.logger.error("Plugin project :$name not found. Please update settings.gradle.")
    }
}
```
这些插件都被添加到flutter module中，所以如果是自己的插件中依赖了第三方的插件，只要添加compileOnlye或者provide依赖就行

```
dependencies {
    compileOnly rootProject.findProject(":url_launcher")
}
```

### 二、把所有的依赖工程分别打成aar并上传到maven
我们也是通过读取.flutter-plugins文件来把所有的依赖都打成aar并上传到maven。
```
for line in $(cat .flutter-plugins)
do
plugin_name=${line%%=*}
plugin_path=${line##*=}
echo '==Build and publish plugin: ['${plugin_name}']='${plugin_path}

cd .android

# 在使用变量比较字符串之前，最好在判断之前加一个判断变量是否为空 或者使用双引号将其括起来，不然会出现异常[: ==: unary operator expected
if [ "$debugMode" == "debug" ]
then
echo "==start build debug aar for plugin ["${plugin_name}"]"
./gradlew :${plugin_name}:clean :${plugin_name}:assembleDebug -PfGroupId=${fGroupId} -PfArtifactId=${plugin_name} -PfVersion=${fVersion}
else
echo "==start build release aar for plugin ["${plugin_name}"]"
# uploadArchives命令自动回生成release模式的aar，不用执行该命令
./gradlew :${plugin_name}:clean :${plugin_name}:assembleRelease -PfGroupId=${fGroupId} -PfArtifactId=${plugin_name} -PfVersion=${fVersion}
fi

echo "==start upload maven for plugin ["${plugin_name}"]"
./gradlew :${plugin_name}:uploadArchives -PfGroupId=${fGroupId} -PfArtifactId=${plugin_name} -PfVersion=${fVersion}

echo -e "\n"
cd ../
done
```
上传到maven，那肯定是要指定每个maven的groupId:artifactId:version，大家可能注意到上面的代码中，每个./gradlew后面都有三个参数-PfGroupId=${fGroupId} -PfArtifactId=${plugin_name} -PfVersion=${fVersion}，groupId和version是执行shell命令的时候传进来的。为了保证依赖的正确性和一致性，每次打包我让所有的依赖的group和version都是一样的。也就是说不管第三方的插件原来的group和version是什么，我这里都统一指定成我们公司内部的group和version，这也会导致另一个问题，第三方的插件过多的话，导致我们的maven库上库也变多了，能不能再给group多一级收敛一下呢，比如xxx.xxx.thirdparty，是可以的这个在.android/build.gradle中处理。
PS:这里有个比较坑的地方就是，不管使我们自己创建的flutter plugin还是第三方的flutter plugin，在插件的build.gradle中会指定一个默认的group和version，导致刚开始打的flutter module的maven中的pom显示的依赖就是默认的，和我们实际上传的依赖的maven对不上，所以在.android/build.gradle中要覆盖。

```
// 在project.afterEvaluate中读取group version等参数并apply上传maven脚本
subprojects {
    project.afterEvaluate {
    project.plugins.withId('com.android.library') {
        if(project.name != 'app') {
            // 读取shell脚本中运行./gradlew xxx命令时传递的参数
            project.group = project.rootProject.hasProperty('gArtGroupId') ? project.rootProject.ext.gArtGroupId : null
            project.version = project.rootProject.hasProperty('gArtVersion') ? project.rootProject.ext.gArtVersion : null

            // 第三方的包在group里加上.thirdparty
            if(project.projectDir.absolutePath.indexOf("pub-cache/hosted/pub.flutter-io.cn") >= 0
|| project.projectDir.absolutePath.indexOf("pub-cache\\hosted\\") >= 0) {
                project.group += ".thirdparty"
            }

            println "==group: $project.group:$project.name:$project.version"

            // 给每个module添加上传maven脚本
            def mavenScriptPath = project.rootProject.file('../android_flutter_maven.gradle')
            project.apply from: mavenScriptPath
        }
    }
    }
}

// 很重要，用来覆盖各个自依赖中的group和version
// ./gradlew clean assembleRelease -PfGroupId=${fGroupId} -PfArtifactId=${fArtifactId} -PfVersion=${fVersion}
// 执行了这个命令，根目录的build.gradle就会执行，这时候把参数设置到project.rootProject.ext，后面各个subProject就可以拿到
final def artGroupId = project.hasProperty('fGroupId') && project.fGroupId ? project.fGroupId : null
final def artVersion = project.hasProperty('fVersion') && project.fVersion ? project.fVersion : null
project.rootProject.ext {
    gArtGroupId = artGroupId
    gArtVersion = artVersion
}
```

三、把flutter module打包成aar并上传到maven
上面所有的依赖都上传到maven了，现在可以把flutter module上传到maven了，代码基本上和处理依赖时一样

```
###### 3.上传flutterModule
cd .android

if [ "$debugMode" == "debug" ]
then
echo "==start build debug aar for [moduleflutter]"
./gradlew clean assembleDebug -PfGroupId=${fGroupId} -PfArtifactId=${fArtifactId} -PfVersion=${fVersion} -PfModule=true
else
echo "==start build release aar for [moduleflutter]"
# uploadArchives命令自动回生成release模式的aar，不用执行该命令
./gradlew clean assembleRelease -PfGroupId=${fGroupId} -PfArtifactId=${fArtifactId} -PfVersion=${fVersion} -PfModule=true
fi

echo "==start upload maven for [moduleflutter]"
./gradlew :flutter:uploadArchives -PfGroupId=${fGroupId} -PfArtifactId=${fArtifactId} -PfVersion=${fVersion} -PfModule=true

cd ../
```

四、切换源码和maven依赖变量设置
为了方便测试，我在.android/gradle.properties中设置了两个变量分别控制源码依赖和maven依赖，以及打本地maven还是远程maven.
gradle.properties
```
#0=>maven 1=>source
FLUTTER_SOURCE=1
#0=>clound 1=>local
CLOUND_MAVEN=1
```
.android/app/build.gradle
```
dependencies {

if ("1".equals(FLUTTER_SOURCE)) {
    api project(':flutter')
} else {
    // 现有的工程中引用flutter module库
    api('com.xxx:kwmoduleflutter:0.0.5-SNAPSHOT')
}
}
```
五、一键打包上传maven
只要执行这个命令就可以完成所有流程。
```
 ./androidFlutter 0.0.5-SNAPSHOT kwmoduleflutter com.xxx
```

到这里所有的流程都完成了，下面把完整的代码贴出来。涉及到的文件：
1. androidFlutter shell脚本
2. android_flutter_maven.gradle 上传maven脚本
3. .android/build.gradle 读取groupId artifactId version覆盖原有的group和version 
4. .android/gradle.properties 设置maven地址、源码和maven依赖切换变量

#### androidFlutter
```
#!/usr/bin/env bash

######################################################################
# 使用方式: ./androidFlutter version artifactId groupId
# 如： ./androidFlutter 0.0.1-SNAPSHOT kwmoduleflutter com.xxx
######################################################################

# 定义默认groupId
readonly DEF_GROUP='com.xxx'

# 读取参数
debugMode='release'
fVersion=$1         # flutter module的版本和所有的插件的版本保持一致
fArtifactId=$2      # flutter module的artifactId
fGroupId=$3         # 所有library的groupId都一样，默认是com.xxx

# 检查参数
if [ ! -n "$fArtifactId" -o ! -n "fVersion" ] ;then
    echo "==fArtifactId or fVersion is null"
    exit
fi

if [ ! -n "${fGroupId}" ] # 当串的长度不大于0时为真
then
    fGroupId=$DEF_GROUP
fi

###### 1. 获取/更新包
echo "==Start flutter packages get"
flutter packages get
echo -e "\n"

###### 2. 读取.flutter-plugin中的插件，进入对应的工程目录，编译成aar并上传
for line in $(cat .flutter-plugins)
do
    plugin_name=${line%%=*}
    plugin_path=${line##*=}
    echo '==Build and publish plugin: ['${plugin_name}']='${plugin_path}

    cd .android

#######################################################################
#这部分代码之前是想往每个依赖的工程中插入apply from: "./android_flutter_maven.gradle"，
#现在在.android/build.gradle中动态apply这种方案替换了
#
#   echo "==copy upload maven script file"
#   cp ../android_flutter_maven.gradle ${plugin_path}'/android'
#
#   # 给build.gradle第一行插入apply from: "./android_flutter_maven.gradle"
#   pushd .
#   cd ${plugin_path}'/android'
#
#   # 读取第一行比较是否已经插入apply from
#   #firstLine=$(sed -n '1p' build.gradle)
#   if [ "$firstLine" != 'apply from: "./android_flutter_maven.gradle"' ]
#   then
#       # mac上插入内容要换行才行
#       sed -i '' -e '1i \
#       apply from: "./android_flutter_maven.gradle"' build.gradle
#   else
#       echo "==flutter_maven already apply "
#   fi
#   popd
######################################################################

    # 在使用变量比较字符串之前，最好在判断之前加一个判断变量是否为空  或者使用双引号将其括起来，不然会出现异常[: ==: unary operator expected
    if [ "$debugMode" == "debug" ]
    then
       echo "==start build debug aar for plugin ["${plugin_name}"]"
       ./gradlew :${plugin_name}:clean :${plugin_name}:assembleDebug -PfGroupId=${fGroupId} -PfArtifactId=${plugin_name} -PfVersion=${fVersion}
    else
        echo "==start build release aar for plugin ["${plugin_name}"]"
        # uploadArchives命令自动回生成release模式的aar，不用执行该命令
        ./gradlew :${plugin_name}:clean :${plugin_name}:assembleRelease -PfGroupId=${fGroupId} -PfArtifactId=${plugin_name} -PfVersion=${fVersion}
    fi

    echo "==start upload maven for plugin ["${plugin_name}"]"
    ./gradlew :${plugin_name}:uploadArchives -PfGroupId=${fGroupId} -PfArtifactId=${plugin_name} -PfVersion=${fVersion}

    echo -e "\n"
    cd ../
done


###### 3.上传flutterModule
cd .android

#   echo "==copy upload maven script file"
#   cp ../android_flutter_maven.gradle ./Flutter
#
#   # 给build.gradle第一行插入apply from: "./android_flutter_maven.gradle"
#   pushd .
#   cd Flutter
#
#   firstLine=$(sed -n '1p' build.gradle)
#   if [ "$firstLine" != 'apply from: "./android_flutter_maven.gradle"' ]
#   then
#       # mac上插入内容要换行才行
#       sed -i '' -e '1i \
#       apply from: "./android_flutter_maven.gradle"' build.gradle
#   else
#       echo "==flutter_maven already apply "
#   fi
#   popd

if [ "$debugMode" == "debug" ]
then
   echo "==start build debug aar for [moduleflutter]"
   ./gradlew clean assembleDebug -PfGroupId=${fGroupId} -PfArtifactId=${fArtifactId} -PfVersion=${fVersion} -PfModule=true
else
    echo "==start build release aar for [moduleflutter]"
    # uploadArchives命令自动回生成release模式的aar，不用执行该命令
    ./gradlew clean assembleRelease -PfGroupId=${fGroupId} -PfArtifactId=${fArtifactId} -PfVersion=${fVersion} -PfModule=true
fi

echo "==start upload maven for [moduleflutter]"
./gradlew :flutter:uploadArchives -PfGroupId=${fGroupId} -PfArtifactId=${fArtifactId} -PfVersion=${fVersion} -PfModule=true

cd ../


```

#### android_flutter_maven.gradle
```
apply plugin: 'maven'

// 这里不需要artifacts，uploadArchives命令会自动生成并上传./build/outputs/flutter-release.aar，不然出现下面错误
// A POM cannot have multiple artifacts with the same type and classifier
//artifacts {
//    archives file('./build/outputs/flutter-release.aar')
//}

final def localMaven = "1".equals(CLOUND_MAVEN) //true: 发布到本地maven仓库， false： 发布到maven私服

final def artGroupId = project.group
final def artVersion = project.version
final def artifactId = project.hasProperty('fArtifactId') && project.fArtifactId ? project.fArtifactId : null
final def isFlutterModule = project.hasProperty('fModule') && project.fModule ? project.fModule : false


if(artifactId == null || artVersion == null) {
   return
}

// 因为只要执行./gradlew xxx等命令，rootProject和subProject的build.gradle都要执行一次，
// 所以这里要判断当前module和是否和正在处理的module一样，不是相同module就不处理，
// 但是因为不同业务的flutter module名称都是flutter并且artifactId和module名称又不相同，所以要通过isFlutterModule参数区分
if(!project.name.equals(artifactId) && (!project.name.equals("flutter") || !isFlutterModule)) {
    return
}

//project.group = artGroupId
//project.version = artVersion

uploadArchives {
    repositories {
        mavenDeployer {
            println "==maven url: ${artGroupId}:${artifactId}:${artVersion}"

            if(localMaven) {
                repository(url: uri(project.rootProject.projectDir.absolutePath + '/repo-local'))
            } else {
                repository(url: MAVEN_URL) {
                    authentication(userName: MAVEN_ACCOUNT_NAME, password: MAVEN_ACCOUNT_PWD)
                }

                snapshotRepository(url: MAVEN_URL_SNAPSHOT) {
                    authentication(userName: MAVEN_ACCOUNT_NAME, password: MAVEN_ACCOUNT_PWD)
                }
            }

            pom.groupId = artGroupId
            pom.artifactId = artifactId
            pom.version = artVersion

            pom.project {
                licenses {
                    license {
                        name 'The Apache Software License, artVersion 2.0'
                        url 'http://www.apache.org/licenses/LICENSE-2.0.txt'
                    }
                }
            }
        }
    }
}
```

#### .android/build.gradle
```
buildscript {
    repositories {
        maven { url "http://172.172.177.240:8081/nexus/content/repositories/releases" }
        maven { url "http://172.172.177.240:8081/nexus/content/repositories/snapshots" }
        maven { url "https://maven.aliyun.com/repository/google" }
        maven { url "https://maven.aliyun.com/repository/central" }
        maven { url "https://maven.aliyun.com/repository/jcenter" }

        google()
        jcenter()
    }

    dependencies {
        classpath 'com.android.tools.build:gradle:3.2.1'
    }
}

allprojects {
    repositories {
        maven { url "http://172.172.177.240:8081/nexus/content/repositories/releases" }
        maven { url "http://172.172.177.240:8081/nexus/content/repositories/snapshots" }
        maven { url "https://maven.aliyun.com/repository/google" }
        maven { url "https://maven.aliyun.com/repository/central" }
        maven { url "https://maven.aliyun.com/repository/jcenter" }

        google()
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}


// 在project.afterEvaluate中读取group version等参数并apply上传maven脚本
subprojects {
    project.afterEvaluate {
        project.plugins.withId('com.android.library') {
            if(project.name != 'app') {
                // 读取shell脚本中运行./gradlew xxx命令时传递的参数
                project.group = project.rootProject.hasProperty('gArtGroupId') ? project.rootProject.ext.gArtGroupId : null
                project.version = project.rootProject.hasProperty('gArtVersion') ? project.rootProject.ext.gArtVersion : null

                // 第三方的包在group里加上.thirdparty
                if(project.projectDir.absolutePath.indexOf("pub-cache/hosted/pub.flutter-io.cn") >= 0
                    || project.projectDir.absolutePath.indexOf("pub-cache\\hosted\\") >= 0) {
                    project.group += ".thirdparty"
                }

                println "==group: $project.group:$project.name:$project.version"

                // 给每个module添加上传maven脚本
                def mavenScriptPath = project.rootProject.file('../android_flutter_maven.gradle')
                project.apply from: mavenScriptPath
            }
        }
    }
}

// 很重要，用来覆盖各个自依赖中的group和version
// ./gradlew clean assembleRelease -PfGroupId=${fGroupId} -PfArtifactId=${fArtifactId} -PfVersion=${fVersion}
// 执行了这个命令，根目录的build.gradle就会执行，这时候把参数设置到project.rootProject.ext，后面各个subProject就可以拿到
final def artGroupId = project.hasProperty('fGroupId') && project.fGroupId ? project.fGroupId : null
final def artVersion = project.hasProperty('fVersion') && project.fVersion ? project.fVersion : null
project.rootProject.ext {
    gArtGroupId = artGroupId
    gArtVersion = artVersion
}
```

#### .android/gradle.properties
```
# 远程maven url和账号
MAVEN_URL=http://172.172.177.240:8081/nexus/content/repositories/releases
MAVEN_URL_SNAPSHOT=http://172.172.177.240:8081/nexus/content/repositories/snapshots
MAVEN_ACCOUNT_NAME=deployment
MAVEN_ACCOUNT_PWD=123456

#0=>maven 1=>source
FLUTTER_SOURCE=1
#0=>clound 1=>local
CLOUND_MAVEN=1
```
