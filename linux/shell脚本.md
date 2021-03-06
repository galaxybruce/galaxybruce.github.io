## 参考文章：
1. [Shell 教程](http://www.runoob.com/linux/linux-shell.html)
2. [Linux下Shell脚本字符串单引号、双引号、反引号、反斜杠的作用和区别](https://www.cnblogs.com/EasonJim/p/8018545.html)

## shell脚本作用
以Linux系统为例，linux系统里面有很多个命令，脚本可以是一个或者多个命令的集合，通过运行脚本，达到既定的功能或者效果。Shell脚本可以帮助我们系统、自动化的去管理和处理一些东西，举个例子，我们在做集成自动化测试的过程当中，经常都需要做的几步工作
1. 更新代码（svn、git）
2. 更新第三方库（cocoapods）
3. 提交代码到测试平台（fir、bugly）

此过程中可能会需要用到一系列的命令和一些手工操作，使用Shell脚本，我们可以让这一系列的过程自动执行。

## shell与bat区别
shell script就像早期dos年代的.bat
bat是windows平台下的批处理
shell一般是指linux下的shell

## 运行 Shell 脚本有两种方法:
1、作为可执行程序：
1. 执行脚本一定要指明路径，在当前路径也要指明,如./test.sh;不在当前路径需指明全路径
2. 使脚本具有执行权限 chmod +x 脚本路径
3. 文件第一行要指明解释器类型，告诉系统其后路径所指定的程序即是解释此脚本文件的 Shell 程序。
如 #!/bin/sh、 #!/bin/bash

2、作为解释器参数
这种运行方式是，直接运行解释器，其参数就是 shell 脚本的文件名，如：
/bin/sh test.sh
/bin/php test.php
这种方式运行的脚本，不需要在第一行指定解释器信息，写了也没用。

## 当前shell的配置文件
echo $SHELL 查看当前shell版本, 使用的shell不同, 对应的配置文件也不一样

* 如果是bash
```
vim ~/.bash_profile
//或者
vim ~/.bashrc
```
* 如果是zsh
```
vim ~/.zshrc
```
PS.通过如下命令可切换shell
```
//切换到zsh
chsh -s `which zsh` 
//切换到bash
chsh -s `which bash`
```
## Shell 文件包含
在文件顶部用如下代码
. filePath #注意点号(.)和文件名中间有一空格
或
source filePath

## 反引号
两个反引号包围起来的字符串，将作为命令来运行，执行的输出结果作为该反引号的内容，称为命令替换，它有另一种更好的写法: $(command)

## 变量
注意，变量名和等号之间不能有空格
### 只读变量
使用 readonly 命令可以将变量定义为只读变量，只读变量的值不能被改变。
```
myUrl="http://www.google.com"
readonly myUrl
myUrl="http://www.runoob.com"
```

### 删除变量
使用 unset 命令可以删除变量，不可以删除readonly变量。语法：
```
unset variable_name
```

## shell数组
### 定义数组
在 Shell 中，用括号来表示数组，数组元素用"空格"符号分割开。定义数组的一般形式为：
```
数组名=(值1 值2 ... 值n)
```
还可以单独定义数组的各个分量,可以不使用连续的下标，而且下标的范围没有限制。：
```
array_name[0]=value0
array_name[1]=value1
array_name[n]=valuen
```
### 读取数组
```
${数组名[下标]}
```
使用 @ 或者 * 符号可以获取数组中的所有元素，例如：
```
echo ${array_name[@]}
```
### 获取数组的长度
```
取得数组元素的个数
length=${#array_name[@]}
或者length=${#array_name[*]}
取得数组单个元素的长度
lengthn=${#array_name[n]}
```

## Shell传递参数
在命令后面用空格分隔参数，脚本中用$n接收，$0表示脚本文件名。
```
./test.sh 1 2 3
echo "Shell 传递参数实例！";
echo "执行的文件名：$0";
echo "第一个参数为：$1";
echo "第二个参数为：$2";
```
## 别名alias传参数
[给 alias 添加命令行参数](http://toy.linuxtoy.org/2012/03/20/pass-command-line-argument-to-alias.html)
就是把要执行的命令定义在一个函数中
注意每一行代码后最好加上分号，不然可能失败。
```
// 安装带有日期和版本号的apk
alias xxxDebug='fakeMethod() { adb install -r /Users/bruce/work/sourcecode/Projects_Android/git/xxx/app/build/outputs/apk/pc/debug/com.xxx._pc_$(date "+%Y%m%d")_$1.apk; };fakeMethod'

// 命令输入
xxxDebug '1.6.0'
```

## shell运算符
注意点：

1. 表达式和运算符之间要有空格，例如 2+2 是不对的，必须写成 2 + 2
2. 完整的表达式要被反引号包含，注意这个字符不是常用的单引号，在 Esc 键下边。不过推荐用 \$() 代替 \`\`
3. 原生bash不支持简单的数学运算，但是可以通过其他命令来实现，例如 awk 和 expr，expr 最常用
val=\`expr 2 + 2\`
echo "两数之和为 : $val"
4. 条件表达式要放在方括号之间，并且要有空格，例如: [\$a==\$b] 是错误的，必须写成 [ \$a == \$b ]
5. 乘号(\*)前边必须加反斜杠(\\)才能实现乘法运算；
val=\`expr $a \\\* $b\`

表达式四种写法：
val=\`expr 2 + 2\`
val=\$(expr 2 + 2)
val=\$((2 + 2))
var=\$[2 + 2]
## 符号$后的括号
（1）\${a} 变量a的值, 在不引起歧义的情况下可以省略大括号。
（2）\$(cmd) 命令替换，和\`cmd\`效果相同，结果为shell命令cmd的输，过某些Shell版本不支持\$()形式的命令替换, 如tcsh。
（3）\$((expression)) 和\`exprexpression\`效果相同, 计算数学表达式exp的数值, 其中exp只要符合C语言的运算规则即可, 甚至三目运算符和逻辑表达式都可以计算。
## 字符串运算符
字符串有专用的运算符 用来计算字符串是否相等、长度是否为0、字符串是否为空等

## 文件测试运算符
文件测试运算符用于检测 Unix 文件的各种属性。如文件是普通文件还是目录、是否可写、是否只读

## Shell echo命令
echo输出的字符串总结
| | 能否引用变量 | 能否引用转移符 | 能否引用文本格式符(如：换行符、制表符) |
| --- | --- | --- | --- |
| 单引号 | 否 | 否 | 否 |
| 双引号 | 能 | 能 | 能 |
| 无引号 | 能 | 能 | 否 |
显示结果定向至文件
echo "It is a test" > myfile
原样输出字符串，不进行转义或取变量(用单引号)
echo '$name\"' \#输出\$name\"
显示命令执行结果
echo \`date\` \#输出日期Thu Jul 24 10:08:46 CST 2014

## 函数
略
## 输入/输出重定向
略

## 常用命令
1. echo $SHELL 查看当前shell版本
2. help [命令] 查看某个命令的用法


## 常用操作案例
1. 美团外卖操作so库
```
cd $FLUTTER_ROOT/bin/cache/artifacts/engine
for arch in android-arm android-arm-profile android-arm-release; do
pushd $arch
cp flutter.jar flutter-armeabi-v7a.jar # 备份
unzip flutter.jar lib/armeabi-v7a/libflutter.so
mv lib/armeabi-v7a lib/armeabi
zip -d flutter.jar lib/armeabi-v7a/libflutter.so
zip flutter.jar lib/armeabi/libflutter.so
popd
done
```

2. [xargs将搜索到文件更换文件名或者移动目录](https://www.cnblogs.com/chyingp/p/linux-command-xargs.html)
```
// 将所有的.js结尾的文件，都加上.backup后缀
ls *.js | xargs -t -I '{}' mv {} {}.backup
// 将7天前的日志备份到特定目录
find . -mtime +7 | xargs -I '{}' mv {} /tmp/otc-svr-logs/
```