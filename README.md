# English_note

本项目的介绍文件



# 一、Git SSH 密钥配置

安全注意事项:

1. **私钥安全**：不要共享私钥文件，定期更换密钥
2. **密码保护**：为私钥设置强密码
3. **密钥轮换**：建议每年更换一次SSH密钥
4. **审计密钥**：定期检查平台上的SSH密钥列表，删除无用密钥

## 1.前置检查

### 1.1 查看现有密钥

```bash
ls -al ~/.ssh
```

**可能存在的密钥文件**：

- `id_rsa`（私钥）
- `id_rsa.pub`（公钥）
- `id_ed25519`（Ed25519算法密钥）


## 2.生成新密钥对

### 2.1 生成RSA密钥（适用于大多数场景）

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

### 2.2 生成Ed25519密钥（更高安全性）

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

### 2.3 生成时的选项说明

- `-t`：指定密钥类型（rsa/ed25519）
- `-b`：指定密钥长度（RSA建议4096）
- `-C`：添加注释（通常使用邮箱）
- `-f`：指定密钥保存路径（默认`~/.ssh/id_rsa`）


## 3.各系统配置步骤

### 3.1 macOS配置

```bash
# 1. 生成密钥（同上）
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# 2. 添加到钥匙串（避免重复输入密码）
ssh-add --apple-use-keychain ~/.ssh/id_rsa
```

### 3.2 Windows配置

```bash
# 1. 使用Git Bash生成密钥
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# 2. 启动SSH Agent并添加私钥
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_rsa
```

**Windows 10/11用户**：  
可通过控制面板启用OpenSSH客户端：  
`控制面板 > 程序 > 启用或关闭Windows功能 > OpenSSH客户端`

### 3.3 Linux配置

```bash
# 1. 生成密钥
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# 2. 自动加载密钥（编辑~/.bashrc或~/.zshrc）
echo 'eval "$(ssh-agent -s)"' >> ~/.bashrc
echo 'ssh-add ~/.ssh/id_rsa' >> ~/.bashrc
source ~/.bashrc
```


## 4.添加公钥到Git平台

### 4.1 复制公钥内容

```bash
# RSA公钥
cat ~/.ssh/id_rsa.pub

# Ed25519公钥
cat ~/.ssh/id_ed25519.pub
```

### 4.2 在平台添加公钥

1. 登录GitHub/Gitee/GitLab等平台
2. 进入`Settings > SSH and GPG keys`
3. 点击`New SSH key`
4. 粘贴公钥内容并设置标题


## 5.测试SSH连接

```bash
# 测试GitHub连接
ssh -T git@github.com

# 测试Gitee连接
ssh -T git@gitee.com
```

**成功响应示例**：

```
Hi username! You've successfully authenticated...
```


## 6.多账户配置方法

### 6.1 生成多个密钥

```bash
# 工作账户密钥
ssh-keygen -t rsa -b 4096 -C "work_email@example.com" -f ~/.ssh/id_rsa_work

# 个人账户密钥
ssh-keygen -t rsa -b 4096 -C "personal_email@example.com" -f ~/.ssh/id_rsa_personal
```

### 6.2 配置SSH config文件

```bash
nano ~/.ssh/config
```

**配置示例**：

```conf
# 默认账户
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa

# 工作账户
Host github-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_work

# 个人账户
Host github-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_personal
```

### 6.3 使用指定账户克隆仓库

```bash
# 使用工作账户克隆
git clone git@github-work:your_username/repo.git

# 使用个人账户克隆
git clone git@github-personal:your_username/repo.git
```


## 7.常见问题排查

### 7.1 连接被拒绝

```bash
# 查看详细连接日志
ssh -vT git@github.com
```

### 7.2 权限问题

```bash
# 确保私钥文件权限正确
chmod 600 ~/.ssh/id_rsa
```

### 7.3 清除缓存的密钥

```bash
# 删除已知主机缓存
ssh-keygen -R github.com
```





# 二、推送文件

## 1.首次推送

以下是将文件正确推送到Git远程仓库的详细步骤和注意事项：

![image-20250509184651602](./C:/Users/Mcjy/AppData/Roaming/Typora/draftsRecover/images/image-20250509184651602.png)



### 1）初始化本地仓库

在项目根目录下执行：

```bash
git init
```

这会创建一个隐藏的 `.git` 文件夹，用于管理版本控制。


### 2）添加文件到暂存区

添加单个文件：

```bash
git add filename.txt
```

