#linux 权限

Linux继承和发展了Unix的“一切皆文件”的思想，是unix在新时代下的先锋和旗手。

GNU/Linux 通过用户和用户组实现访问控制 —— 包括对文件访问、设备使用的控制。Linux 默认的访问控制机制相对简单直接，不过还有一些更加高级的机制，包括 ACL 和 LDAP Authentication. 

##1. 使用者与群组

首先先来介绍以下linux下的文件拥有者，群组，与其他人的概念。

###1.1. 文件拥有者（ownwer）

文件拥有者描述的是这个文件到底是谁的。就像锅一样，总得有一个人把锅背起来；那么背锅的人就是锅这个文件的文件拥有者。当然，如同锅可以甩一样，文件的拥有者是可以变的（`chown`命令就是来干这个的）。

###1.2. 群组（group）

群组呢就是一群用户的集合，他们共同属于一个集体。举个栗子，背锅的文件拥有者就是我们项目组这个群组（group）的一份子；而另一方面，背锅的文件拥有者也可是是另一个群组的一份子。

那么问题来了

1. 背锅的文件拥有者不属于文件的当前群组么？
>答案是可以的，原因么就不知道了

2. 如果一个用户属于多个群组，那么他创建的文件默认属于哪个用户组呢？
>答案是用户执行该命令是所在的有效群组

3. 有效群组（effective group）又是个啥？
>回答这个之前先扯一下初始群组(initial group)：
>初始群组(initial group)是指用户一登入系统就拥有的群组权限，记录在`/etc/passwd`的第四栏；却不需要在`/etc/group`的第四栏中写入该用户
>好了，有效群组就是该用户在`/etc/group`中所属的群组（即，第四栏中包含该帐号）

4. 那么问题又来了，如果一个用户同时有初始群组和有效群组，那到底新建文件的群组到底是谁呢？
>答案：使用`groups`命令查看用户当前所属群组中的第一个群组就是啦。

5. 怎样切换有效群组呢？
>答案：`newgrp groupname`就可以啦，当然当前用户必须属于groupname，否则嘿嘿

###1.3. 其他人(others)
其他人就是就不是文件拥有者又不是文件所属群组的用户，举个栗子来说隔壁项目组的成员和楼下的保安叔叔对于背锅者创建的文件来说就是其他人，除非嘿嘿嘿

###1.4 root用户
一句话，root用户拥有操作系统的无上权限，随时可以横着走

##2. 文件权限
扯了那么多用户，现在该干正事来说说文件权限了。先上图（注：下图为`ls -al`的输出）

