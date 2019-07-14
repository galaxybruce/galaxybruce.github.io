```
#!/usr/bin/env bash

######################################################################
# 使用方式: ./androidFlutter version artifactId groupId
# 如： ./androidFlutter 0.0.14-SNAPSHOT kwmoduleflutter com.galaxybruce.flutter
# 注意，在使用该命令之前先把.android/gradle.properties中的FLUTTER_SOURCE变为1
######################################################################

# 定义默认groupId
readonly DEF_GROUP='com.galaxybruce.flutter'

# 读取参数
debugMode='release'
fVersion=$1         # flutter module的版本和所有的插件的版本保持一致
fArtifactId=$2      # flutter module的artifactId
fGroupId=$3         # 所有library的groupId都一样，默认是com.galaxybruce

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

###### 2. copy更目录build.gradle文件覆盖.android/build.gradle（每次执行flutter packages get或者upgrade命令，.android目录都会被覆盖）
cp -rf ./gradle/root_build.gradle ./.android/build.gradle


###### 3. 读取.flutter-plugin中的插件，进入对应的工程目录，编译成aar并上传
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


###### 4.上传flutterModule
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