添加所有文件（推荐）：

```bash
git add .
```

检查状态：

```bash
git status  # 查看哪些文件被添加/修改
```


### 3）提交到本地仓库

```bash
git commit -m "提交描述，例如：添加新功能或修复bug"
```

- **提交信息**应清晰描述本次修改的目的。


### 4）关联远程仓库

首次关联：

```bash
git remote add origin git@github.com:用户名/仓库名.git  # SSH方式
# 或
git remote add origin https://github.com/用户名/仓库名.git  # HTTPS方式
```

检查关联：

```bash
git remote -v
```


### 5）推送代码到远程仓库

首次推送（设置上游分支）：

```bash
git push -u origin main  # 推送至main分支（GitHub默认）
# 或
git push -u origin master  # 推送至master分支（传统默认）
```

- **`-u` 参数**：建立本地分支与远程分支的关联，后续可直接用 `git push`。

## 2.日常推送


### 1）添加文件到暂存区

添加单个文件：

```bash
git add filename.txt
```

添加所有文件（推荐）：

```bash
git add .
```

检查状态：

```bash
git status  # 查看哪些文件被添加/修改
```


### 2）提交到本地仓库

```bash
git commit -m "提交描述，例如：添加新功能或修复bug"
```

- **提交信息**应清晰描述本次修改的目的。


### 3）关联远程仓库

检查关联

```bash
git remote -v   
```


### 4）推送代码到远程仓库

```bash
git push  # 简化命令，需已设置上游分支
```

### 5）强制推送

~~~bash
git add .
git push -f origin main
~~~



## 3.常见问题及解决方案

### 1）远程仓库已存在未合并的更改

```bash
git pull origin main  # 先拉取远程更新，合并后再推送
```

### 2）忘记关联远程仓库

```bash
git remote add origin <远程仓库URL>
```

### 3）权限问题（SSH/HTTPS）

- **SSH**：需提前配置公钥到GitHub账户。
- **HTTPS**：推送时需输入GitHub用户名和密码（或访问令牌）。



## 5.分支操作

### 1）创建并切换分支

```bash
git checkout -b new-feature  # 创建并切换到new-feature分支
```

### 2）推送分支到远程

```bash
git push -u origin new-feature
```


### 3）总结流程

1. **初始化仓库**：`git init`
2. **添加文件**：`git add .`
3. **提交本地**：`git commit -m "描述"`
4. **关联远程**：`git remote add origin <URL>`
5. **推送代码**：`git push -u origin main`



## 6.Git忽略文件`.gitignore`

`.gitignore` 文件用于指定 Git 版本控制系统应忽略的文件和目录。这些文件不会被跟踪、暂存或提交到仓库中。一般在项目根目录下创建：

```bash
touch .gitignore
```

**注意事项**：

1. 尽早创建 `.gitignore` 文件
2. 将 `.gitignore` 本身加入版本控制
3. 定期检查是否有新文件需要忽略
4. 使用全局 `.gitignore` 文件处理用户特定文件（如编辑器配置）

### 1）忽略特定文件

```gitignore
# 忽略单个文件
secrets.txt
config.ini

# 忽略特定类型文件
*.log
*.tmp
```

### 2）忽略目录

```gitignore
# 忽略整个目录（包括子目录）
node_modules/
.idea/
.vscode/
```

### 3）精确控制忽略范围

| 规则       | 作用范围                     | 示例说明                    |
| ---------- | ---------------------------- | --------------------------- |
| `logs/`    | 忽略所有路径下的 logs 目录   | 忽略 `logs/` 和 `src/logs/` |
| `/logs/`   | 仅忽略根目录下的 logs 目录   | 仅忽略 `logs/`              |
| `**/logs/` | 明确表示忽略所有路径下的目录 | 同 `logs/`                  |

### 4）排除例外

```gitignore
# 忽略 public 下所有 .txt 文件
public/*.txt

# 但不忽略 public/readme.txt
!public/readme.txt
```

### 5）系统文件

```gitignore
# macOS
.DS_Store

# Windows
Thumbs.db
```

### 6）特殊注意事项

1. **已跟踪文件的处理**：

   ```bash
   git rm --cached filename  # 停止跟踪单个文件
   git rm --cached -r dir/   # 停止跟踪整个目录
   ```

2. **验证忽略规则**：

   ```bash
   git check-ignore -v filename
   ```

3. **文件位置**：`.gitignore` 应放在项目根目录，也可以在不同子目录中添加额外的忽略文件



