# 鸟哥的linux私房菜整理（1）---文件系统、磁盘
本文用于整理鸟哥的linux私房菜中关于文件系统、磁盘相关的知识，主要包括下面几个方面：
- **linux用户组与权限配置**
- **linux文件搜索和文件权限相关知识**
- **linux磁盘挂载与格式化**
- **linux压缩和打包相关**
---------------
## **linux用户组与权限配置**  
&ensp;&ensp;&ensp;&ensp;linux任何一个文件或目录都具有User、Group和Others三个身份的个别权限。分别表示文件所有者、所在组别和既非文件所有者也非同一组别的用户对该文件的权限。此外root用户拥有对所有文件的全部权限。关于用户及root的信息存在/etc/passwd中,组名存在/etc/group中,个人的密码存在/etc/shadow中。  
&ensp;&ensp;&ensp;&ensp;需要查询linux的文件属性可以用ls -al命令，ls是list的意思,-al表示输出当前目录下所有文件拥有的权限和属性。出现的七个字段的意义如下：  
![Alt text](./img/权限属性.gif)