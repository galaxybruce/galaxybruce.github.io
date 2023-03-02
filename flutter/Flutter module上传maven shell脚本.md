### shell脚本androidFlutter
```
#!/usr/bin/env bash

######################################################################
# 使用方式: ./androidFlutter version artifactId groupId
# 如： ./androidFlutter 0.0.14-SNAPSHOT kwmoduleflutter com.galaxybruce.flutter
######################################################################

# 定义默认groupId
readonly DEF_GROUP='com.galaxybruce.flutter'

# 读取参数
debugMode='release'
fVersion=$1         # flutter module的版本和所有的插件的版本保持一致
fArtifactId=$2      # flutter module的artifactId
fGroupId=$3         # 所有library的groupId都一样，默认是com.galaxybruce

# 检查参数
if [ ! -n "$fArtifactId" -o ! -n "$fVersion" ] ;then
    echo "==fArtifactId or fVersion is null"
    exit
fi

if [ ! -n "${fGroupId}" ] # 当串的长度不大于0时为真
then
    fGroupId=$DEF_GROUP
fi

###### 1. 获取/更新包
echo "==Start flutter packages get"
# flutter packages get 这里改用自定义脚本命令
addDepFile="./tools/add_dependency.dart"
if [ ! -f "addDepFile" ]; then
 echo "add_dependency file exits"
 # 先备份，整个流程执行完毕后再恢复
 cp -rf ./pubspec.yaml ./pubspec.yaml.bak
 dart $addDepFile getForLong
else
 echo "add_dependency file not exits"
 flutter packages get
fi

echo -e "\n"

###### 2. copy更目录build.gradle文件覆盖.android/build.gradle（每次执行flutter packages get或者upgrade命令，.android目录都会被覆盖）
cp -rf ./tools/root_build.gradle ./.android/build.gradle


###### 3. 读取.flutter-plugin中的插件，进入对应的工程目录，编译成aar并上传
for line in $(cat .flutter-plugins)
do
    # 过滤掉注释行
    if [[ "$line" =~ ^#.* ]]; then
      continue
    fi

    plugin_name=${line%%=*}
    plugin_path=${line##*=}
    if [ "$plugin_name" == "$plugin_path" ]; then
      continue
    fi

    echo '==Build and publish plugin: ['${plugin_name}']='${plugin_path}

    cd .android

    # 在使用变量比较字符串之前，最好在判断之前加一个判断变量是否为空  或者使用双引号将其括起来，不然会出现异常[: ==: unary operator expected
    if [ "$debugMode" == "debug" ]
    then
       echo "==start build debug aar for plugin ["${plugin_name}"]"
       ./gradlew :${plugin_name}:clean :${plugin_name}:assembleDebug -PfGroupId=${fGroupId} -PfArtifactId=${plugin_name} -PfVersion=${fVersion}
    else
        echo "==start build release aar for plugin ["${plugin_name}"]"
        # uploadArchives命令自动回生成release模式的aar，不用执行该命令
        # ./gradlew :${plugin_name}:clean :${plugin_name}:assembleRelease -PfGroupId=${fGroupId} -PfArtifactId=${plugin_name} -PfVersion=${fVersion}
    fi

    echo "==start upload maven for plugin ["${plugin_name}"]"
    ./gradlew :${plugin_name}:clean :${plugin_name}:uploadArchives -PfGroupId=${fGroupId} -PfArtifactId=${plugin_name} -PfVersion=${fVersion}

    echo -e "\n"
    cd ../
done


###### 4.上传flutterModule
cd .android

if [ "$debugMode" == "debug" ]
then
   echo "==start build debug aar for [moduleflutter]"
   ./gradlew clean assembleDebug -PfGroupId=${fGroupId} -PfArtifactId=${fArtifactId} -PfVersion=${fVersion} -PfModule=true
else
    echo "==start build release aar for [moduleflutter]"
    # uploadArchives命令自动会生成release模式的aar，不用执行该命令
#    ./gradlew clean assembleRelease -PfGroupId=${fGroupId} -PfArtifactId=${fArtifactId} -PfVersion=${fVersion} -PfModule=true
fi

echo "==start upload maven for [moduleflutter]"
./gradlew :flutter:clean :flutter:uploadArchives -PfGroupId=${fGroupId} -PfArtifactId=${fArtifactId} -PfVersion=${fVersion} -PfModule=true

cd ../


###### 5. 恢复pubspec.yaml文件
if [ -f "./pubspec.yaml.bak" ]; then
    cp -rf ./pubspec.yaml.bak ./pubspec.yaml
    rm -f ./pubspec.yaml.bak
fi
```

### pubspec.yaml动态添加依赖 add_dependency.dart

