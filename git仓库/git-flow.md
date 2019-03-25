## 参考文章：
[为无线前端团队打造高效git工作流](https://juejin.im/post/5b2b76e251882574934c388d)


## 主要有以下分支：

| 分支 | 命名规则 |
| --- | --- |
| master |  |
| develop |  |
|  feature|  feature_mallapp_b2c_时间或者需求名称|
|  hotfix|  hotfix_mallapp_版本名称|
|  tag|  tag_mallapp_版本名称|		


### master  
是始终存在的，始终是稳定版本，也充当发布分支角色。只能合并代码不能修改代码。

### develop 
也是始终存在的，和master始终是并行的。一般是封板后合并到master，同时自动打包机器切换到master分支。  
**注意：可能存在多个业务线，发版时间也不同步，所以每个业务线都有自己的develop，这样各个业务不用相互等待或者相互干扰，master和develop的关系是1:n**
### feature 
是需求开发分支，从develop分支创建，最终在确认需求开发完毕，要提测了才合并回develop分支。
        待该版本发布了，应该立马删除。
### hotfix  
从tag拉出来，修复后合并master并同步到develop，应该也要立马删除。
### tag     
是在master发版了，打一个tag，方便版本回退。

## 注意：几个关键节点
在封板之前：
1. 自动打包机器配置成develop分支。
2. 修改bug，一定要在各自feature上修改，并合并到develop(最好由leader每次打测试包前统一合并)。
    目的是防止某个需求不上，方便回退！！！
    
在封板之后：
1. 自动打包机器配置成master分支。
2. 修改bug，最好在develop分支上(由于确定所有功能都可以上了，所以没必要继续在feature上修改了)，
    并合并到master(最好由leader每次打测试包前统一合并)。

## 番外篇，帮助理解
develop分支存在意义？不可以从tag或者master上直接创建feature吗？
1. tag存在的意义一般不是作为创建分支，请参考tag存在的意义
2. 即使可以从tag创建feature分支，如果每个需求开发周期是连贯的，不存在时间上的重叠器，这样是可以从tag直接拉分支的。  
但是实际情况是，迭代很频繁，可能这个需求还没发布，测试还没完成，就要基于当前feature开发新的
需求，那这时候只有在develop分支上创建新的feature。
3. 如果没有develop分支，会导致很多feature直接合并到master，这样就不存在一个稳定的版本分支，如果此时有一个基于
上次稳定的版本开发一个需求，这个分支又从哪里创建呢。


tag存在的意义？
1. 基于历史某个版本添加一个新功能，发布一个临时版本。
2. 某个版本存在bug，修复改版本的bug。如果是最新的版本出现bug，是可以从master创建分支修复bug的。
3. 回退到某个版本。比找commit id更简单。