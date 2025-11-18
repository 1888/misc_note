# git operations
## git push -- 将本地仓库的提交上传到远程仓库
理解git push之前，介绍几个关键点：
* 本地分支：你在自己电脑上工作的分支。
* 远程仓库：托管在服务器（如 GitHub, GitLab, Gitee）上的仓库。
* 远程分支：远程仓库上的分支，通常是 origin/<分支名> 的形式，这是你本地分支对应的“镜像”。
* 上游分支：一个本地分支可以关联一个远程分支作为其“上游”。设置了上游后，可以直接使用 git push 而不需要指定参数。

基本语法：
> git push <远程仓库名> <本地分支名>:<远程分支名>

常用示例：
1. git push origin main
   
   将本地的 main 分支推送到远程 origin 的 main 分支

2. git push origin bugfix-123:hotfix-123
   
   将本地的 bugfix-123 分支推送到远程 origin仓库的hotfix-123分支

3. git push --set-upstream origin main
   
   将本地的main分支对应的上游分支设为远程仓库的main分支。可以通过git branch -vv 可以查看所有本地分支及其关联的上游分支
   ```bash
   cityday@ubuntu24:~/work/misc_note$ git branch -vv
   main 6c47499 [origin/main] add ssh method for Invalid username or token
   ```

4. git push
   
   将本地分支推送到对应的上游分支。


## error handling
1. remote: Invalid username or token. Password authentication is not supported for Git operations.
目前已不支援密码方式登陆。
### method1. 需要通过github网站生产token，再将token作为密码输入才行
```
cityday@ubuntu24:~/work/misc_note$ git push
Username for 'https://github.com': 1888
Password for 'https://1888@github.com': your_token
```
个人习惯将token保存到Data/01.work/99.personal_info/github_personal_access_token.txt. method1 在过一段时间时候，需要再次输入token。

### method2. 添加ssh公钥到github
#### 步骤 1: 检查并生成 SSH 密钥
在 Ubuntu 终端中，检查是否已有密钥：
```bash
ls -al ~/.ssh/id_*.pub
```
如果看到 id_rsa.pub 或 id_ed25519.pub 等文件，说明已有密钥，可跳过生成步骤。
如果没有，生成一个新的 SSH 密钥（推荐使用 Ed25519 算法）：
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```
* 按回车接受默认的保存路径（~/.ssh/id_ed25519）。
* 设置一个安全的密码（可选，但推荐）。

#### 步骤 2: 将 SSH 公钥添加到 GitHub
1. 查看并复制你的公钥内容：
```bash
cat ~/.ssh/id_ed25519.pub
```
2. 复制全部内容，形如：ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... your_email@example.com
3. 登录 GitHub -> Settings -> SSH and GPG keys。
4. 点击 New SSH key。
5. Title: 起个名字（如 Ubuntu Desktop）。
6. Key type: 保持默认 Authentication Key。
7. Key: 粘贴你刚才复制的公钥内容。
8. 点击 Add SSH key。

#### 步骤 3: 修改远程仓库地址为 SSH 格式
这是关键一步！如果你之前是用 https:// 链接克隆的仓库，需要将其改为 git@ 开头的 SSH 地址。
1. 查看当前远程地址：
```bash
git remote -v
# 输出可能为：
# origin  https://github.com/username/repo.git (fetch)
# origin  https://github.com/username/repo.git (push)
```
2. 将 URL 改为 SSH 格式：
```bash
git remote set-url origin git@github.com:username/repo.git
```
现在再执行 git push 等操作，就会使用 SSH 密钥进行认证，不会再提示用户名和密码错误。

## 2. 如果看执行某个git命令的debug信息.
可以在git 命令前加下面的设置:
```
GIT_CURL_VERBOSE=1 GIT_TRACE=1 git xxx
e.g., GIT_CURL_VERBOSE=1 GIT_TRACE=1 git push
```


