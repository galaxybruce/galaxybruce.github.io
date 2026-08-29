# pre-commit 简介与使用文档

## 1. 什么是 pre-commit

`pre-commit` 是一个 Git Hook 管理工具，用来在 `git commit`、`git push` 等时机自动执行一组预定义检查。

它主要用于：

- 在提交前自动修正常见格式问题
- 在提交前阻止明显错误进入仓库
- 统一团队成员的基础提交质量标准

通常项目配置文件位于仓库根目录：

- `.pre-commit-config.yaml`

## 2. 常见检查项

常见配置如下：

- `trailing-whitespace`
  - 删除每行结尾多余空格
- `end-of-file-fixer`
  - 确保文件末尾只有一个换行
- `check-yaml`
  - 检查 YAML 文件语法
- `check-merge-conflict`
  - 检查是否遗留 Git 冲突标记
- `check-added-large-files`
  - 阻止误提交过大的文件
- `eslint`
  - 对 `js / jsx / ts / tsx` 文件执行 `npx eslint --fix`

## 3. 配置文件结构说明

通用配置示例：

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-merge-conflict
      - id: check-added-large-files

  - repo: local
    hooks:
      - id: eslint
        name: ESLint
        entry: npx eslint --fix
        language: system
        types_or: [javascript, jsx, ts, tsx]
        pass_filenames: true
