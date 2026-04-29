## 第一步：在 GitHub 网页端操作 (准备仓库)

1. **登录 GitHub**：访问 [ABACUS 仓库地址](https://github.com/deepmodeling/abacus-develop) 。
    
2. **点击 Fork**：在页面右上角点击 **Fork** 按钮，这会在你自己的 GitHub 账号下创建一个完全一样的代码副本（例如 `your-username/abacus-develop`）。
    

## 第二步：在 Trae IDE / 云虚拟机中操作 (下载与修改)

由于你已经通过 SSH 连接了虚拟机，直接在 IDE 的终端里执行：

1. **Clone 你自己的仓库**：

```bash
    # 注意这里是你 Fork 出来的那个地址，不是官方地址
    git clone https://github.com/你的用户名/abacus-develop.git
    cd abacus-develop
```
2. **建立开发分支**：
```bash
    # 养成好习惯，不在 develop 分支直接改，建一个新分支
    git checkout -b hw5-fix-case1
```

3. **修改代码**：在 IDE 的文件树里找到作业指定的对应文件（例如 `source/source_pw/module_stodft/sto_wf.cpp`）进行修改 。

4. **编译测试**（可选但推荐）：根据项目文档尝试编译，确保你的修改没有语法错误。

## 第三步：提交并同步到 GitHub

1. **提交本地修改**：
```
    git add .
    git commit -m "2026PKUCourseHW5: Fix magic number in sto_wf.cpp" 
```
2. **推送到 GitHub**
```
    git push origin hw5-fix-case1
```
## 第四步：回到 GitHub 网页端 (发起 PR)

1. 进入你自己的 `abacus-develop` 仓库页面，你会看到一个黄色的提示框，写着 **"Compare & pull request"**。
2. **点击并填写标题**：标题必须按照作业要求写成 `2026PKUCourseHW5: XXX` 。
3. **确认提交**：点击 "Create pull request"。