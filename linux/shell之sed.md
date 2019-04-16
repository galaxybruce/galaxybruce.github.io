## 参考文章：
1. [mac下的sed命令教程](https://www.zealllot.com/posts/mac%E4%B8%8B%E7%9A%84sed%E5%91%BD%E4%BB%A4%E6%95%99%E7%A8%8B/)
2. [SED 简明教程](https://coolshell.cn/articles/9104.html)

## 注意点：
1. 换行，Linux版的sed可以使用\n，但是mac版需要使用\'$'\n
2. 如果你要使用单引号，那么你没办法通过\’这样来转义，就有双引号就可以了，在双引号内可以用\”来转义。

#### 在第一行插入内容，注意：mac上插入的内容要换行才行
```
sed -i '' -e '1i \
apply from: "./android_flutter_maven.gradle"' build.gradle
```

#### 替换build.gradle中的所有apply from
sed -i ’‘ "s/^apply from:.*kidswant_maven.gradle'$/哈哈哈哈哈/g" build.gradle