```
/// date: 2019-09-05 14:11
/// author: bruce.zhang
/// description: 动态添加依赖
/// 使用方式:
/// dart ./tools/add_dependency.dart get
/// dart ./tools/add_dependency.dart upgrade
///
/// modification history:
import 'dart:io';

import "package:path/path.dart" as p;
import 'package:process_run/shell.dart';
import 'package:yaml/yaml.dart';


/// (^\s*dependency_overrides\s*\:[\s\S]*)(\s+front_end\s*\:)(\s+git\s*\:)(\s+[a-z]+\s*\:.*)(\s+[a-z]+\s*\:.*)(\s+[a-z]+\s*\:.*)
/// (^(?!#).*dependency_overrides\s*\:[\s\S]*)(\s+kernel\s*\:)(\s+[a-z]+\s*\:.*){1,4}
void main(List<String> args) async {
  final Directory curDir = Directory(Platform.script.toFilePath());
  var dependencySpec = new File(p.join(curDir.parent.path, 'add_dependency.yaml'));
  if(!dependencySpec.existsSync()) {
    _executeShell(args);
    return;
  }

  YamlMap yamlMap = loadYamlNode(dependencySpec.readAsStringSync());
  if(yamlMap == null || yamlMap.nodes == null || yamlMap.nodes.isEmpty) {
    _executeShell(args);
    return;
  }

  yamlMap = yamlMap.nodes['dependency_overrides'];
  if(yamlMap == null || yamlMap.nodes == null || yamlMap.nodes.isEmpty) {
    _executeShell(args);
    return;
  }

  // 1.1. 读取yaml，用正则匹配内容，查看是否存在，不存在就插入
  var pubSpec = new File(p.join(curDir.parent.parent.path, 'pubspec.yaml'));
  final pubSpecContent = pubSpec.readAsStringSync();
  var flyPubSpecContent = pubSpecContent;

  if(_haveDependencyOverides(pubSpecContent)) {
    print('dependency_overrides exits:');

    yamlMap.nodes.forEach((key, value) {
      String regExpSource = r'((\r\n|\r|\n)\s*dependency_overrides\s*\:[\s\S]*)(\s+$key\s*\:\s*)((\s*[a-z]+\s*\:.*){1,4})';
      regExpSource = regExpSource.replaceAll(r"$key", key.value);
      RegExp regExp = new RegExp(regExpSource);
      Iterable<Match> allMatches = regExp.allMatches(flyPubSpecContent);
      // dependency_overrides中没有就在第一项前面插入
      if(allMatches.isEmpty) {
        flyPubSpecContent = flyPubSpecContent.replaceAllMapped(new RegExp(r'((\r\n|\r|\n)\s*dependency_overrides\s*\:\s*)'), (m) {
          String space = m.group(0).replaceAll(m.group(0).trimRight(), '');
          space = space.replaceAll('\n', '');

          String replace = '$key:';
          if(value is YamlMap) {
            if(value['path'] != null) {
              replace += '\n$space  path: ${value['path']}';
            } else if(value['git'] != null) {
              // 获取path或者git左边的空格
              replace += '\n$space  git:';

              var git = value['git'];
              if(git['url'] != null) {
                replace += '\n$space    url: ${git['url']}';
              }
              if(git['ref'] != null) {
                replace += '\n$space    ref: ${git['path']}';
              }
              if(git['path'] != null) {
                replace += '\n$space    path: ${git['path']}';
              }
            }
          } else {
            // flutter_component: '1.0.0'
            replace += ' ${value.value}';
          }
          return m.group(0) + '$replace\n$space';
        });
      } else {
        // dependency_overrides中有就替换
        // 这里其实可以不用循环，allMatches.length == 1
        for (Match m in allMatches) {
          String replace = '';
          if(value is YamlMap) {
            if(value['path'] != null) {
              replace += 'path: ${value['path']}';
            } else if(value['git'] != null) {
              // 获取path或者git左边的空格
              final String space = m.group(3).replaceAll(m.group(3).trimRight(), '');
              replace += 'git:';

              var git = value['git'];
              if(git['url'] != null) {
                replace += '$space  url: ${git['url']}';
              }
              if(git['ref'] != null) {
                replace += '$space  ref: ${git['path']}';
              }
              if(git['path'] != null) {
                replace += '$space  path: ${git['path']}';
              }
            }
            print(replace);
            flyPubSpecContent = flyPubSpecContent.replaceAll(m.group(4), replace);
          } else {
            // flutter_component: '1.0.0'
            replace = value.value;
            flyPubSpecContent = flyPubSpecContent.replaceAll(m.group(3) + m.group(4), m.group(3).trimRight() + ' $replace');
          }
        }
      }
    });
  } else {
    print('dependency_overrides not exits:');
    flyPubSpecContent += '\n' + dependencySpec.readAsStringSync();
  }

  pubSpec.writeAsStringSync(flyPubSpecContent, flush: true);
//  print('\n\n$flyPubSpecContent');

  // 2. flutter packages get
  await _executeShell(args);

  if(args == null || args.isEmpty) {
    return;
  }
  // 3. 删除yaml中添加的内容(注意：通过androidFlutter命令打包时不能删除，这时候参数是getForLong)
  if(args[0] != "getForLong") {
    pubSpec.writeAsStringSync(pubSpecContent, flush: true);
  }
}

/// 已换行符号开头 空格跟随判断是否dependency_overrides有效
bool _haveDependencyOverides(String pubSpecContent) {
  // ([(\r\n)|\r|\n]\s*dependency_overrides\s*\:)
  String regExpSource = r'((\r\n|\r|\n)\s*dependency_overrides\s*\:)';
  RegExp regExp = new RegExp(regExpSource);
  return regExp.hasMatch(pubSpecContent);
}

_executeShell(List<String> args) async{
  if(args == null || args.isEmpty) {
    return;
  }
  var shell = Shell();

  if(args[0] == "get" || args[0] == "getForLong") {
    await shell.run('''flutter packages get''');
  } else if(args[0] == "upgrade"){
    await shell.run('''flutter packages upgrade''');
  }
}

```