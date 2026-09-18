### Gerrit 团队协同开发规范指南

### 📌 核心前置铁律（全员必背）

1. **本地物理分支（如 master）只用来拉取最新代码**，绝对禁止直接在 master 上写代码和提交。
2. **操作前必看状态**：去 Gerrit 网页更新/拿代码前，必须先在本地运行 git status。 

  * 如果工作区**有未提交的修改** ➡️ 先用 git stash 存入保险箱，保持工作区干净。
  * 如果本地**已经 commit** ➡️ 严禁使用 -B 或 --hard 强刷，必须用 git rebase。
3. **禁止产生 Merge 节点**：团队严禁使用普通的 git pull 或 git merge。同步他人未合入的代码时，必须使用 **--rebase** 参数。

### 💡 核心机制：依赖树 Rebase 模式

**🎯 本方案核心解决的痛点**：
本方案专门用于解决**“当 A 同学的代码还在 Gerrit 评审池中进行审核、尚未真正被点击 Submit（合入主干）时，依赖 A 代码的 B 同学该如何与其进行高效并行开发、高频同步代码且保证互不覆盖”**的硬核场景。 

### 🏃 案例演练：开发“商城购物车与支付”功能

* **A 同学** 本地在自己的 **feat-cart** 分支开发“购物车底层 API”（推送到 Gerrit 后生成 Change 2002，**未 Submit**）。
* **B 同学** 本地在自己的 **feat-pay** 分支开发“收银台支付界面”（依赖 A 且也**未 Submit**）。同时，B 还要处理其他紧急 Bug，并在本地产生了自己的多次 commit。

### 第一步：A 同学在本地特性分支开发底层并推送（此时代码处于未 Submit 状态）

1. **【在 A 的本地 master 分支】** 确保代码最新：git pull。
2. **【在 A 的本地 master 分支】** 新建并切换到自己的特性分支：git checkout -b feat-cart。
3. **【在 A 的本地 feat-cart 分支】** 写完购物车底层逻辑，提交并推送到 Gerrit： 

```bash

git add .
git commit -m "feat: 购物车增删改查API" 
git push origin HEAD:refs/for/master

```

*(此时 Gerrit 服务器生成 Change 2002，当前为 Patch Set 1。注意：此时代码还停留在网页端等待审核，**没有 Submit**)*

### 第二步：B 同学创建独立分支，并引入 A “尚未 Submit” 的代码并行开发

1. **【在 B 的本地 master 分支】** 确保本地基础代码是最新的：git pull。
2. **【在 B 的本地 master 分支】** 先基于当前 master 建立自己的本地特性分支：git checkout -b feat-pay
3. **【在 B 的本地 feat-pay 分支】** 去 Gerrit 2002 页面，复制其专属引用路径。通过 **fetch + rebase** 纯正的变基组合拳，将 A 尚未 Submit 的代码拉入自己当前的分支： 

```bash

# 1. 把 A 还没合入的代码抓取到本地的临时缓存中
git fetch https://gerrit.yourcompany.com/project refs/changes/02/2002/1

# 2. 正式变基！让自己的 feat-pay 分支以 A 的代码作为基座
git rebase FETCH_HEAD

```

*(此时 B 的 feat-pay 分支里，已经完美且干净地包含了 A 正在审核中的购物车 API 代码)*

4. **【在 B 的本地 feat-pay 分支】** 独立高频开发，连续提交两次（B1 和 B2）： 

```bash

git add . && git commit -m "feat: 支付接口对接" # 本地生成第一笔 commit B1
git add . && git commit -m "feat: 收银台UI美化" # 本地生成第二笔 commit B2

```

### 💡 第二步补充场景：B 同学开发到一半，突然接到任务去修 Bug（多任务切换）

1. **【在 B 的本地 feat-pay 分支】** 支付代码写到一半，紧急修复另一个 Bug： 

```bash

git stash                  # 1. 将没写完的私货放进临时保险箱
git checkout master        # 2. 切回本地 master
git checkout -b bugfix-hot # 3. 建新分支去修 Bug（修完正常 commit 并推 Gerrit）

```
2. **【在 B 的本地 bugfix-hot 分支】** Bug 修完后，切换回原来的支付分支继续开发： 

