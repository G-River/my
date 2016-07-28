title: hexo3.0 安装疑难
date: 2015-02-20 08:51:22
tags: [技术,hexo]
---
![](http://7xnwgz.com1.z0.glb.clouddn.com/2015-10-30%20003125.jpg)


系统版本：Ubuntu14  
node.js版本：

完全依照hexo官方文档说明安装3.1.1（https://hexo.io/docs/#Installation ），期间却出现了几处异常的问题。

1.安装hexo后，hexo generate报错
hexo generate Cannot find module 'escape-string-regexp'
会出现很多类似报错，都是由于缺少对应的module 。
解决办法：
npm install escape-string-regexp
    即把缺的包安装上，如果npm命令被墙，可以使用淘宝的国内镜像，安装cnpm，具体见 https://npm.taobao.org/

2.guang@guang:~/blog$ hexo d
ERROR Deployer not found: git
解决方法：安装hexo-deployer-git
npm install hexo-deployer-git --save

3.把公司电脑的sshkey加进自己的github，pull代码失败，提示
	ssh: connect to host github.com port 22: Connection timed out
	fatal: Could not read from remote repository.
	Please make sure you have the correct access rights
	and the repository exists.

推测原因是生成该key的邮箱不是github的账号邮箱

解决办法：在公司电脑中设置两个git账号，一个用于公司的gitlab库，另一个用于个人的github库。

添加第二个git账号的方法：  
        guang@guang:~/blog$ git config user.nama "git用户名"  
        guang@guang:~/blog$ git config user.email "git注册邮箱"  
        guang@guang:~/blog$ ssh-keygen -t rsa -f ~/.ssh/id_rsa_personal -C "git注册邮箱"  
回车运行   
        Generating public/private rsa key pair.
        Enter passphrase (empty for no passphrase):
        Enter same passphrase again:  
        Your identification has been saved in /home/guang/.ssh/id_rsa_personal.(将文件名另起一个新名字)  
        Your public key has been saved in /home/guang/.ssh/id_rsa_personal.pub.  
        The key fingerprint is:56:2c:80:a2:97:f8:c7:aa:2x:1a:83:57:50:21:d5:ec g******@****.com  
        The key's randomart image is:
        +--[ RSA 2048]----+
        |  ..o=.          |
        |  ..o o. .       |
        | o + .  . o      |
        |o +   E  o       |
        | o o    H        |
        |  . +  .         |
        |.  +             |
        |ooo              |
        |B+.              |
        +-----------------+
        guang@guang:~/blog$ cat ~/.ssh/id_rsa_personal.pub `
将新生成的key加进自己的github。  
然后在~/.ssh目录新建文件 config，作用是配置不同账户的key对应的服务器。内容如下：  
        #gitlab user(first@mail.com)   
        Host github.com
        HostName github.com
        User git
        IdentityFile C:/Users/username/.ssh/id_rsa
        #github user(second@mail.com)
        Host github-second  
        HostName github.com   
        User git   IdentityFile C:/Users/username/.ssh/id_rsa_personal`

4.为了在多台电脑上能更新blog，把blog文件夹下的文件都同步到另一个git库中。但是push到远程库上的主题文件夹是空的。  
原因：主题文件夹是由git管理的，要想同步到远程库，要把主题文件夹下的.git目录删除，才能将整个blog的hexo文件同步到远程库，包括主题文件

5.由于本机用了两个git账号，username不同，想分别给两个不同的远程库提交代码。提交其中一个git远程库的时候发现username是另一个账户的。
原因：hexo d命令向远程库提交代码时，使用的是全局的git变量中的username和email，（在~/.gitconfig中配置），而我在全局变量中配置的username和email是另一个账户的。所以名字错了。
git在push代码时，优先使用local配置的email，如果local没有配置，则使用global配置。
    			git config --global -e 查看全局配置，相应文件在~/.gitconfig
    			git config --local -e 查看局部配置，相应文件在项目路径下的.git/config
    			git config --global user.email "xxx@xxx.com" 配置全局的邮箱
    			git config --local user.name "***" 配置全局的username
    			要配置局部的姓名和邮箱，把全局的 --global 改为 --local即可
