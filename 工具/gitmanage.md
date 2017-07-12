在此假设你已经在 github 上创建好了一个项目，像这样
![Alt text](/assets/git1.png)
并且你已经完成了自己的项目代码，同时你也已经安装了 git，然后 let's start.

首先，建一个文件夹比如文中演示的是 微信小程序 文件夹，然后打开的你的终端，定位到该文件夹，
![Alt text](/assets/git2.png)

然后输入命令:  `git init`
![Alt text](/assets/git3.png)

然后配置 ssh , 输入：`ssh-keygen -t rsa -C "jiayi_li10@163.com" `(邮箱替换成你登录github的邮箱)
![Alt text](/assets/git4.png)

这个地方请注意，它会在你选择的路径下上生成 ssh key，如果你直接点击回车，会在默认路径下创建 ssh 。如果你有多个项目，有工作的，有自己玩的，那么请配置不同的路径，或者一个路径换个文件名，我就用：/Users/lijiayi/.ssh/id_test_rsa 作为演示。输入路径之后点击回车。
![Alt text](/assets/git5.png)

这个地方是要你输入密码，直接回车则是不设置密码。直接回车就可以。然后会让你重复密码，也是直接回车。
![Alt text](/assets/git6.png)

当你出现如图所示，就代表 ssh 已经生成了。
这个执行命令：`pbcopy < ~/.ssh/id_test_rsa.pub`   这个的作用是将你的 ssh 代码复制到剪贴板。
![Alt text](/assets/git7.png)

现在，咱们在重新回到 github 页面，需要将刚才生成的 ssh 配置到 github 里。点击你的呆萌头像：
![Alt text](/assets/git8.png)

然后点击 settings 设置：
![Alt text](/assets/git9.png)

点击配置 ssh：
![Alt text](/assets/git10.png)

点击新建 New SSH key:
![Alt text](/assets/git11.png)

直接 Crl＋v 将刚才你已经复制在剪贴板里的 ssh 复制到 key input 里面，title 你随意起喽。然后点击 Add SSH key.
![Alt text](/assets/git12.png)

现在，咱们再打开终端，验证一下是否添加ssh成功了，输入命令： `ssh -T git@github.com`
![Alt text](/assets/git13.png)

出现如上图的句子，你就起来跳个舞。倘若是类似如下的句子：

```
The authenticity of host 'git.net (116.211.167.152)' can't be established.
ECDSA key fingerprint is SHA256:FQGC9Kn/eye1W8icdBgrQp+KkGYoFgbVr17bmjey0Wc.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'git.oschina.net,116.211.167.152' (ECDSA) to the list of known hosts.
Permission denied (publickey).
```
或者permission denied,你就再执行命令：`ssh-add ~/.ssh/id_test_rsa`
再次输入 `ssh -T git@github.com` 如果提示成功了，咱们就继续，如果没有成功，你就 google 一下报的什么错误。
![Alt text](/assets/git14.png)

当你successfully之后，咱们就在 git config 里设置一下你的 github 登录名以及登陆邮箱，执行以下两个命令：
`git config --global user.name "your name"`
`git config --global user.email "your_email@youremail.com"`
![Alt text](/assets/git15.png)

现在咱们就可以上传代码啦！！
将你的项目代码拉到这个文件夹，执行命令，`git status`
![Alt text](/assets/git16.png)

这个时候你就会看到所有的改动，然后执行 `git add .`    (有个点哦，这个点表示更改所有的改动)
then 执行命令 `git commit -m "第一次更新"`
![Alt text](/assets/git17.png)

然后执行命令：`git remote add origin git@github.com:用户名/项目名.git` （后面的地址从下面标注的地方可以找到）
![Alt text](/assets/git18.png)

最后执行命令：`git push -f origin master`
现在 回到你的 github 页面，然后刷新该项目页，哇色，这是什么
![Alt text](/assets/git19.png)


<br>
<br>
一些有可能遇到的问题以及参考网站： 
＊mac多个git账户配置：http://www.jianshu.com/p/fbbf6efb50ba </br>
＊cannot push to github ,keeps saying need merge: http://stackoverflow.com/questions/10298291/cannot-push-to-github-keeps-saying-need-merge  </br>
＊删除github远程分支：https://my.oschina.net/tsingxu/blog/84601  </br>