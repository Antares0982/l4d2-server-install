# l4d2-server-install
notes for installing l4d2 server on linux(Debian)

## 安装代理
中国大陆需要安装代理才能下载求生之路服务端(如果服务器在墙外，游戏会有严重的丢包，不推荐)。参考https://github.com/Antares0982/v2ray-linux-install-notes 安装v2ray客户端以及设置polipo。

更多详情参见[v2ray-linux-install-notes](https://github.com/Antares0982/v2ray-linux-install-notes)

## 下载dedicated server
浏览https://linuxgsm.com/lgsm/l4d2server/ ，点左边的Install。按照说明添加用户，设置一个强密码（重要！不能用root权限做这些操作，也不能设置和用户名一样的密码！！！不然等着服务器被人拿去挖矿= =）
```
adduser l4d2
```
切换用户并使用脚本傻瓜安装
```
su l4d2
cd
wget -O linuxgsm.sh https://linuxgsm.sh && chmod +x linuxgsm.sh && bash linuxgsm.sh l4d2server
./l4d2server install
```
在这一步中可能会遇到需要安装一些依赖，按照它的提示安装就好了。

## 安装sourcemod，metamod，l4dtoolz，tickrate enabler
进入游戏文件夹
```
cd /home/l4d2/serverfiles/left4dead2
```
浏览[sourcemod官网](https://www.sourcemod.net/)，点击左边的stable builds，找到中间的linux系统标志，右键复制链接，用wget下载这个链接
```
wget https://sm.alliedmods.net/smdrop/1.10/sourcemod-1.10.0-git6478-linux.tar.gz
```
解压
```
tar -xzvf sourcemod-1.10.0-git6478-linux.tar.gz
```
删除压缩包
```
rm sourcemod-1.10.0-git6478-linux.tar.gz
```
用同样的方法安装metamod，[官网](http://www.metamodsource.net/)

安装l4dtoolz，先用自己的电脑[下载](https://forums.alliedmods.net/attachment.php?attachmentid=122230&d=1373147952)并改名为l4dtoolz-1.0.0.9h.zip（把括号去掉就行，名字无所谓）。打开xftp，建立l4d2这一用户的远程连接（如果不是用这个权限的话可能无法运行！），将zip文件传入进入addons文件夹。进入addons文件夹并解压，然后删除压缩包
```
cd addons
unzip l4dtoolz-1.0.0.9h.zip
rm l4dtoolz-1.0.0.9h.zip
```
安装tickrate enabler，同样的在自己电脑[下载](https://forums.alliedmods.net/attachment.php?attachmentid=164400&d=1500874894)，先在自己电脑上解压，打开l4d2的文件夹（另一个是一代游戏用的），把addons里的东西用xftp（l4d2用户）传到addons文件夹下。

## 修改服务器配置
首先找到`/home/l4d2/serverfiles/left4dead2/cfg`目录下的文件`l4d2server.cfg`，如果是用xftp的话建议用notepad++直接打开编辑。（否则就用nano，vi之类）

在最前面添加如下的服务器参数，其中有的部分要自己改。顺序是不影响的
```
hostname "hostname" //这是服务器的名字，可以自行修改

//下面三条参数可以防止被随机玩家加入
sv_allow_lobby_connect_only 0  
sv_tags hidden  //可能可以防止ddos
sv_search_key "43fhye8rvi3h4g35hbi7" //玩家要搜索这个关键词才能找到此服务器。随便滚键盘输一些东西就行了，不必是我这个。

//使用组服务器要加上下面这两条参数
sv_steamgroup  "12345"  //这里要改成你的steam组id，如果没有很可能会被ddos
sv_steamgroup_exclusive 1 //参数0：公共游戏 1：组成员可加入，且可以通过好友的游戏加入 2：仅组成员可加入

//设置地区为亚洲
sm_cvar sv_region "4"

//难度，这一选项会导致每次过关都会重置为该难度。如果要玩别的难度在每次过安全门后要重新投票，不建议使用这个参数
//z_difficulty "Impossible"

//人数调整
sv_maxplayers 8
sv_removehumanlimit 1

//调整服务器tick，可以调整到100。如果调至100可能会造成服务器的卡顿以及崩溃，具体还要看服务器的配置如何
sm_cvar sv_maxrate 0  //0代表无限制
sm_cvar sv_minupdaterate 60 //可以调至100
sm_cvar sv_maxupdaterate 100
sm_cvar sv_mincmdrate 59  //可以调至100
sm_cvar sv_maxcmdrate 60  //可以调至100
sm_cvar sv_client_min_interp_ratio -1
sm_cvar sv_client_max_interp_ratio 2
sm_cvar nb_update_frequency 0.06
sm_cvar net_splitrate 2
sm_cvar fps_max 150
sm_cvar net_splitpacket_maxrate 100000
```
修改sourcemod参数：找到`/home/l4d2/serverfiles/left4dead2/cfg/sourcemod`目录下`sourcemod.cfg`，打开并在最前面添加
```
// 显示支持玩家数
sv_visiblemaxplayers 8

//子弹数量修改，如果要玩纯净服的话将这一段删除
sm_cvar ammo_smg_max 900
sm_cvar ammo_buckshot_max 720
sm_cvar ammo_assaultrifle_max 720
sm_cvar ammo_huntingrifle_max 225 
sm_cvar ammo_shotgun_max 225  
sm_cvar ammo_autoshotgun_max 225
```
管理员设置：找到`/home/l4d2/serverfiles/left4dead2/addons/sourcemod/configs`并编辑`admins_simple.ini`。这个文件是用来设置sourcemod的权限的，`"99:z"`代表拥有全部权限，可以在游戏内管理服务器。注意这个文件的空格非常有问题，文件前的注释掉的例子中，`"STEAM_0:1:16"		"bce"`两个字符串中间的空格或制表符，把它复制下来用，否则很可能读取不到内容。在文件的最后面添加你和你的朋友的steam id（举例）：
```
"STEAM_1:0:12345"		"99:z"
"STEAM_1:1:67890"		"99:z"
"STEAM_0:1:1234567"		"99:z"
```
不知道自己的steam id？打开l4d2，进入游戏，开控制台输入`status`，把上面的信息照抄下来就行了。如果想获取你的朋友的id，只需要和他们在同一个游戏里输入`status`。

编辑启动参数：找到`/home/l4d2/lgsm/config-default/config-lgsm/l4d2server`并编辑`_default.cfg`，第16行的最大人数改为你的服务器设置的人数，并在第20行字符串末尾加入几个新的参数：
```
-insecure -nomaster -tickrate 60
```
第一个参数是防止某些特殊情况下，服务器中魔改过多导致服务器内玩家被VAC封禁。第二个参数是防止被ddos的，非常重要！！！第三个参数是调整tick到60.如果你在l4d2server.cfg里写的是100tick，那么这里的参数60也改为100。如果还想改默认地图的话，也可自行修改，但是其他的参数就不要调整了，尤其是ip和端口。

现在可以安装插件了。将插件传输到`/home/l4d2/serverfiles/left4dead2/addons/sourcemod/plugins`中（用户需是l4d2，否则无法读取）。不需要的插件移到`/home/l4d2/serverfiles/left4dead2/addons/sourcemod/plugins/disabled`中，在plugins文件夹中的插件在服务器开启时一定会启用。但是请注意，不要将原本那些admin相关的插件停用，否则管理员功能将无法使用。

## 关闭polipo服务及其自启动
如果不关闭polipo，服务器所有流量都会走代理，造成一些软件不必要的卡顿。

更多详情参见[v2ray-linux-install-notes](https://github.com/Antares0982/v2ray-linux-install-notes)

## 启动游戏
返回用户文件夹
```
cd
```
启动游戏服务器
```
./l4d2server start
```
查看进程
```
ps aux | grep left
```
可以查看启动参数是否已经正确写入。

服务器运行时可以使用指令
```
./l4d2server console
```
查看游戏控制台。游戏控制台中按下`ctrl+b`，再按下`d`键退出控制台。如果按下`ctrl+c`，游戏服务器会关闭，请不要用这种方法退出控制台。

在控制台中使用
```
sm plugins list
```
就可以看到成功启用的插件，使用
```
sm plugins refresh
```
就可以重新加载plugins文件夹中全部插件（不需要关闭服务器或重启）
关闭服务器使用指令
```
./l4d2server stop
```
重启服务器使用指令
```
./l4d2server restart
```

## 连接到服务器以及游戏内插件控制
首先，连接到服务器的方法是控制台连接公网ip。打开游戏，加载好mod之后控制台输入
```
connect <你的公网ip>
```
如果你的服务器在组服务器栏上显示了，接下来你可能很快会收到被ddos的提示短信，服务器被迫关闭。这是因为某些RPG服玩家的利益被纯净服玩家影响了。如果不想被攻击的话就安安心心的在`l4d2server.cfg`中加上之前说的
```
sv_allow_lobby_connect_only 0  
sv_tags hidden 
sv_search_key "whateverthefxxkis"
```
以及服务器启动参数中加上之前说过的
```
-nomaster
```
进入服务器之后，在对话框中发送 `!admin`就可以打开管理员菜单。如果要玩第三方地图，将地图vpk文件传到`/home/l4d2/serverfiles/left4dead2/addons/`中（注意，用户：l4d2），然后用`!admin`指令切换地图就行了。