```

关键字段含义：

- `repo`
  - hook 来源仓库
- `rev`
  - 指定 hook 仓库版本
- `hooks`
  - 当前仓库启用的具体规则
- `id`
  - hook 的配置标识
- `entry`
  - hook 实际执行的命令
- `language: system`
  - 使用当前机器环境执行命令，不额外创建独立运行时
- `types_or`
  - 指定触发文件类型
- `pass_filenames: true`
  - 将本次变更文件路径传给命令

## 4. 命令执行流程

### 4.1 初始化流程

首次接入一个仓库时，建议先进入该仓库的根目录，再完成初始化。

其中要特别注意：

- `.pre-commit-config.yaml` 默认不会自己出现
- 它通常需要开发者手动创建
- 也可以用 `pre-commit sample-config` 先自动生成一份模板，再按项目需要修改
- 按官方文档推荐顺序，应先准备 `.pre-commit-config.yaml`，再执行 `pre-commit install`
- `pre-commit run --all-files` 属于可选步骤，通常用于首次接入或新增 hooks 后的全量检查

```bash
cd /path/to/your-repo
python3 -m pre_commit --version
# 方式一：手动创建 .pre-commit-config.yaml
# 方式二：自动生成模板 pre-commit sample-config > .pre-commit-config.yaml
pre-commit install
pre-commit run --all-files
```

含义如下：

1. 先切换到目标 Git 仓库的根目录
2. 检查 `pre-commit` 是否已安装
3. 在仓库根目录准备 `.pre-commit-config.yaml`
   - 可以手动创建并填写项目需要的 hooks
   - 也可以先执行 `pre-commit sample-config > .pre-commit-config.yaml` 自动生成初始模板
4. 将 Git Hook 安装到当前仓库的 `.git/hooks/pre-commit`
5. 可选：手动对整个仓库跑一遍规则，提前清理历史问题

### 4.2 日常提交流程

日常开发时的典型流程：

```bash
git add <files>
git commit -m "feat: xxx"
```

此时 Git 会自动触发：

1. `.git/hooks/pre-commit`
2. `pre-commit` 读取仓库根目录的 `.pre-commit-config.yaml`
3. 按规则筛选本次提交涉及的文件
4. 执行对应 hooks
5. 全部通过则继续提交
6. 若有失败或自动修复，则提交中断，需重新 `git add` 后再提交

### 4.3 自动修复类 hook 的行为

例如：

- `trailing-whitespace`
- `end-of-file-fixer`
- `eslint --fix`

这些 hook 可能会直接修改文件。出现这种情况时，正常现象是：

```bash
git add <files>
git commit -m "feat: xxx"
```

执行后被拦住，然后你再执行：

```bash
git add <files>
git commit -m "feat: xxx"
```

第一次是修复，第二次才是提交。

## 5. 命令应该在哪个目录执行

这是最容易混淆的部分。

### 5.1 `pre-commit install`

应在 **Git 仓库内部执行**，最佳位置是 **仓库根目录**。

推荐命令：

```bash
cd /path/to/your-repo
pre-commit install
```

原因：

- 它会把 hook 安装到当前仓库的 `.git/hooks/`
- 如果不在 Git 仓库里执行，会报错

### 5.2 `pre-commit run --all-files`

也应在 **仓库根目录** 执行。

推荐：

```bash
cd /path/to/your-repo
pre-commit run --all-files
```

原因：

- 它需要读取当前仓库根目录下的 `.pre-commit-config.yaml`
- 某些本地命令如 `npx eslint --fix` 依赖当前仓库的 `package.json` 和 `node_modules`

### 5.3 `pre-commit run <hook-id>`

也建议在仓库根目录执行，例如：

```bash
cd /path/to/your-repo
pre-commit run trailing-whitespace --all-files
pre-commit run eslint --all-files
```

### 5.4 可以在哪些目录执行

经验规则：

- 在 **仓库根目录执行**：最稳妥
- 在 **仓库子目录执行**：有时也能工作，但不推荐作为标准操作
- 在 **非仓库目录执行**：不应该执行

## 6. 常用命令速查

### 6.1 查看版本

```bash
pre-commit --version
```

若 PATH 尚未生效，也可用：

```bash
python3 -m pre_commit --version
```

### 6.2 安装 hook

```bash
cd /path/to/your-repo
pre-commit install
```

### 6.3 运行全部检查

```bash
cd /path/to/your-repo
pre-commit run --all-files
```

### 6.4 只运行某一个 hook

```bash
cd /path/to/your-repo
pre-commit run trailing-whitespace --all-files
pre-commit run eslint --all-files
```

### 6.5 更新 hook 版本

```bash
cd /path/to/your-repo
pre-commit autoupdate
```

更新后建议：

```bash
pre-commit run --all-files
```

### 6.6 卸载 hook

```bash
cd /path/to/your-repo
pre-commit uninstall
```

## 7. 错误与排查思路

### 7.1 提交时提示文件被修改

说明某些 hook 自动修复了文件，例如：

- 删除了行尾空格
- 补了文件末尾换行
- ESLint 自动修复了格式

处理方式：

```bash
git add <files>
git commit -m "your message"
```

### 7.2 提示找不到 `.pre-commit-config.yaml`

通常是因为命令不在项目根目录执行，或者配置文件缺失。

先检查：

```bash
pwd
ls -la
```

### 7.3 提示 `pre-commit: command not found`

这通常表示：

- `pre-commit` 已经安装了，但它所在的目录没有加入当前终端的 `PATH`
- 或者你虽然已经配置过 `PATH`，但当前终端会话还没有重新加载这份配置

可先验证：

```bash
python3 -m pre_commit --version
```

如果这条命令能执行成功，说明：

- Python 里的 `pre-commit` 包本身已经安装好了
- 问题只是在终端里直接输入 `pre-commit` 时，系统暂时找不到这个命令

这时通常需要：

- 补充 `PATH` 配置
- 或重新打开终端 / 重新加载 shell 配置文件

### 7.4 `eslint` 失败

常见原因：

- Node 依赖未安装
- 当前目录不是项目根目录
- 代码里确实存在 ESLint 无法自动修复的问题

排查命令示例：

```bash
cd /path/to/your-repo
npx eslint . --ext .js,.jsx,.ts,.tsx
```

## 8. 和 AI 工具配合使用的建议

`pre-commit` 非常适合和 AI 编程工具配合，因为 AI 生成代码后，最需要的是一个稳定、自动、可重复的质量闸门。

### 8.1 推荐协作流程

建议流程如下：

1. AI 先生成或修改代码
2. AI 执行：

```bash
cd /path/to/your-repo
pre-commit run --all-files
```

3. AI 根据输出修复问题
4. 再次执行 `pre-commit run --all-files`
5. 通过后再提交代码

### 8.2 为什么适合和 AI 配合

原因主要有三点：

- `pre-commit` 能兜底格式和基础错误
- AI 修改较多文件时，容易遗漏细节，hook 可以统一校验
- 规则可重复执行，避免“这次修好了，下次又坏了”

### 8.3 AI 使用时的具体建议

#### 建议一：先改代码，再跑 hook

不要只让 AI “解释配置”，应让它在改完代码后主动执行：

```bash
pre-commit run --all-files
```

#### 建议二：把失败结果继续喂给 AI

如果 hook 失败，把输出交给 AI，让它继续修复。例如：

- 哪个文件被 `trailing-whitespace` 修了
- 哪个文件 `eslint` 还有错误
- 哪个 YAML 文件语法不合法

#### 建议三：把 pre-commit 当成 AI 的验收器

一个很实用的原则：

- AI 负责生成和修改
- `pre-commit` 负责做第一层验收
- 人再做第二层业务确认

### 8.4 通用推荐习惯

每次 AI 完成代码改动后，建议至少执行：

```bash
cd /path/to/your-repo
pre-commit run --all-files
<project test command>
```

如果只是改了少量某类文件，也可以先执行：

```bash
cd /path/to/your-repo
pre-commit run <hook-id> --all-files
```

## 9. 建议的团队实践

对任意项目，建议采用以下约定：

1. 所有开发者 clone 仓库后先执行：

```bash
pre-commit install
```

2. 修改 `.pre-commit-config.yaml` 后执行：

```bash
pre-commit run --all-files
```

3. AI 或人工在提交前，都尽量手动执行一次：

```bash
pre-commit run --all-files
```

4. 若 hook 自动改了文件，重新 `git add` 后再提交

## 10. 一句话总结

`pre-commit` 的作用可以概括为：

> 在代码真正进入 Git 历史之前，先做一轮自动化清洗和基础质量检查。

在任意项目里，最稳妥的使用方式就是：

```bash
cd /path/to/your-repo
pre-commit run --all-files
git add <files>
git commit -m "..."
```