![文件属性的示意图](http://vbird.dic.ksu.edu.tw/linux_basic/0210filepermission_files/0210filepermission_2.gif)

(图：文件属性的示意图)

###2.1. 文档的类型与权限

其中，第一部分是 文件的类型与权限，再来一张图

![文件的类型与权限之内容](http://vbird.dic.ksu.edu.tw/linux_basic/0210filepermission_files/0210filepermission_3.gif)

(图：文件的类型与权限之内容)

- 其中第一个字符代表的是文件类型
>d 代表目录
>- 代表文件
>l 代表链接文件(link file)
>b 代表为装置文件里的可供存储的接口设备（可随机存取装置）
>c 代表为装置文件里的串行端口设备

- 接下来就是三个一组的权限描述了，分别描述了 文档拥有者权限、 所属组权限 和 其他人权限
>r表示可读(read) 
>w表示可写(write)
>x表示可执行(execute)

###2.2. 剩下的栏

-     第二栏表示有多少档名连结到此节点(i-node)：
~每个文件都会将他的权限与属性记录到文件系统的i-node中，不过，我们使用的目录树却是使用文件名来记录， 因此每个档名就会连结到一个i-node啰！这个属性记录的，就是有多少不同的档名连结到相同的一个i-node号码去就是了。~

-    第三栏表示这个文件(或目录)的『拥有者账号』
-    第四栏表示这个文件的所属群组
~在Linux系统下，你的账号会附属于一个或多个的群组中~
- 第五栏为这个文件的容量大小，默认单位为bytes；
- 第六栏为这个文件的建档日期或者是最近的修改日期：
~这一栏的内容分别为日期(月/日)及时间。如果这个文件被修改的时间距离现在太久了，那么时间部分会仅显示年份而已~
-    第七栏为这个文件的档名
~这个字段就是档名了。比较特殊的是：如果档名之前多一个『 . 』，则代表这个文件为『隐藏档』~

## 3. 为什么要有文件权限

一切的一切都只是为了：安全和共享
举个栗子，鸟哥的私房菜曾经曰过
>在Linux系统当中，每一个文件都多加了很多的属性进来，尤其是群组的概念，这样有什么用途呢？ 其实，最大的用途是在『数据安全性』上面的。
>
>    系统保护的功能：
>    举个简单的例子，在你的系统中，关于系统服务的文件通常只有root才能读写或者是执行，例如/etc/shadow这一个账号管理的文件，由于该文件记录了你系统中所有账号的数据， 因此是很重要的一个配置文件，当然不能让任何人读取(否则密码会被窃取啊)，只有root才能够来读取啰！所以该文件的权限就会成为[ -rw------- ]啰！
>
>    团队开发软件或数据共享的功能：
>    此外，如果你有一个软件开发团队，在你的团队中，你希望每个人都可以使用某一些目录下的文件， 而非你的团队的其他人则不予以开放呢？以上面的例子来说，testgroup的团队共有三个人，分别是test1, test2, test3，那么我就可以将团队所需的文件权限订为[ -rwxrwx--- ]来提供给testgroup的工作团队使用啰！
>
>    未将权限设定妥当的危害：
>    再举个例子来说，如果你的目录权限没有作好的话，可能造成其他人都可以在你的系统上面乱搞啰！ 例如本来只有root才能做的开关机、ADSL的拨接程序、新增或删除用户等等的指令，若被你改成任何人都可以执行的话， 那么如果使用者不小心给你重新启动啦！重新拨接啦！等等的！那么你的系统不就会常常莫名其妙的挂掉啰！ 而且万一你的用户的密码被其他不明人士取得的话，只要他登入你的系统就可以轻而易举的执行一些root的工作！
>
>可怕吧！因此，在你修改你的linux文件与目录的属性之前，一定要先搞清楚， 什么数据是可变的，什么是不可变的！千万注意啰！接下来我们来处理一下文件属性与权限的变更吧！

##4. 如何更改权限

-    chgrp ：改变文件所属群组
-    chown ：改变文件拥有者
-    chmod ：改变文件的权限, SUID, SGID, SBIT等等的特性

注：命令如何使用请`man`之,`info`和`--help`都可以啦

##5. 文件权限和目录权限

###5.1. 文件权限

文件是实际含有数据的地方，包括一般文本文件、数据库内容文件、二进制可执行文件(binary program)等等。
-    r (read)：可读取此一文件的实际内容，如读取文本文件的文字内容等；
-    w (write)：可以编辑、新增或者是修改该文件的内容(但不含删除该文件)；
-    x (execute)：该文件具有可以被系统执行的权限。


**在Linux下，文件能否被执行，则是藉由是否具有『x』这个权限来决定的**
**当用户对一个文件具有w权限时，可以具有`写入/编辑/新增/修改`文件的内容的权限， 但并不具备有删除该文件本身的权限！**
对于文件的rwx来说， 主要都是针对『文件的内容』而言，与文件档名的存在与否没有关系喔！因为文件记录的是实际的数据嘛！

###5.2.  目录文件

目录主要的内容在记录文件名列表，文件名与目录有强烈的关连啦！ 那 r, w, x 对目录是什么意义呢？

-    r (read contents in directory)：

    表示具有读取目录结构列表的权限，所以当你具有读取(r)一个目录的权限时，表示你可以查询该目录下的文件名数据。 所以你就可以利用 ls 这个指令将该目录的内容列表显示出来！

-    w (modify contents of directory)：

    这个可写入的权限对目录来说，是很了不起的！ 因为他表示你具有异动该目录结构列表的权限，也就是底下这些权限：

        建立新的文件与目录；
        删除已经存在的文件与目录(不论该文件的权限为何！)
        将已存在的文件或目录进行更名；
        搬移该目录内的文件、目录位置。

    总之，目录的w权限就与该目录底下的文件名异动有关就对了啦！

-    x (access directory)：

    目录的x代表的是用户能否进入该目录成为工作目录的用途！ 所谓的工作目录(work directory)就是用户目前所在的目录啦！
	举例来说，当你登入Linux时， 你所在的家目录就是你当下的工作目录。而变换目录的指令是『cd』(change directory)啰！

## 6. 文件与目录的默认权限与隐藏权限

###6.1. 文件默认权限：umask

 umask 就是指定 『目前使用者在创建文件或目录时候的权限默认值』
```
umask <- 查看数字形态的权限配置
umask -S <- 查看符号类型的权限配置
umask 002 <- 设置umask为002
```
在默认权限的属性上，目录与文件是不一样的。从第六章我们知道 x 权限对於目录是非常重要的！ 但是一般文件的创建则不应该有运行的权限，因为一般文件通常是用在於数据的记录嘛！当然不需要运行的权限了。 因此，默认的情况如下：

-    若使用者创建为『文件』则默认『没有可运行( x )权限』，亦即只有 rw 这两个项目，也就是最大为 666 分，默认权限如下：
    -rw-rw-rw-

-    若使用者创建为『目录』，则由於 x 与是否可以进入此目录有关，因此默认为所有权限均开放，亦即为 777 分，默认权限如下：
    drwxrwxrwx

**要注意的是，umask 的分数指的是『该默认值需要减掉的权限！』因为 r、w、x 分别是 4、2、1 分**

### 6.2. 隐藏属性`lsattr`及`chattr`

- lsattr （现实文件隐藏类型）
```
lsattr [-adR] 文件或目录
选项与参数：请man之
```
-    chattr (配置文件隐藏属性)
```
chattr [+-=][ASacdistu] 文件或目录名称
选项与参数：请man之
```
鸟哥曾经曰过
>最重要的当属 +i 与 +a 这个属性了。+i 可以让一个文件无法被更动，对於需要强烈的系统安全的人来说， 真是相当的重要的！
>此外，如果是 log file 这种的登录档，就更需要 +a 这个可以添加，但是不能修改旧有的数据与删除的参数了！
> 里头还有相当多的属性是需要 root 才能配置的呢！

### 6.3. 文件特殊权限：SUID, SGID, SBIT

- SUID
当 s 这个标志出现在文件拥有者的 x 权限上时，此时就被称为 Set UID，简称为 SUID 的特殊权限。 那么SUID的权限对於一个文件的特殊功能是什么呢？基本上SUID有这样的限制与功能：
	- SUID 权限仅对二进位程序(binary program)有效；
	- 运行者对於该程序需要具有 x 的可运行权限；
	- 本权限仅在运行该程序的过程中有效 (run-time)；
	- 运行者将具有该程序拥有者 (owner) 的权限。

- SGID
那 s 在群组的 x 时则称为 Set GID, SGID 罗！
**与 SUID 不同的是，SGID 可以针对文件或目录来配置！如果是对文件来说， SGID 有如下的功能：**
	- SGID 对二进位程序有用；
	- 程序运行者对於该程序来说，需具备 x 的权限；
	- 运行者在运行的过程中将会获得该程序群组的支持！
**事实上 SGID 也能够用在目录上，这也是非常常见的一种用途！ 当一个目录配置了 SGID 的权限后，他将具有如下的功能：**
	- 使用者若对於此目录具有 r 与 x 的权限时，该使用者能够进入此目录；
	- 使用者在此目录下的有效群组(effective group)将会变成该目录的群组；
	- 用途：若使用者在此目录下具有 w 的权限(可以新建文件)，则使用者所创建的新文件，该新文件的群组与此目录的群组相同。

- SBIT
这个 Sticky Bit, SBIT 目前只针对目录有效，对於文件已经没有效果了。 SBIT 对於目录的作用是：
	- 当使用者对於此目录具有 w, x 权限，亦即具有写入的权限时；
	- 当使用者在该目录下创建文件或目录时，仅有自己与 root 才有权力删除该文件

换句话说：当甲这个使用者於 A 目录是具有群组或其他人的身份，并且拥有该目录 w 的权限， 这表示『甲使用者对该目录内任何人创建的目录或文件均可进行 "删除/更名/搬移" 等动作。』 不过，如果将 A 目录加上了 SBIT 的权限项目时， 则甲只能够针对自己创建的文件或目录进行删除/更名/移动等动作，而无法删除他人的文件。

###6.4. 观察文件类型：file

如果想要知道某个文件的基本数据，就可以利用 file 这个命令来检阅，具体用法请man之

### 7. 参考文献

[鸟哥的私房菜： 第六章、Linux 的文件权限与目录配置](http://vbird.dic.ksu.edu.tw/linux_basic/0210filepermission.php)
[鸟哥的私房菜：  第七章、Linux 文件与目录管理](http://vbird.dic.ksu.edu.tw/linux_basic/0220filemanager.php)

