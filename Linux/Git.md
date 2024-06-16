# Git配置

官网下载：https://git-scm.com/downloads

镜像下载：http://npm.taobao.org/mirrors/git-for-windows/

安装到D盘，如果对话框下面有勾选new options，取消掉才能看到自定义安装路径

无脑下一步

安装成功之后配置用户名and邮箱：

```perl
git config --global user.name ${username}
git config --global user.email ${emailAddress}
```

生成[公钥](https://so.csdn.net/so/search?q=公钥&spm=1001.2101.3001.7020)私钥，连按三次回车：

```bash
ssh-keygen
```

生成的公私钥在C:/Users/username/.ssh，复制公钥到github上

至此，配置成功

# 配置仓库

本地新建空目录，添加远程仓库：

```perl
mkdir directory
cd directory
git init
git remote add origin git@github.com:NepKisa/leetcode.git
```

origin是远程源的名称

如果不是新仓库，每次需要(pull后面的分支名视情况而定)

```perl
git pull originName 分支名
```

添加文件，.表示当前目录下的所有文件，可以只写需要添加的文件名：

```perl
git add .
git commit -m "change message"
git push originName 本地分支名:远程分支名
```

去远程检查是否提交成功

有过一次pull 和 push之后可以执行以下命令指定默认分支，本地分支名下次就可以直接git pull和git push了

```perl
git branch --set-upstream-to origin/远程分支名 
```

原文链接：https://blog.csdn.net/qq_45484072/article/details/129781556

