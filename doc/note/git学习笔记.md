# git学习笔记

<!-- TOC -->

- [git学习笔记](#git学习笔记)
  - [1. git 简介](#1-git-简介)
  - [2. git 命令](#2-git-命令)
    - [2.1. git submodule](#21-git-submodule)
      - [2.1.1. 添加子模块](#211-添加子模块)
      - [2.1.2. 初始化与更新子模块](#212-初始化与更新子模块)
    - [2.2. git clone](#22-git-clone)
    - [2.3. git commit](#23-git-commit)
      - [2.3.1. git commit语法](#231-git-commit语法)
      - [2.3.2. git commit多行备注](#232-git-commit多行备注)
      - [2.3.3. git commit递交的推荐格式](#233-git-commit递交的推荐格式)
    - [2.4. git增删改](#24-git增删改)
    - [2.5. reset命令](#25-reset命令)
      - [2.5.1. reset的语法格式与基本用法](#251-reset的语法格式与基本用法)
      - [2.5.2. reset的样例](#252-reset的样例)
        - [2.5.2.1. 拉取代码到最新版本](#2521-拉取代码到最新版本)
        - [2.5.2.2. 拉取代码到指定版本](#2522-拉取代码到指定版本)
  - [3. git配置SSH](#3-git配置ssh)
    - [3.1. git配置SSH时遇到的问题](#31-git配置ssh时遇到的问题)
      - [3.1.1. clone时反复要求输入密码，无论输入的密码是否正确都会报错](#311-clone时反复要求输入密码无论输入的密码是否正确都会报错)
      - [3.1.2. 读取密钥文件报权限错误](#312-读取密钥文件报权限错误)
  - [4. Q&A](#4-qa)
    - [4.1. remote: Support for password authentication was removed on August 13, 2021. Please use a perso](#41-remote-support-for-password-authentication-was-removed-on-august-13-2021-please-use-a-perso)
      - [4.1.1. 生成个人令牌](#411-生成个人令牌)
      - [4.1.2. 使用个人令牌](#412-使用个人令牌)
    - [4.2. clone时报错：Failed to connect to 127.0.0.1 port 4780 after 2038 ms: Connection refused](#42-clone时报错failed-to-connect-to-127001-port-4780-after-2038-ms-connection-refused)

<!-- /TOC -->

## 1. git 简介

## 2. git 命令

### 2.1. git submodule

#### 2.1.1. 添加子模块

首先进入到对应的项目目录下，打开 git bash，执行以下命令即可。

```bash
git submodule add <submodule_url> <本地路径（缺省则默认为当前路径）> [-b 分支名（指明分支名）] [--force]

// 推送到远程仓库
git commit -am "add leg"  
git push
```

<!--![PNG-添加子模块命令执行效果图Base64][PNG-添加子模块命令执行效果图]-->
![PNG-添加子模块命令执行效果图Base64](../pic/git_learn/PNG-添加子模块命令执行效果图.png)

例子：

```bash
git clone https://git.oschina.net/gaofeifps/body.git
cd body
git submodule add https://git.oschina.net/gaofeifps/leg.git
```

这时查看下状态会多两个文件（git status命令）：一个是子模块、另一个是 git submodule 的配置文件。

<!--![PNG-添加子模块后多出了两个文件Base64][PNG-添加子模块后多出了两个文件]-->
![PNG-添加子模块后多出了两个文件Base64](../pic/git_learn/PNG-添加子模块后多出了两个文件.jfif)

> 注：`.gitmodules` 文件中会记录子模块的名称与源代码url

#### 2.1.2. 初始化与更新子模块

在对应的项目目录下，执行以下命令即可：

```bash
git submodule init            //初始化本地配置文件
git submodule update          //从该项目中抓取所有数据并检出父项目中列出的合适的递交。

//或使用命令：
git clone --recurse-submodules <主项目地址>
```

### 2.2. git clone

克隆指定分支：

```bash
// 未指定远程分支名，默认 clone master 分支。
git clone -b 远程分支名 远程仓库地址

// 指定存放目录
git clone -b 远程分支名 远程仓库地址 存放目录

// 如果已经 clone 了其他分支，可以采用以下方法切换分支
git branch -a   // 1. 先查看当前远端分支情况
git checkout origin/xxx     // 2. 选择远端 xxx 分支（origin 为远程仓库别名）
git branch xxx              // 3. 创建本地 xx 分支
git checkout xxx            // 4. 选择刚创建的分支
```

### 2.3. git commit

#### 2.3.1. git commit语法

语法：

```bash
// 递交暂存区内容到本地仓库（[message] 是备注信息）。
git commit -m [message]

// 递交暂存区指定的文件到本地仓库。
git commit [file1] [file2] ... -m [message]

// -a 参数可以直接将暂存区中被修改的文件全部递交，无需使用 git add
git commit -am [message]
```

#### 2.3.2. git commit多行备注

```bash
// 方法1：使用多个 `-m` 选项：`commit` 命令会把多个 `-m` 选项的值串联成一个单独的段落，每一 `-m` 为单独一行
git co -am "line1" -m "line2" -m "line3"

// 方法2：先输入一个引号 "，然后换行，即可进行多行编辑，最后再输入一个引号 "，然后换行，即可完成编辑
git co -am "line1
>
> line2"
```

#### 2.3.3. git commit递交的推荐格式

推荐格式如下：

```default
<type>: <subject>
//空一行
<body>
```

1. `type`：用于说明本次提交的类别，规定为以下几种：
   - `feat`：新功能
   - `fix`：修复错误
   - `refactor`：重构代码
   - `style`：调整代码格式（不影响代码运行）
   - `docs`：更新文档
2. `subject`：递交的简要说明
   1. 规定1：简要说明的内容不大于90个字符
   2. 规定2：`subject` 之前要有一个空格。在浏览有很多条提交信息的历史记录时，多出的这个空格，可以更容易的查看提交说明。
3. `body`：本次递交的详细描述，可以分多行。
   1. 规定1：`body` 前有一个空行，这是为了在浏览日志中只显示提交的简要说明。
   2. 规定2：如果 `type` 是 `fix` (修复错误)，必须在 `body` 中记录以下信息：
      1. 重现错误的步骤
      2. 预期应有的行为
      3. 观察到的错误行为

### 2.4. git增删改

增加新文件:

```bash
git add 新文件
git commit 新文件 -m "递交信息"
git push origin
```

删除文件：

```bash
rm -rf 文件名
git rm 文件名
git commit -m "递交信息"
git push origin
```

修改文件：

```bash
git commit 修改的文件 -m "递交信息"
git push origin
```

### 2.5. reset命令

#### 2.5.1. reset的语法格式与基本用法

*reset* 命令一般结合 *pull* 命令来使用，设置当前节点到指定版本，然后把该版本的代码拉下来，从而达到回退本地代码或更新本地代码的效果。

语法格式：

```bash
# --mixed 为默认，可以不用带该参数，用于重置暂存区的文件与上一次的提交(commit)保持一致，工作区文件内容保持不变。
git reset [--soft | --mixed | --hard] [HEAD]

#样例：
## --mixed 选项
git reset HEAD^            # 回退所有内容到上一个版本  
git reset HEAD^ hello.php  # 回退 hello.php 文件的版本到上一个版本  
git reset  052e            # 回退到指定版本

## --soft 选项
git reset --soft HEAD~3    # 回退上上上一个版本 

## --hard 选项：撤销工作区中所有未提交的修改内容，将暂存区与工作区都回到上一次版本，并删除之前的所有信息提交
git reset --hard HEAD~3    # 回退上上上一个版本  
git reset –hard bae128     # 回退到某个版本回退点之前的所有信息。 
git reset --hard origin/master    # 将本地的状态回退到和远程的一样 
```

> 注1：谨慎使用 –hard 参数，它会删除回退点之前的所有信息。
>
> 注2：HEAD 说明：
>
> 1. HEAD 或 HEAD~0 表示当前版本
> 2. HEAD^ 或 HEAD~1 表示上一个版本
> 3. HEAD^^ 或 HEAD~2 表示上上个版本
> 4. 以此类推...

#### 2.5.2. reset的样例

##### 2.5.2.1. 拉取代码到最新版本

```bash
git fetch -f
git reset --hard origin/master(或 origin/HEAD)
git pull
```

##### 2.5.2.2. 拉取代码到指定版本

```bash
git fetch -f
git reset --hard origin/版本号
git pull
```

## 3. git配置SSH

1. 在本地生成 SSH key：`ssh-keygen -b 4096 -t rsa -f %USERPROFILE%\.ssh\id_rsa -q -P ""` 或 `ssh-keygen -t ed25519 -C "你的邮箱/git的用户名"`。（建议生成 ed25519 类型的key，因为它比 rsa 类型更安全。并且高版本的git 移除了对 rsa 类型的 SSH Key 的支持，采用该类型的 key 会导致 git clone时反复要求输入密码，并且输入任何密码都会提示错误。）
2. 在 git 账户中添加 SSH Key：登录 git -> 点击头像 -> 编辑个人资料 -> SSH 密钥 -> 将密钥复制上去即可。

### 3.1. git配置SSH时遇到的问题

#### 3.1.1. clone时反复要求输入密码，无论输入的密码是否正确都会报错

报的错误：

```txt
permission denied (publickey,keyboard-interactive).
```

解决办法：将密钥换成 `ed25519` 类型的密钥即可。

#### 3.1.2. 读取密钥文件报权限错误

报错信息如下：

```txt
permissions 0644 for '/home/arnicedeng/.ssh/id_ed25519' are too open. it is required that your private key files are not accessible by others. this private key will be ignored.
```

以上错误的意思是：私钥文件不能被其他人所访问。私钥是访问linux服务器的凭证，如果被别人获取到，就可能对服务器安全造成影响，这可能也就是这个问题的初衷。

根据提示，将私钥文件的权限改成 600 即可。使用命令：

```txt
chmod 600 私钥文件
```

## 4. Q&A

### 4.1. remote: Support for password authentication was removed on August 13, 2021. Please use a perso

意思是：自从21年8月13后不再支持用户名密码的方式验证了，需要创建个人访问令牌(personal access token)。

#### 4.1.1. 生成个人令牌

![PNG-生成个人令牌步骤1Base64](../pic/git_learn/git_person_token_setp1.png)

![PNG-生成个人令牌步骤2Base64](../pic/git_learn/git_person_token_setp2.png)

![PNG-生成个人令牌步骤3Base64](../pic/git_learn/git_person_token_setp3.png)

下面的 *scopes* 没头绪就先全选上：

![PNG-生成个人令牌步骤4Base64](../pic/git_learn/git_person_token_setp4.png)

#### 4.1.2. 使用个人令牌

修改现有项目的 *remote*

```bash
git remote set-url origin  https://<your_token>@github.com/<USERNAME>/<REPO>.git
```

### 4.2. clone时报错：Failed to connect to 127.0.0.1 port 4780 after 2038 ms: Connection refused

完整的报错：

```txt
fatal: unable to access '[git仓库链接]': Failed to connect to 127.0.0.1 port 4780 after 2038 ms: Connection refused
```

原因：一般是配置了全局的 *http* 代理。可以使用 `git config --list` 命令查看 *git* 的 *http* 代理。

解决办法：去除全局代理的配置：

```bash
git config --global --unset http.proxy  
git config --global --unset https.proxy
```

<!-- 以下是图片资源 
[PNG-添加子模块后多出了两个文件]:data:image/jpeg;base64,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

[PNG-添加子模块命令执行效果图]:data:image/png;base64,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

-->
