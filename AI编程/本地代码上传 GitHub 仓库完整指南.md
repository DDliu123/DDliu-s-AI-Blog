


## 一、前置必备准备（必须先完成）

### 1. 安装Git客户端

Git是实现本地与GitHub通信的核心工具，未安装无法执行后续操作。

- 下载地址：[Git官网](https://git-scm.com/downloads)，根据你的系统（Windows/Mac/Linux）选择对应安装包，全程默认安装即可。
    
- 安装验证：安装完成后，任意位置右键打开「Git Bash Here」（Windows）/ 打开终端（Mac/Linux），输入以下命令，出现版本号即安装成功。
    

```Bash
git --version
```

### 2. 注册并登录GitHub账号

打开[GitHub官网](https://github.com/)，完成账号注册并登录，确保账号可正常访问。

### 3. 配置Git全局身份信息（核心必做）

这一步是让Git和你的GitHub账号绑定，否则无法正常提交代码，**用户名和邮箱必须和GitHub账号完全一致**。

1. 打开Git Bash/终端，依次执行以下2条命令：
    

```Bash
# 配置用户名（GitHub的账号昵称，不是登录邮箱）
git config --global user.name "你的GitHub用户名"

# 配置邮箱（GitHub账号绑定的注册邮箱）
git config --global user.email "你的GitHub注册邮箱"
```

2. 验证配置是否成功：
    

```Bash
git config --global --list
```

能看到你刚填写的用户名和邮箱，即配置完成。

---

## 二、核心分步操作（全程按顺序执行）

### 第一步：在GitHub创建远程空仓库（避坑关键）

1. 登录GitHub后，点击页面右上角的 **+ 号**，在下拉菜单中选择 **New repository**（新建仓库）。
    
2. 进入仓库创建页面，填写核心信息：
    
    1. **Repository name**：仓库名称（自定义，比如my-code-project，不能有中文和空格）
        
    2. **Description**（可选）：仓库描述，比如“我的项目核心代码”
        
    3. **Public/Private**：选择仓库权限，Public是公开所有人可见，Private是仅自己可见
        
    4. ❗❗ **新手绝对避坑**：页面底部的3个勾选框（Add a README file、Add .gitignore、Choose a license）**全部不要勾选**，直接创建空仓库！勾选后会导致本地和远程分支冲突，新手无法处理。
        
3. 点击页面底部 **Create repository**，完成远程仓库创建，会自动跳转到仓库主页。
    
4. 复制仓库地址：页面中会显示HTTPS和SSH两个地址，**新手优先选择HTTPS地址**，点击复制按钮保存（格式类似：`https://github.com/你的用户名/你的仓库名.git`）。
    

### 第二步：进入本地项目文件夹，打开Git终端

1. 打开你要上传的、包含代码的本地文件夹。
    
2. Windows系统：在文件夹空白处右键，选择 **Git Bash Here**，会直接打开对应路径的终端；
    

3. Mac/Linux系统：打开终端，输入`cd 你的文件夹绝对路径`（比如`cd /Users/xxx/Desktop/my-code`），回车进入文件夹。
    

### 第三步：初始化本地Git仓库

在打开的Git Bash/终端中，输入以下命令，回车执行：

```Bash
git init
```

- 执行效果：文件夹中会生成一个隐藏的`.git`目录，Git正式接管该文件夹的版本控制，这是本地仓库的核心。
    
- 补充：如果看不到隐藏文件，Windows在文件管理器顶部勾选「隐藏的项目」，Mac按`Command+Shift+.`即可显示。
    

### 第四步：（可选）配置.gitignore忽略文件

如果你的项目中有不需要上传的文件（比如依赖包node_modules、环境变量文件.env、IDE配置文件.idea等），先在项目根目录创建`.gitignore`文件，写入要忽略的文件/文件夹名称，再执行后续添加操作，避免无用文件被上传。

示例.gitignore文件内容：

```Plain
node_modules/
.env
.idea/
*.log
```

### 第五步：将本地所有文件添加到Git暂存区

在终端中输入以下命令，回车执行：

```Bash
git add .
```

- 命令说明：`.`代表当前目录下的所有文件和文件夹，会全部添加到Git暂存区；如果只想添加指定文件，把`.`换成文件名即可（比如`git add main.py`）。
    
- 验证（可选）：输入`git status`，查看文件状态，所有文件显示绿色即代表添加成功。
    

### 第六步：将暂存文件提交到本地Git仓库

在终端中输入以下命令，回车执行：

```Bash
git commit -m "feat: 初始化项目，提交核心代码"
```

- ❗ 关键说明：`-m` 后面必须跟双引号包裹的**提交说明**，必填项，不写会进入vim编辑界面，新手容易卡住；提交说明建议写清楚本次提交的内容，方便后续回溯。
    
- 执行后，终端会显示提交的文件数量、改动行数，即提交成功。
    

### 第七步：关联本地仓库与GitHub远程仓库

在终端中输入以下命令，把「你的仓库地址」换成第一步复制的HTTPS地址，回车执行：

```Bash
git remote add origin 你的GitHub仓库地址
```

- 示例：`git remote add origin https://github.com/zhangsan/my-code-project.git`
    
- 命令说明：`origin`是远程仓库的默认别名，行业通用，不建议修改。
    
- 验证（可选）：输入`git remote -v`，终端显示出你填写的仓库地址，即关联成功。
    

### 第八步：（避坑必做）分支名统一与远程内容拉取

1. GitHub当前默认主分支名为`main`，老教程的`master`已废弃，先执行以下命令统一本地分支名：
    

```Bash
git branch -M main
```

2. 如果你创建仓库时，不小心勾选了README等初始化文件，必须先执行以下命令，拉取远程内容合并，否则推送会报错；纯空仓库可跳过这一步：
    

```Bash
git pull origin main --rebase
```

### 第九步：将本地代码推送到GitHub远程仓库

在终端中输入以下命令，回车执行：

```Bash
git push -u origin main
```

- 关键说明：`-u`参数是设置上游分支，执行一次后，后续推送代码只需要输入`git push`即可，无需再加后面的参数。
    
- 账号验证：执行后，HTTPS方式会弹出账号输入框，先输入你的GitHub用户名，**密码不是GitHub登录密码，必须使用个人访问令牌PAT**（下方附详细获取教程）。
    

---

## 三、HTTPS推送必备：GitHub个人访问令牌PAT获取

2021年8月起，GitHub已全面禁用账号密码的HTTPS推送权限，密码处必须填写PAT令牌，否则会报403权限错误，获取步骤如下：

1. 打开GitHub，点击右上角头像 → 下拉选择 **Settings**。
    
2. 页面左侧拉到最底部，选择 **Developer settings**。
    
3. 左侧选择 **Personal access tokens** → 下拉选择 **Tokens (classic)**。
    
4. 点击右上角 **Generate new token** → 选择 **Generate new token (classic)**。
    
5. 填写配置：
    
    1. Note：令牌备注，比如“我的电脑代码推送”
        
    2. Expiration：令牌有效期，建议选No expiration（永久），也可自定义时间
        
    3. 权限勾选：必须勾选 **repo** 完整权限，其他可不用勾选
        
6. 拉到页面最底部，点击 **Generate token**，生成令牌。
    
7. ❗ 关键：生成后**立刻复制保存令牌**，页面刷新后就无法再次查看，丢失只能重新生成。
    
8. 推送时，密码输入框直接粘贴这个PAT令牌，回车即可完成推送。
    

---

## 四、最终验证与后续操作

1. 推送完成后，刷新你的GitHub仓库页面，就能看到本地文件夹里的所有代码文件，即上传成功。
    
2. 后续修改代码后，只需要重复3个核心命令，即可更新到GitHub：
    

```Bash
git add .
git commit -m "修改说明"
git push
```

---

## 五、高频报错与解决方案

|报错信息|核心原因|解决方案|
|---|---|---|
|fatal: remote origin already exists|本地仓库已经绑定过远程地址|执行`git remote rm origin`，再重新执行关联远程仓库的命令|
|failed to push some refs to xxx|本地和远程分支内容不一致，远程有本地没有的文件|先执行`git pull origin main --rebase`，再重新push|
|remote: Permission to xxx denied|账号权限错误，PAT令牌无效/权限不足|重新生成PAT令牌，确保勾选repo权限，推送时密码处粘贴新令牌|
|error: src refspec main does not match any|本地没有main分支，或commit未成功|先执行`git commit -m "提交说明"`，再执行`git branch -M main`，最后重新push|

---

## 六、长期使用推荐：SSH密钥配置（免密推送）

如果不想每次推送都输入PAT令牌，可配置SSH密钥，一次配置永久生效，核心步骤如下：

1. 本地生成SSH密钥对，终端执行：
    

```Bash
ssh-keygen -t ed25519 -C "你的GitHub注册邮箱"
```

全程回车，无需设置密码，默认生成在用户目录的`.ssh`文件夹中。

2. 打开生成的`id_ed25519.pub`文件，复制里面的全部内容（公钥）。
    
3. GitHub点击右上角头像 → Settings → 左侧选择**SSH and GPG keys** → 点击**New SSH key**。
    
4. Title填备注（比如“我的电脑”），Key处粘贴刚复制的公钥内容，点击Add SSH key。
    
5. 配置完成后，远程仓库地址换成SSH格式，重新关联即可，后续推送无需再输入账号密码。