```bash

git checkout feat-pay      # 1. 切回支付分支
git stash pop              # 2. 弹出保险箱的私货，继续写完 B1 和 B2 提交

```
3. **【在 B 的本地 feat-pay 分支】** 将自己本地的多次提交推送到 Gerrit 评审： 

```bash

git push origin HEAD:refs/for/master

```

*(Gerrit 自动识别并生成依赖树。网页右侧 Related Changes 面板会完美展示：3002 -> 3001 -> 2002。此时它们**全部处于未 Submit 状态**)*

### 第三步：A 同学更新了未 Submit 的 2002，B 同学如何在本地安全变基同步？

开发到一半，A 修复了底层 API 的 Bug，更新了 2002 的内容（变成了 2002 的 **Patch Set 2**，**依然未 Submit**）。此时 B 的本地 feat-pay 分支上**已经积攒了 B1 和 B2 两个正式 commit**，我们必须使用 **rebase** 让 B 的提交重新排列在 A 的最新修补版之上。 

### 情况 1：B 同学工作区干净，无未保存的修改

1. **【在 B 的本地 feat-pay 分支】** 去 Gerrit 2002 页面切换到最新 Patch Set 2（未 Submit），通过 **pull --rebase** 强制进行变基合并： 

```bash

git pull --rebase https://gerrit.yourcompany.com/project refs/changes/02/2002/2

```

*(内幕：Git 会先把 B 本地的 B1、B2 提交临时抽离，将本地代码对齐成 A 的最新 PS2 版，然后再把 B1 和 B2 自动补在最上面。保证历史是一条直线)*
2. **【安全避坑：变基过程中提示 Conflict 冲突怎么办？】** 

  * **原因**：A 修改的底层接口，与 B 已经 commit 的支付逻辑产生了代码冲突。
  * **解决**：在本地打开冲突文件手动解掉。解完冲突后，**千万不要用普通的 commit**，必须运行以下命令继续变基： 

```bash

git add .
git rebase --continue

```

*(注：由于 B 本地有两个 commit，如果它们都和 A 的修改冲突，这个“解冲突 -> add -> continue”的过程可能会重复两次)*
3. **【在 B 的本地 feat-pay 分支】** 变基完成后，把本地分支再次推回 Gerrit： 

```bash

git push origin HEAD:refs/for/master

```

*(Gerrit 上的 3001 和 3002 会自动更新，且依赖树会自动挂载到 2002 的 Patch Set 2 下面)*

### 情况 2：B 同学手里还有“未写完、未 commit”的代码，突然需要变基同步 A 的修复

1. **【联调避坑：手头有私货时如何安全拉取？】** 先把手头写到一半的代码安全存入保险箱： 

```bash

git stash

```
2. **【在 B 的本地 feat-pay 分支】** 运行 **pull --rebase** 拉取 A 最新未 Submit 的 Patch Set 2 并变基： 

```bash

git pull --rebase https://gerrit.yourcompany.com/project refs/changes/02/2002/2

```

*(如果遇到冲突，请参考情况 1 的 git rebase --continue 方式解决)*
3. **【在 B 的本地 feat-pay 分支】** 变基完成后，从保险箱取出自己的私货继续开发： 

```bash

git stash pop

```
4. **【在 B 的本地 feat-pay 分支】** 写完剩余代码后，正常 commit 并推送到 Gerrit： 

```bash

git add .
git commit -m "feat: 支付模块最终收尾"
git push origin HEAD:refs/for/master

```

### 第四步：最终合并与收尾大结局（正式 Submit 阶段）

* **规则**：组长在网页端进行代码 Review。因为这一串 Change 有依赖关系，组长点击 **Submit** 时，必须**从下往上**点击。先 Submit A 的 2002，再 Submit B 的 3001 和 3002。

当组长在网页上将所有关联的 Change 均点击 **Submit** 真正合入主干并关闭后，B 同学进行最后的收尾： 

1. **【在 B 的本地 feat-pay 分支】** 任务完成，准备切回本地 master： 

```bash

git checkout master

```
2. **【在 B 的本地 master 分支】** 从远程物理 master 拉取刚刚被组长 Submit 进去的最终总成果： 

```bash

git pull origin master

```
3. **【在 B 的本地 master 分支】** 安全销毁本地的临时特性分支，不留任何垃圾： 

```bash

git branch -D feat-pay

```

*(此时 B 的本地只留下一条干干净净的 master 分支，完美收尾)*
