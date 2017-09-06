# 鸟哥的linux私房菜整理（1）---文件系统、磁盘
本文用于整理鸟哥的linux私房菜中关于文件系统、磁盘相关的知识，主要包括下面几个方面：
- **linux用户组与权限配置**
- **linux目录配置**
- **linux文件和目录管理**
- **linux磁盘挂载与格式化**
- **linux压缩和打包相关**
---------------
## **linux用户组与权限配置**  
linux任何一个文件或目录都具有User、Group和Others三个身份的个别权限。分别表示文件所有者、所在组别和既非文件所有者也非同一组别的用户对该文件的权限。此外root用户拥有对所有文件的全部权限。关于用户及root的信息存在/etc/passwd中,组名存在/etc/group中,个人的密码存在/etc/shadow中。  
需要查询linux的文件属性可以用ls -al命令，ls是list的意思,-al表示输出当前目录下所有文件拥有的权限和属性。出现的七个字段的意义如下：   
<div align=left><img src="https://github.com/cjh9368/cjh_blog/blob/master/img/%E6%9D%83%E9%99%90%E5%B1%9E%E6%80%A7.gif"></div>  

第一栏分别表示文件的类型（文件、目录或链接文件），文件拥有者、管理组和others所拥有的对文件操作的权限。三个权限分别为可读、可写和可执行。后面几栏的意义如图所示。   
改变文件属性的命令主要有三个：chgrp、chown和chmod，分别用于改变文件的群组、用户和权限。   
chgrp是change group的意思,用法如下,其中-R表示递归的改变目录下所有文件的群组。

```shell
chgrp [-R] group dirname/filename
chgrp users install.log
chgrp -R users tmp
```  
chown是change owner的意思，其中的owner必须是/etc/passwd中存在的用户，另外还可以用于改变文件所在的群组，它的用法如下。 

```shell
chown [-R] owner dirname/filename
chown [-R] owner group dirname/filename
chown bin install.log
chown root root install.log
```  
chmod是change mode的意思。文件的权限字符为-rwxrwxrwx，分别表示用户、群组和others的rwx权限，rwx分别用4、2、1表示，对于一部分的权限将这三个分数叠加即可，例如将某个文件test的权限配置为[-rwxrwx---]，则它的分数为：

```  
owner = rwx = 4+2+1 = 7  
group = rwx = 4+2+1 = 7  
others= --- = 0+0+0 = 0   
```
配置命令为：

``` 
chmod 770 test
```
此外，chmod还有一种符号改变权限的方法，如下所示： 

```Bash  
chmod u=rwx,go=rx test
```
文件默认权限可以通过umask查看。和前面chmod的分数不同，umask的分数为默认值需要减去的权限，r、w、x分别为4、2、1，若创建为文件，则默认没有可运行权限，最大只有666分，若创建为目录则默认所有权限均开放，最大为777分。举个例子，如果查到umask分数为022，那么就说明被拿掉了group和others的w权限，所以对于文件和目录相应的权限计算就如下所示：

```Bash
创建文件时：(-rw-rw-rw-) - (-----w--w-) ==> -rw-r--r--
创建目录时：(drwxrwxrwx) - (d----w--w-) ==> drwxr-xr-x
```
另外，对于umask而言更重要的特性是可以为不同的目标定制权限，例如要设置只有文件的所有者可以修改所有文件，那么可以设置umask为022，拿掉除创建用户外所有用户的w权限，如下所示：

```Bash
umask 022
```
除了上面的9个权限外，在ext2/ext3文件系统中还有一些隐藏权限（A\S\a\c\d\i\s\u）其中最重要的权限为a和i，a权限表示用户只能对该文件进行添加操作，不能进行删除和修改操作，i权限表示该文件不能被删除、改名、配置链接，也无法对文件新增数据，只有root用户可以配置这个属性。
配置和查看这些隐藏属性可以通过chattr和lsattr进行，如下所示：

```Bash
chattr [+-=][ASacdistu] 文件或目录名称
lsattr [-adR] 文件或目录

[root@www tmp]# chattr +i attrtest <==给予 i 的属性
[root@www tmp]# chattr +aij attrtest
[root@www tmp]# lsattr attrtest
----ia---j--- attrtest
```

## **linux目录配置**
linux的目录配置遵循着FHS(Filesystem Hierarchy Standard )规则，将文件按照是否可以共享和是否可以改动定义为以下四种交互形式：

