### 问题unkown the request的原因是端口被占。解决端口占用问题：若8082端口被占用

开始运行cmd，或者是window+R组合键（注意需要管理员身份运行），调出DOS命令窗口

2_方法1、netstat -ano|findstr 8082   直接查看被占用端口8082，若此行的PID为8492 2_方法netstat -ano    列出所有端口的情况，找到本地地址列表下的*.*.*.*.8082，若此行的PID为8492 3_方法

taskkill /pid 8492 -t -f    杀死占用进程 3_方法2、tasklist|findstr 8492    查看是哪个进程或者程序占用了8082端口，结果是：qq.exe

taskkill /f /t /im qq.exe 3_方法3、启动任务管理器》打勾 显示所有用户的进程(S)》切换 进程》点击 查看(V)>选择列(U)...》打勾 PID(进程标识符)>点击确定》查看进程的PID为8492，对应映像名称是谁，结果是：**.exe

在任务管理器中选中该进程，点击”结束进程(E)“按钮

按住shift键，点击鼠标右键，可以弹出在此处打开命令窗口

* 1、使用组合键，同时按下【Windows徽标键】+【Ctrl】键加上【左右方向键】，就可以切换不用的桌面；
* 2、同样是按下【Windows徽标键】+【Ctrl】+【D】就可以新建一个虚拟桌面；
* 3、切换到要删除的桌面后按快捷键【Windows徽标键】+【Ctrl】+【F4】，就可以将当前的桌面删除掉；
* 4、按【Alt】+【Tab】组合键可以切换桌面，但默认是【仅限我正在使用的桌面】，进入【设置】-【多任务】选项中可以将其修改为【所有桌面】，就可以在虚拟桌面间进行切换。

### Windows运行常用命令（win+R）
* 1、calc: 启动计算器
* 2、notepad: 打开记事本
* 3、write: 写字板
* 4、mspaint: 画图板
* 5、snippingtool：截图工具，支持无规则截图
* 6、mplayer2: 简易widnows media player
* 7、Sndvol: 音量控制程序
* 8、osk: 打开屏幕键盘
* 9、mstsc: 远程桌面连接
* 10、cleanmgr: 打开磁盘清理工具
* 11、compmgmt.msc: 计算机管理
* 12、cmd.exe: CMD命令提示符
* 13、explorer: 打开资源管理器
* 14、gpedit.msc: 组策略
* 15、logoff: 注销命令
* 16、taskmgr: 任务管理器
* 17、control：控制面板
* 18、control userpasswords2：用户账户控制
* 19、shutdown -s -t 0：立刻关机（时间t参数为0-300秒）
* 20：shutdown -r -t 0：立刻重启
* 21、shutdown -l：注销   --- shutdown /h  休眠命令
* 22、regedit: 注册表编辑器
* 23、mmc: 打开控制台
* 24、services.msc: 本地服务设置
* 25、winver: 检查Windows版本
* 26、PowerShell：提供强大远程处理能力
* 27、devmgmt.msc--- 设备管理器
* 28、diskmgmt.msc---磁盘管理实用程序
* 29、msconfig---系统配置实用程序（工具选项卡里面可以查看一些命令的使用）
* 30、magnify--------放大镜实用程序
* 31、secpol.msc-----本地安全策略

### utilman: 轻松访问
win + e 打开此电脑