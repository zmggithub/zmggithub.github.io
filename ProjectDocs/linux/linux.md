# man  不懂就问命令 

# Linux

## vim

## **jobs**

在LInux中，这个组合按键的作用是当前进程暂停，被[挂起](https://so.csdn.net/so/search?q=挂起&spm=1001.2101.3001.7020)到后台了。
输入命令 **jobs**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513191903735.png)
可以看到后台挂起的进程

输入命令fg %num可以恢复指定标号的进程。
这里我只有一个进程被挂起，就是我不小心[中断](https://so.csdn.net/so/search?q=中断&spm=1001.2101.3001.7020)的这个。
所以我这里使用fg %1就可以继续进行我的编辑工作了 ~~~

或者 使用fg2

foreground --》 fg 

## netstat   查看端口号占用状态

```
netstat -anpt |grep 3306
netstat -tlnp|grep 8080
```

## tar

```
tar cvf archive_name.tar dirname	创建一个新的tar文件
tar xvf archive_name.tar	解压tar文件
tar tvf archive_name.tar	查看tar文件
```



## gzip

```
gzip -l *.gz	显示压缩的比率
gzip -d test.txt.gz	解压*.gz文件
gzip test.txt	创建一个*.gz的压缩文件
```

​			

## unzip

1、把文件解压到当前目录下

```
unzip test.zip
```

2、如果要把文件解压到指定的目录下，需要用到-d参数。

```
unzip -d /temp test.zip
```

3、解压的时候，有时候不想覆盖已经存在的文件，那么可以加上-n参数

```
unzip -n test.zip
unzip -n -d /temp test.zip
```

4、只看一下zip压缩包中包含哪些文件，不进行解压缩

```
unzip -l test.zip
```

5、查看显示的文件列表还包含压缩比率

```
unzip -v test.zip
```

6、检查zip文件是否损坏

```
unzip -t test.zip
```

7、将压缩文件test.zip在指定目录tmp下解压缩，如果已有相同的文件存在，要求unzip命令覆盖原先的文件

```
unzip -o test.zip -d /tmp/	
```

## rm	

```
rm -rf 	rm -rf /var/log/httpd/access	删除文件夹和文件的命令  -r 就是向下递归，不管有多少级目录，一并删除 ； -f 就是直接强行删除，不作任何提示的意思
rm -i file*		递归删除文件夹下所有文件，并删除该文件夹 
```

## tail

```
tail -f consoleMsg.log | grep --line-buffered findUserList	实时跟踪日志，这里是只要findUserList 这个方法被运行，就会将它的日志打印出来，用于跟踪特定的日志运行。--line-buffered 是一行的缓冲区，只要这一行的缓冲区满了就会打印出来，所以可以用于实时监控日志。
tail -f -n 500 consoleMsg.log	打印最后500行日志，并且持续跟踪日志。
tail -n 2000 consoleMsg.log | more	分页查看最后2000行日志
```

## df

```
命令是linux系统以磁盘分区为单位查看文件系统，可以加上参数查看磁盘剩余空间信息
	df -hl		查看磁盘剩余空间
	df -h		查看每个根路径的分区大小
	du -sh [目录名]		返回该目录的大小
	du -sm [文件夹]		返回该文件夹总M数
	du -h [目录名]		查看指定文件夹下的所有文件大小（包含子文件夹）
	df -k		显示文件系统的磁盘使用情况，默认情况下df -k 将以字节为单位输出磁盘的使用量
	df -T		使用-T选项显示文件系统类型 
```

## date

```
date -s ‘2018-12-14 10:50:00’		修改时间
timedatectl status		时间详情
hwclock --show		显示硬件时间
hwclock --systohc --localtime		当你修改了系统时间，你需要同步硬件时间和系统时间 
clock -w		当前日期写入CMOS
```

​			

## wget

```
wget http://prdownloads.sourceforge.net/sourceforge/nagios/nagios-3.2.1.tar.gz	使用wget从网上下载软件、音乐、视频 
wget -O taglist.zip http://www.vim.org/scripts/download_script.php?src_id=7701	下载文件并以指定的文件名保存文件
```

## touch

```
touch consoleMsg.log   	命令用于创建空白文件，以及设置文件的时间。
```

# Linux中的三剑客 grep、sed、awk

## grep

```
grep -i "the" demo_file	在文件中查找字符串(不区分大小写)
grep -A 3 -i "example" demo_text	输出成功匹配的行，以及该行之后的三行
grep -r "ramesh" *	在一个文件夹中递归查询包含指定字符串的文件
grep  -r  "要查找的字符串" *
grep -R -w -l ‘boot’ /etc > ./output.txt
grep "要查找的字符串" * -r 

# grep的选项介绍
-a ：将 binary 文件以 text 文件的方式搜寻数据
-r ：递归搜索
-v ：反向选取
-o ：只显示被模式匹配到的字符串，而不是整个行
-i ：匹配时不区分大小写
-A  5 ：显示匹配到的行时，显示后面的 5 行
-B  5 ：显示匹配到的行时，前面的 5 行
-C  5 ：显示匹配到的行时，前后的 5 行
-E ：使用扩展的正则表达式
```

## sed

```
sed 's/.$//' filename	当你将Dos系统中的文件复制到Unix/Linux后，这个文件每行都会以\r\n结尾，sed可以轻易将其转换为Unix格式的文件，使用\n结尾的文件
sed -n '1!G; h; p' filename	反转文件内容并输出
sed '/./=' thegeekstuff.txt | sed 'N; s/\n/ /'	为非空行添加行号
```

## awk

```
awk '!($0 in array) { array[$0]; print}' temp	删除重复行
awk -F ':' '$3=$4' /etc/passwd	打印/etc/passwd中所有包含同样的uid和gid的行
awk '{print $2,$5;}' employee.txt	打印文件中的指定部分的字段
```



# Linux命令之搜索.gz压缩文件包含关键字的内容

    # linux需要查看历史日志，且又不想解压该日志，搜索订单编号884381688886关键字
    zcat nbsp-2021-05-06-11-82.log.gz  | grep -a -C 30 '884381688886'



## find 查找文件

```
find -i name "MyProgram.c"	查找指定文件名的文件(不区分大小写)
find -i name "MyProgram.c" -exec md5sum {} \;	对找到的文件执行某个命令
find ~ -empty	查找home目录下的所有空文件
```



## ssh 远程登录

```
`ssh USERNAME@IP  `  或者   `ssh IP -l USERNAME -p PORT` 端口默认22时可不加

ssh -l jsmith remotehost.example.com	登录到远程主机
ssh -v -l jsmith remotehost.example.com	调试ssh客户端
ssh -V	显示ssh客户端版本
ssh USERNAME@IP  或者 ssh IP -l USERNAME -p PORT	ssh远程登录


```

## 





## diff

```
diff -w name_list.txt name_list_new.txt	比较的时候忽略空白符
```

## sort

```
sort names.txt	以升序对文件内容排序
sort -r names.txt	以降序对文件内容排序
sort -t: -k 3n /etc/passwd | more   以第三个字段对/etc/passwd的内容排序
```

## export

```
export | grep ORCALEdeclare -x ORACLE_BASE="/u01/app/oracle"declare -x ORACLE_HOME="/u01/app/oracle/product/10.2.0"declare -x ORACLE_SID="med"declare -x ORACLE_TERM="xterm"
输出跟字符串oracle匹配的环境变量
export ORACLE_HOME=/u01/app/oracle/product/10.2.0	设置全局环境变量
```



## xargs			护花使者

```
ls *.jpg | xargs -n1 -i cp {} /external-hard-drive/directory	将所有图片文件拷贝到外部驱动器
find / -name *.jpg -type f -print | xargs tar -cvzf images.tar.gz	将系统中所有jpd文件压缩打包
cat url-list.txt | xargs wget –c	下载文件中列出的所有url对应的页面
ls -lh-rw-r----- 1 ramesh team-dev 8.9M Jun 12 15:27 arch-linux.txt.gz	以易读的方式显示文件大小(显示为MB,GB...)
ls -ltr	以最后修改时间升序列出文件
ls -F	在文件名后面显示文件类型
```



## cd		

```
cd - 可以在最近工作的两个目录间切换
使用shopt -s cd spell可以设置自动对cd命令进行拼写检查
```

​			
  			

## shutdown		

```
shutdown -h now	关闭系统并立即关机 
shutdown -h +10	10分钟后关机 
shutdown -r now	重启 
shutdown -Fr now	重启期间强制进行系统检查 
```



## ftp		

```
ftp IP/hostnameftp> mget *.html	ftp命令和sftp命令的用法基本相似连接ftp服务器并下载多个文件 
```

​			

## ps	

```
ps aux | grep [pid]		ps命令用于显示正在运行中的进程的信息，ps命令有很多选项，这里只列出了几个 
ps -ef | more	查看当前正在运行的所有进程 
ps -efH | more	以树状结构显示当前正在运行的进程，H选项表示显示进程的层次结构
```

 

## free		

```
free 这个命令用于显示系统当前内存的使用情况，包括已用内存、可用内存和交换内存的情况，默认情况下free会以字节为单位输出内存的使用量
free -g	如果你想以其他单位输出内存的使用量，需要加一个选项，-g为GB，-m为MB，-k为KB，-b为字节 
free -t	如果你想查看所有内存的汇总，请使用-t选项，使用这个选项会在输出中加一个汇总行 
```

## `du`  命令可用于查看磁盘占用空间，是对文件和目录磁盘使用空间的查看

du` 是 `disk usage 的缩写

du [选项] [文件或目录]
-a	显示目录中所有文件和目录的大小
-k	以KB为单位显示文件大小
-m	以MB为单位显示文件大小
-g	以GB为单位显示文件大小
-h	人类可读，以易读方式显示文件大小
-s	只显示总和
-c	显示所有文件和子目录大小后，显示总和，在 total 行
–max-depth=n	指定统计子目录的深度为第 n 层
–help	显示帮助信息
–version	显示版本信息

```
如果要查看当前目录下所有文件的大小情况，只需要直接执行 du 命令即可，不需要任何选项参数：

du
以人类可读方式显示大小;如果想要展示出来的内容具有可读性，可以加上 -h 选项，就会为文件和目录的大小加上单位信息：
du -h


显示指定文件的大小
如果要显示指定文件的大小，可以用如下的命令格式：
# 语法
du 文件名
# 示例
du test.txt


显示指定目录的大小
如果要显示指定目录的大小，可以用如下的命令格式：
# 语法
du 目录名
# 示例
du test/
注：如果该目录下还有子目录，那么也会显示子目录的大小，并且是有多少子目录就会显示多少。

显示指定目录下所有文件的大小
可以显示指定目录下的所有文件和目录的大小，但不会深入到更深层的子目录中。格式如下：
# 语法
du -a 目录名
# 示例
du -a test/


仅显示总大小
我们可以查看某个目录的总大小：
# 语法
du -s 目录名
# 示例
du -s test/

显示当前目录下各个子目录所使用的大小
如果想要指定层次下各个子目录的大小，可以使用 --max-depth 选项，该选项会指定显示的层次：
# 语法
du --max-depth=层次 目录名
# 示例
du --max-depth=1 /root/

```











## top			

```
top命令会显示当前系统中占用资源最多的一些进程（默认以CPU占用率排序）如果你想改变排序方式，可以在结果列表中点击O（大写字母O）会显示所有可用于排序的列，这个时候你就可以选择你想排序的列
top -u oracle	如果只想显示某个特定用户的进程，可以使用-u选项 
```



## kill		

```
kill -9 :port	kill用于终止一个进程。一般我们会先用ps -ef查找某个进程得到它的进程号，然后再使用kill -9 进程号终止该进程。你还可以使用killall、pkill、xkill来终止进程
```



## cp

```
cp -p file1 file2	拷贝文件1到文件2，并保持文件的权限、属主和时间戳
cp -i file1 file2	拷贝file1到file2，如果file2存在会提示是否覆盖 
```

​			
​			

## mv

```
mv -i file1 file2	将文件名file1重命名为file2，如果file2存在则提示是否覆盖
		mv -v file1 file2	注意如果使用-f选项则不会进行提示 -v会输出重命名的过程,当文件名中包含通配符时,这个选项会非常方便
mv	mv [-fiv] source destination	mv /test1/file1 /test3/file2	文件移动命令
```

## cat	

```
cat file1 file2	你可以一次查看多个文件的内容，下面的命令会先打印file1的内容，然后打印file2的内容
cat -n /etc/logrotate.conf	 -n命令可以在每行的前面加上行号
```

## 	

## mount   如果要挂载一个文件系统，需要先创建一个目录，然后将这个文件系统挂载到这个目录上

## less

```
less huge-log-file.log	这个命令可以在不加载整个文件的前提下显示文件内容，在查看大型日志文件的时候这个命令会非常有用
CTRL+F' or 'CTRL+B'	当你用less命令打开某个文件时，下面两个按键会给你带来很多帮助，他们用于向前和向后滚屏
```

## su

```
su - USERNAME	su命令用于切换用户账号，超级用户使用这个命令可以切换到任何其他用户而不用输入密码
su - raj -c 'ls'	用另外一个用户名执行一个命令下面的示例中用户john使用raj用户名执行ls命令，执行完后返回john的账号 
su -s 'SHELLNAME' USERNAME	用指定用户登录，并且使用指定的shell程序，而不用默认的
```

​			

## yum/rpm

```
yum install/update/remove name	rpm -ivh/uvh/ev
```

​			

## history   ! 行号

## rz/sz

```
使用yum安装运行命令 yum install lrzsz（默认使没有安装运行命令的）上传命令rz 下载命令sz
```

## curl

```
命令可以用来执行下载、发送各种HTTP请求，指定HTTP头部等操作，如果系统没有curl可以使用yum install curl 安装，也可以下载安装
这是最简单的使用方法。用这个命令获得了http://curl.haxx.se；指向的页面。同样，如果这里url指向的是一个文件或者一副图都可以直接下载到本地；如果下载的是html文档。那么缺省的将只显示文件头部，即html文档的header。要显示全部，请加参数 -i
```





## linux添加用户和用户组

- 用户

  - 创建用户：useradd <用户名>

  - 设置密码：passwd <用户名>
  - 删除用户：userdel <用户名>

- 用户组：
  - 新建用户组：groupadd <用户组名称>
  - 创建用户并将其加入到用户主组，每个用户有且只有一个主用户组：useradd -g <用户组名> <用户名>
  - 创建用户并将其加入到用附属用户组，每个用户可以有多个附属用户组（常用）：useradd -G <用户组名> <用户名>

- 更改用户配置
  - 添加用户的附属用户组： usermod -a -G <用户组名> <用户名>
  - 更改用户主用户组：usermod -g <新用户组名> <用户名>

- 删除
  - 将用户从用户组中删除：gpasswd -d <用户名> <用户组名>
  - 删除用户：userdel <用户名>





## linux发送http请求

```
curl -H "Content-Type: application/json" -X POST -d '{"data": {"fileType": "TXT", "filename": "BATCH2020060300173749KPBS1"}, "missionId":"0500", "subMissionId":"0300", "sysTrxId": "BATCH2020060300173749"}' "http://6.6.6.6:8888/55668899"
```