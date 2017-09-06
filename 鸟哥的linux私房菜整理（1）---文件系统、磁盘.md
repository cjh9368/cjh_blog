# 鸟哥的linux私房菜整理（1）---文件系统、磁盘
本文用于整理鸟哥的linux私房菜中关于文件系统、磁盘相关的知识，主要包括下面几个方面：
- **linux用户组与权限配置**
- **linux目录配置**
- **linux文件搜索和文件权限相关知识**
- **linux磁盘挂载与格式化**
- **linux压缩和打包相关**
---------------
## **linux用户组与权限配置**  
&ensp;&ensp;&ensp;&ensp;linux任何一个文件或目录都具有User、Group和Others三个身份的个别权限。分别表示文件所有者、所在组别和既非文件所有者也非同一组别的用户对该文件的权限。此外root用户拥有对所有文件的全部权限。关于用户及root的信息存在/etc/passwd中,组名存在/etc/group中,个人的密码存在/etc/shadow中。  
&ensp;&ensp;&ensp;&ensp;需要查询linux的文件属性可以用ls -al命令，ls是list的意思,-al表示输出当前目录下所有文件拥有的权限和属性。出现的七个字段的意义如下：  
<div align=left><img src="https://github.com/cjh9368/cjh_blog/blob/master/img/%E6%9D%83%E9%99%90%E5%B1%9E%E6%80%A7.gif"></div>  
&ensp;&ensp;&ensp;&ensp;第一栏分别表示文件的类型（文件、目录或链接文件），文件拥有者、管理组和others所拥有的对文件操作的权限。三个权限分别为可读、可写和可执行。后面几拦的意义如图所示。   
&ensp;&ensp;&ensp;&ensp;改变文件属性的命令主要有三个：chgrp、chown和chmod，分别用于改变文件的群组、用户和权限。   
&ensp;&ensp;&ensp;&ensp;chgrp是change group的意思,用法如下,其中-R表示递归的改变目录下所有文件的群组。

```shell
chgrp [-R] group dirname/filename
chgrp users install.log
chgrp -R users tmp
```  
&ensp;&ensp;&ensp;&ensp;chown是change owner的意思，其中的owner必须是/etc/passwd中存在的用户，另外还可以用于改变文件所在的群组，它的用法如下。 

```shell
chown [-R] owner dirname/filename
chown [-R] owner group dirname/filename
chown bin install.log
chown root root install.log
```  
&ensp;&ensp;&ensp;&ensp;chmod是change mode的意思。文件的权限字符为-rwxrwxrwx，分别表示用户、群组和others的rwx权限，rwx分别用4、2、1表示，对于一部分的权限将这三个分数叠加即可，例如将某个文件test的权限配置为[-rwxrwx---]，则它的分数为：  
owner = rwx = 4+2+1 = 7  
group = rwx = 4+2+1 = 7  
others= --- = 0+0+0 = 0  
配置命令为：  

```
chmode 770 test
```
此外，chmod还有一种符号改变权限的方法，如下所示： 

```Bash  
chmode u=rwx,go=rx test
```   
--------------------------
## **linux目录配置**
&ensp;&ensp;&ensp;&ensp;linux的目录配置遵循着FHS(Filesystem Hierarchy Standard )，将文件按照是否可以共享和是否可以改动定义为以下四种交互形式：

|    |    可共享           | 不可共享      |
|----| ------------------- | ------------- |
|不变|/usr (软件放置处)    |/etc (配置文件)|
|    |/opt (第三方协力软件)|/boot (开机档) |
|可变|/var/mail(邮箱)      |/var/run       |
|    |/var/spool/news      |/var/lock      |

&ensp;&ensp;&ensp;&ensp;根目录是linux最重要的目录，FHS标准建议根目录所在的分隔槽越小越好，因为分隔槽越小放入的无关数据越少，越不容易发生错误。根目录下的主要文件夹及其用途如下所示：

|    目录           | 应放置文件内容      |
| ------------------- | ------------- |
|/bin|主要放置可执行文件，包括cat, chmod, chown, date, mv, mkdir, cp, bash等等常用的指令。|
|/boot|主要在放置开机会使用到的文件|
|/dev(device)|主要放置接口文件，如/dev/null, /dev/zero, /dev/tty, /dev/lp*, /dev/hd*, /dev/sd*等等|
|/etc|主要放置系统的配置文件，包括人员的账号密码文件、 各种服务的启始档等等。一般来说，这个目录下的各文件属性是可以让一般使用者查阅的， 但是只有root有权力修改。|
|/home|家目录，即~|
|/lib|主要放置库函数|
|/media|媒体文件夹，放置可以移除的设备包括光盘、DVD等|
|/mnt|暂时挂载一些额外的设备|
|/opt(optional application software packages)|第三方软件安装目录|
|/root|系统管理员的家目录|
|/sbin|用于设定系统环境的指令，这些指令只有root用户能使用，常见的包括：fdisk, fsck, ifconfig, init, mkfs等等。|
|/srv|网络服务包括WWW，FTP等|
|/tmp|临时文件存放位置|

&ensp;&ensp;&ensp;&ensp;除了上面FHS规定的这几个文件夹外，根目录下/usr和/var也很重要。usr的全称为Unix Software Resource(之前一直以为是user/(ㄒoㄒ)/~~)是unix系统软件资源放置位置，它的次目录与跟目录的次目录比较相似，如下所示：

|    目录           | 应放置文件内容      |
| ------------------- | ------------- |
|/usr/bin/|绝大多数用户可以使用的指令，与/usr主要的差别在于不包括开机相关的指令|
|/usr/include/|c/c++使用的头文件存放位置|
|/usr/lib/|各种应用软件的函数库和目标文件|
|/usr/local/|系统管理员自行下载的软件建议安装到此目录，此目录下也有bin，etc，include，lib等此目录|
|/usr/sbin/|非系统正常运行所需的系统指令(没太明白)|
|/usr/share/|放置共享文件，住要为一些文本文件（说明文档、帮助文档）|
|/usr/src/|一般源代码存放位置|

&ensp;&ensp;&ensp;&ensp;/var主要用于存储大型的数据文件，包括缓存文件、日志文件、软件运行时产生的文件以及数据库文件等，它的次目录结构如下：

|    目录           | 应放置文件内容      |
| ------------------- | ------------- |
|/var/cache/|应用程序本身的缓存文件|
|/var/lib/|应用程序运行过程中本身所需要用到的数据文件放置位置，例如MySQL的数据库放置在/var/lib/mysql/|
|/var/lock/|linux系统下挂载各个系统资源所需要的锁的存放位置|
|/var/log/|登录文件放置的位置|
|/var/mail/|个人电子邮箱的目录，和/var/spool/mail互为链接文件|
|/var/run/|程序或服务启动时存放进程PID的位置|
|/var/spool/|存放进程的队列数据的位置|
&ensp;&ensp;&ensp;&ensp;综上，根目录下主要文件结构的目录树如下所示： 
<div align=left><img src="https://github.com/cjh9368/cjh_blog/blob/master/img/directory_tree.gif"></div> 


