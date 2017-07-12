最近发现 github 改版了，已没有像原来的 Launch automatic page generator 这样的按钮等，所以我对我的文章也进行了修正，对于新版来说，步骤更加简单了。欢迎享用。

－－－－－－华丽丽的分割线，以下是在原版的基础上的修正版－－－－－－－

学了前端小半年，如今写了个自己的网页想要去应聘，却发现部署很麻烦，部署到阿里云之类，买域名啊啥的还要收费，说贵也不贵，但我就是傲娇~

google一下了解到Github有一个Github pages的功能可以搭建博客或者托管网页，而且免费耶，搜了下教程，猛地一看感觉步骤也不是很麻烦，所以就用这个了！

教程一大堆，却没有几个能看懂的，问题一：90%的都在讲解如何搭建博客，和我想要将自己的网页部署到上面还是有点区别的。问题二：所有的教程都用到了Git，而我只知道Git是一个开源的分布式版本控制系统。完全不知道命令行是什么鬼，只能照猫画虎的小白来说，那些教程只能帮我到桥头，但想要成功过河，还需要深夜里的一包特浓咖啡。
<br />
<br />

开始教程之前的准备工作：<br />
1、需要你自己写的网页文件。
![Alt text](/assets/github1.png)<br />

2、注册Github。<br />
3、下载安装git。下载地址https://git-scm.com/downloads
<br />
<br />
教程开始：（以下出现的test指你的网页名或者你想起的一切名字）
**步骤一：**登录到Github上，新建一个repo，命名为test，勾选 initialize this repository with a README，点击create repository。
![Alt text](/assets/github2.png)
![Alt text](/assets/github3.png)

**步骤二：**打开settings，有一个Github Pages 的设置，点击 source 中的本来的 None ，使其变成 master 分支，也就是作为部署github pages 的分支，然后点击 save。
![Alt text](/assets/github4.png)
![Alt text](/assets/github5.png)

**步骤三：**页面刷新之后，再看 github pages 设置框处，多了一行网址，就是你的 github pages 的网址了。
![Alt text](/assets/github6.png)

点击这个链接<br />
![Alt text](/assets/github7.png)

哇塞，一个 test。<br />
![Alt text](/assets/github8.jpg)

至此以上，github上要处理的操作告一段落，该上Git了！

**步骤四 ：**打开此电脑，选择一个盘，比如 f 盘，右键空白处点击 git bash here。
![Alt text](/assets/github9.png)

**步骤五：**输入如下命令，用来在 f 盘创建 test 文件放你的github上的test repository，克隆test repository到 test 文件中。<br />
![Alt text](/assets/github10.png)

这个时候你的 f 盘，就会多一个 test 文件，打开它，
![Alt text](/assets/github11.png)

会看到一个 README.md 的文件，这个文件是从哪来的呢？追溯到gihub上，你会发现 README 文件是来自 master 分支。
![Alt text](/assets/github12.jpg)

**步骤六：**  将自己的网页文件复制粘贴至 f 盘的 test 文件中
![Alt text](/assets/github13.png)

**步骤七：** 执行如下命令
这里应该是  `cd test/`<br />
![Alt text](/assets/github14.png)
![Alt text](/assets/github15.png)

解释一下上面的命令：首先输入  `git status`   列出当前目录所有还没有被git管理的文件和被git管理且被修改但还未提交(`git commit`)的文件，也就是所有改动文件，红色字体标出。

然后输入 `git add .`  (有个点) 表示添加当前目录下的所有文件和子目录，

然后 再输入一次 `git status` 如果看见文件都变绿了 ，那么就代表 它们已经准备好了被提交（`git commit`）。

**步骤八：**输入如下命令，将你的文件上传至远程 master 分支
![Alt text](/assets/github16.png)
![Alt text](/assets/github17.png)

输入最后一行 `git push`，按回车，等一会，会有弹出框让你输入你的 github 账号和密码。
![Alt text](/assets/github18.png)
![Alt text](/assets/github19.png)

ok之后耐心等待。
当出现像下图中的语句之后，你已经完成了99%。
![Alt text](/assets/github20.png)

**最后一步：**打开浏览器输入给你的网址加上你上传的 html 文件名 test.html，然后重重的按下回车。
![Alt text](/assets/github21.png)
![Alt text](/assets/github22.png)
