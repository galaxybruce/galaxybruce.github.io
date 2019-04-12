### 在第一行插入内容，注意：mac上插入的内容要换行才行
```
sed -i '' -e '1i \
apply from: "./android_flutter_maven.gradle"' build.gradle
```