# git operations
## 1. remote: Invalid username or token. Password authentication is not supported for Git operations.
目前已不支援密码方式登陆。需要通过github网站生产token，再将token作为密码输入才行  
```
cityday@ubuntu24:~/work/misc_note$ git push
Username for 'https://github.com': 1888
Password for 'https://1888@github.com': your_token
```
个人习惯将token保存到Data/01.work/99.personal_info/github_personal_access_token.txt

##2. 如果看执行某个git命令的debug信息.
可以在git 命令前加下面的设置:
GIT_CURL_VERBOSE=1 GIT_TRACE=1 git xxx
e.g., GIT_CURL_VERBOSE=1 GIT_TRACE=1 git push


