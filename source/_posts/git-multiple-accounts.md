title: Git 配置多账号
date: 2015-09-15 23:39:24
category:
	- Git
tags:
	- git
	- github
---

由于想在Github上 试下branch如何用，不想在现有账号试，就重新建了个账号。问题来了，怎么在本地配置呢，因为本地已经配了一个账号了，网上搜了半天，各种出状况。这里记录下，其实很简单。

<!--more-->

### 生成SSH Keys
账号A：已经配置过 ssh key 已经存在，默认名字 ~/.ssh/id_rsa

账号B：新添加账号，照例先生成ssh key

~~~
jacob@JacobdeMacBook-Pro:~/.ssh$ ssh-keygen -t rsa -C "yanjunjie110@gmail.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/jacob/.ssh/id_rsa): id_rsa2
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in id_rsa2.
Your public key has been saved in id_rsa2.pub.
#第一个git项目账号
The key fingerprint is:
38:85:3a:24:bd:06:10:b0:e7:c7:05:56:51:d9:1d:b5 yanjunjie110@gmail.com
The key's randomart image is:
+--[ RSA 2048]----+
|=.  o.oo.o ..o.  |
| o o . .. . .  . |
|. + o o .     E  |
| o = + o         |
|  . B o S        |
|   o . .         |
|                 |
|                 |
|                 |
#第一个git项目账号
+-----------------+
~~~

这里保存为 ~/.ssh/id_rsa2

### SSH Config
这个是多账号配置的关键，在 ~/.ssh 目录下查看是否有 config 文件，如果没有则新建一个。配置内容如下：

~~~
#账号A
Host gitjacob
HostName github.com #要连接的Git服务器地址 
User git
IdentityFile ~/.ssh/id_rsa # A账号 ssh key

#账号B
Host gityjj
HostName github.com
User git
IdentityFile ~/.ssh/id_rsa2 # B账号 ssh key
~~~

### 添加到ssh-agent
先查看 ssh key 是否添加，没就添加以下。我就是卡这里了 卡了好长时间

添加之前先清除下

~~~
jacob@JacobdeMacBook-Pro:~/.ssh$ ssh-add -D
~~~

ssh-add

~~~
jacob@JacobdeMacBook-Pro:~/.ssh$ ssh-add ~/.ssh/id_rsa
Identity added: /Users/jacob/.ssh/id_rsa (/Users/jacob/.ssh/id_rsa)
jacob@JacobdeMacBook-Pro:~/.ssh$ ssh-add ~/.ssh/id_rsa2
Identity added: /Users/jacob/.ssh/id_rsa2 (/Users/jacob/.ssh/id_rsa2)
~~~

查看

~~~
jacob@JacobdeMacBook-Pro:~/.ssh$ ssh-add -l
2048 20:05:b7:e0:a2:6d:5b:18:92:fa:5d:d2:bd:d7:1a:18 /Users/jacob/.ssh/id_rsa2 (RSA)
2048 8d:30:a0:56:52:9b:28:92:ed:ab:04:21:bc:dd:7c:c2 /Users/jacob/.ssh/id_rsa (RSA)
~~~

### 测试下
先试试A账号

~~~
jacob@JacobdeMacBook-Pro:~/.ssh$ ssh -T git@gitjacob
Hi **A**! You've successfully authenticated, but GitHub does not provide shell access.
~~~

这里看到提示 GitHub 账号 A

再看B账号

~~~
jacob@JacobdeMacBook-Pro:~/.ssh$ ssh -T git@gityjj
Hi **B**! You've successfully authenticated, but GitHub does not provide shell access.
~~~

这里看到提示 GitHub 账号 B

这样就配置好了，接下来就可以试试 分别往两个账号 push 了。

### Push&Pull
上面配置好ssh key 之后，需要设置下 remote url，请求的账户分别是哪个。

打开A账户的项目目录

~~~
jacob@JacobdeMacBook-Pro:~/projects/gitnote/pjA$ git remote add gitjacob git@gitjacob:A/pjA.git
~~~

B账户项目目录

~~~
jacob@JacobdeMacBook-Pro:~/projects/gitnote/pjB$ git remote add gityjj git@gityjj:B/pjB.git
~~~

push/pull

~~~
jacob@JacobdeMacBook-Pro:~/projects/gitnote/pjA$ git pull gitjacob master
jacob@JacobdeMacBook-Pro:~/projects/gitnote/pjA$ git push gitjacob master
jacob@JacobdeMacBook-Pro:~/projects/gitnote/pjB$ git pull gityjj master
jacob@JacobdeMacBook-Pro:~/projects/gitnote/pjB$ git push gityjj master
~~~

参考资料

[Multiple GitHub Accounts & SSH Config](http://stackoverflow.com/questions/3225862/multiple-github-accounts-ssh-config)

[Best way to use multiple SSH private keys on one client](http://stackoverflow.com/questions/2419566/best-way-to-use-multiple-ssh-private-keys-on-one-client)

[How to use multiple ssh keys with different accounts and hosts](http://askubuntu.com/questions/269140/how-to-use-multiple-ssh-keys-with-different-accounts-and-hosts)