|    |可共享|不可共享|
|:----:| :---------------| :-----------|
|不变|/usr (软件放置处)    |/etc (配置文件)|
|    |/opt (第三方协力软件)|/boot (开机档) |
|可变|/var/mail(邮箱)      |/var/run       |
|    |/var/spool/news      |/var/lock      |

根目录是linux最重要的目录，FHS标准建议根目录所在的分隔槽越小越好，因为分隔槽越小放入的无关数据越少，越不容易发生错误。根目录下的主要文件夹及其用途如下所示：

|目录|应放置文件内容|
| :------------------- | :------------- |
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

除了上面FHS规定的这几个文件夹外，根目录下/usr和/var也很重要。usr的全称为Unix Software Resource(之前一直以为是user/(ㄒoㄒ)/~~)是unix系统软件资源放置位置，它的次目录与跟目录的次目录比较相似，如下所示：

|目录|应放置文件内容|
| :------------------- | :------------- |
|/usr/bin/|绝大多数用户可以使用的指令，与/usr主要的差别在于不包括开机相关的指令|
|/usr/include/|c/c++使用的头文件存放位置|
|/usr/lib/|各种应用软件的函数库和目标文件|
|/usr/local/|系统管理员自行下载的软件建议安装到此目录，此目录下也有bin，etc，include，lib等此目录|
|/usr/sbin/|非系统正常运行所需的系统指令(没太明白)|
|/usr/share/|放置共享文件，住要为一些文本文件（说明文档、帮助文档）|
|/usr/src/|一般源代码存放位置|

var主要用于存储大型的数据文件，包括缓存文件、日志文件、软件运行时产生的文件以及数据库文件等，它的次目录结构如下：

|目录|应放置文件内容|
| :------------------- | :------------- |
|/var/cache/|应用程序本身的缓存文件|
|/var/lib/|应用程序运行过程中本身所需要用到的数据文件放置位置，例如MySQL的数据库放置在/var/lib/mysql/|
|/var/lock/|linux系统下挂载各个系统资源所需要的锁的存放位置|
|/var/log/|登录文件放置的位置|
|/var/mail/|个人电子邮箱的目录，和/var/spool/mail互为链接文件|
|/var/run/|程序或服务启动时存放进程PID的位置|
|/var/spool/|存放进程的队列数据的位置|  

综上，根目录下主要文件结构的目录树如下所示： 
<div align=left><img src="https://github.com/cjh9368/cjh_blog/blob/master/img/directory_tree.gif"></div> 

## **linux文件和目录管理**
linux的文件查阅命令主要有下面几种：
- **cat由第一行开始显示文件内容**
- **tac从最后一行开始显示**
- **more一页一页的显示文件内容**
- **less与more 类似，但是比more更好的是，他可以往前翻页**
- **head只看头几行**
- **tail只看末尾几行** 

cat的全称是concatenate，用法如下：

```Bash
cat test
cat -n test #打印的同时显示行号
```
tac即从末尾往前显示，用法和cat相同。  
more表示一页一页显示文件内容，首先执行more进入逐页浏览的状态（如下所示），然后按空格进入下一页，回车下一行，b往回翻页，完成浏览后按q退出浏览模式。

```Bash
[root@www ~]# more /etc/man.config
#
# Generated automatically from man.conf.in by the
# configure script.
#
# man.conf from man-1.6d
....(中间省略)....
--More--(28%)  
```
less的功能与more类似，但是比more更加灵活，利用less进入浏览模式后可以输入以下命令继续浏览文件：

```Bash
空白键    ：向下翻动一页；
[pagedown]：向下翻动一页；
[pageup]  ：向上翻动一页；
/字串     ：向下搜寻『字串』的功能；
?字串     ：向上搜寻『字串』的功能；
n         ：重复前一个搜寻 (与 / 或 ? 有关！)
N         ：反向的重复前一个搜寻 (与 / 或 ? 有关！)
q         ：离开 less 这个程序；
```
head和tail操作比较简单，如下所示：

```Bash
head [-n number] 文件 
head -n 20 /etc/man.config #显示前20行
head -n -100 /etc/man.config #显示倒数100行之前的行数

tail [-n number] 文件 
tail -n 20 /etc/man.config
tail -n +100 /etc/man.config #显示100行以后的文件
```




