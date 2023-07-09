2022-08-23
emptysuns

甲骨文大善人给各位来发福利啦
请适度食用，如果因为高强度cpu占用导致封号和博主无关，不是博主把你号封了，请不要报复我蟹蟹～（请自行承担责任，我用了一天没事）
引言
最近找app远程调试环境，最后选择了docker启动容器的比较方便，支持ADB远程连接。使用scrcpy连接可以提供桌面环境，突然发现这个镜像的架构支持Arm64，我还有好几个吃灰的甲骨文，遂试了试体验良好，可以当成一个云手机来挂机。

Arm64架构并不是必须的，就像我如上说明，该镜像支持Amd64，如果你认为你的机器性能不错就可以试试。

更新：如何获得一个web版adb远程桌面 https://blog.imoeq.com/scrcpy-run-a-android-web-page/

介绍
甲骨文Arm使用的CPU给的性能很足，4c的geekbench5跑分有足足3400分左右，参考当前最流行的手机处理器芯片骁龙870 geekbench5 3100分来说是非常良心的了。使用它来虚拟一个云手机来说性能充足。

我在这里选择的是Redroid，ReDroid（Re mote an Droid）是一个 GPU 加速的 AIC（Android In Container）解决方案。Docker您可以在 Linux 主机（Docker, podman, k8s etc.）中启动许多实例。ReDroid同时支持arm64和amd64架构。 ReDroid适用于云游戏、VMI（虚拟移动设备）、自动化测试等。

根据该镜像描述，对云游戏有很好的支持，符合我们的需求，所以这里直接使用它启动容器。

你也可以用来配置python selenium做自动化，因为有root权限，也能用于app开发调试，并不一定拿来挂机游戏，说到底这是我的需求。

接下来配置过程中，最麻烦的不是让容器启动，而是为了你连接桌面过程中更加安全可靠而做的努力，如果你仔细阅读完是没有问题的。

性能/资源占用率参考
看不清图中文字，右键标签打开放大看

总体来说还可以，能不用软解就不用，硬解性能测试着比软解更好，1080p60能维持这个占用率，跑些非大型游戏挂机脚本挺不错的，白嫖还想要什么飞机？（这个测试的是redroid13，如果是8.1占用率应该更低）


使用
下方测试系统是oracle 原生 ubuntu 20.04，其他系统我没有一一测试，没什么区别。

一· 检查内核

首先查看一下你的内核版本是否>=5.0,根据介绍，如果内核在此版本之下，许多指令无法适配，为了不出错还是升级一下内核，如果不升级内核该镜像issue也给出解决方案，我懒得看，有兴趣自己研究去。

uname -r
5.15.0-1013-oracle #这里最好>=5.0
二. 安装模块

apt install linux-modules-extra-`uname -r`
modprobe binder_linux devices="binder,hwbinder,vndbinder" #进程通信模块
modprobe ashmem_linux #内存共享模块
 
#后两条命令不提示错误 / Enter后没有任何反应说明启动成功
上面两个模块都是Android运行所必须的依赖，必须启动成功，否则虚拟化失败，如果这里出错请不要下面的步骤，请自己先解决模块启动失败的问题，如果你是用的oracle 原生镜像，这里应该是没问题的，如果是已经通过网络dd系统发生的错误，这里不给出解决办法请自己查询。(oracle 自带的ubuntu 20.04正常，其他自测)

三. 安装docker环境

怕有些人连docker是什么东东都不知道，这里用docker提供的脚本一键安装环境

curl -fsSL https://get.docker.com | bash
四. 拉取docker镜像并启动容器

docker run -itd \
    --memory-swappiness=0 \
    --privileged --pull always \
    -v /root/test/data:/data \
    -p 55555:5555 \
    redroid/redroid:13.0.0-latest \
    androidboot.hardware=mt6891 ro.secure=0 ro.boot.hwc=GLOBAL    ro.ril.oem.imei=861503068361145 ro.ril.oem.imei1=861503068361145 ro.ril.oem.imei2=861503068361148 ro.ril.miui.imei0=861503068361148 ro.product.manufacturer=Xiaomi ro.build.product=chopin \
    redroid.width=720 redroid.height=1280 \
    redroid.gpu.mode=guest \
    --rm
解释这条命令：

run -itd 意思是后台运行
--memory-swappiness=0 禁止swap，防止I/O成为性能瓶颈，甲骨文Arm给的内存挺多的，满配24g内存，不用担心用得完
--privileged 启动特权模式，使用该参数，container内的root拥有真正的root权限。防止奇奇怪怪的毛病
-v /root/test/data:/data 映射目录，你可以把/root/test/data换成其他的目录，方便你备份
-p 55555:5555 映射ADB端口，这个我选择是55555，你可以选择其他端口，最好修改一个，防止扫描器扫描
redroid/redroid:13.0.0-latest(底层是Android13) 使用的镜像tag，这里我选择这个tag，群友说redroid:8.1.0-arm64(底层是Android8)这个资源占用最少，性能最好，出问题概率最低。你可以替换成这个我没试过。其他版本: https://hub.docker.com/r/redroid/redroid/tags （切换版本或者重置容器时，记得删除原来的映射目录或者选择其他的目录，不然会出错rm -r /root/test/data）
androidboot.hardware=mt6891 ro.secure=0 ro.boot.hwc=GLOBAL ro.ril.oem.imei=861503068361145 ro.ril.oem.imei1=861503068361145 ro.ril.oem.imei2=861503068361148 ro.ril.miui.imei0=861503068361148 ro.product.manufacturer=Xiaomi ro.build.product=chopin 这部分是白嫖群友的，据他说是红米Note10的参数，这个是为了模拟一下手机型号，某些游戏检测到不合法的手机型号会封号，感兴趣自己抓一下build.prop参数，替换掉。方法：https://www.jianshu.com/p/098b8809d85d
redroid.width=720 redroid.height=1280 这个是设置分辨率，对于这种远程ADB来说，这个参数是最好的，如果非常卡可以适当调小
redroid.gpu.mode=guest 强制选择这个容器的软解（如果不加这条参数就是硬解），区别在于：软解更占资源，奇奇怪怪的问题少；硬解性能高、资源占用率低。能用硬解就用硬解，不同版本硬解/软解问题不同，硬解不能用时再开软解，一般没多大问题。欢迎下方评论区反馈，这个就自测了
--rm 最后加上stop容器时删除容器，省得手动删除，因为有目录映射出来，所以重新启动数据也会保存在那个映射目录里
五. 输入如上命令后查看运行状态

root@ucbro:~/a# docker ps
CONTAINER ID   IMAGE                           COMMAND                  CREATED          STATUS          PORTS                                         NAMES
85ce1da669c5   redroid/redroid:13.0.0-latest   "/init qemu=1 androi…"   53 minutes ago   Up 53 minutes   0.0.0.0:55555->5555/tcp, :::55555->5555/tcp   suspicious_panini
 
#查看是否如上启动成功
六. 安全性防护

注意: 如果您觉得步骤六、七太麻烦了，您可以选择使用web反代来保障安全和加速连接速度，可以跳过下面最麻烦的两步(推荐):

https://blog.imoeq.com/scrcpy-run-a-android-web-page/

adb连接 <-> netch <-socks5-> v2rayN/clash <-> 墙 <-> 梯子server <-> 甲骨文Arm里的docker容器（只允许梯子ip访问）

这里有点麻烦，这一步为了给不能鉴权的容器限制登录ip只有你的梯子，如果跳过这步你的服务器就会成为人人可上车的肉B器。=><=

最近一段时间甲骨文的ip被gfw干到生不如死，你在大陆内很难直接通过adb连接到容器，因为adb协议设计就是用来本地调试的，在远程调试方面没有一点安全性，即不能直接过墙，过墙秒死。

这属于远程桌面形式，有人也会通过这种形式去做一些wffz的事情，导致老大哥早就考虑到这个问题了，早就对adb远程连接做了限制，而且这协议特征明显很容易识别，这个协议设计就不是为了这样用的。

而且如果用这个镜像默认就是允许adb调试，无需授权，也导致只要知道端口是什么，人人都能连接上，一个很简单的扫描就能让你的容器成为肉鸡。

解决办法就是用一个线路好的跳板机去连接你的容器，做好你到跳板机之间的加密。说人话就是通过你的梯子去连接容器的adb端口，并且让此adb端口只允许你梯子的ip访问。

是不是有点麻烦，麻烦就对了，要怪就怪老大哥掐你网线吧。

我们这里用netch做例子，首先在这里解决服务器端，客户端netch如何连接看下面：

查看你网卡的名称，方便修改iptables
#下面有很多网卡名称，选择你上网用的，比如arm ubuntu 20.04里就是 enp0s3
root@ucbro:~# ip add
 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9000 qdisc mq state UP group default qlen 1000
    link/ether 02:xx:17:00:xx:e8 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.1xx/24 brd 10.0.0.255 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 260xxxxxxxxxxxxxxxxxxxxxxxxxc3/128 scope global dynamic noprefixroute 
       valid_lft 4928sec preferred_lft 4628sec
    inet6 fe80::17ff:xxxxxxxxxxxx/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:xxxxxxxxxx76 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fexxxxxxxxxxxxxx76/64 scope link 
       valid_lft forever preferred_lft forever
39: vethe253974@if38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether b6xxxxxxxxxxxxx9 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe8xxxxxxxxxxxxxxa49/64 scope link 
       valid_lft forever preferred_lft forever
2. 向docker-user里增加iptables规则，只允许你梯子ip访问adb端口，55555改成你的docker映射出的adb端口，1.2.3.4改成你跳板梯子的ip地址，enp0s3改成你上网网卡的名称。

这里建议选择梯子到你的延迟很快，并且梯子到甲骨文的速度很快，因为转发桌面时会有稍许延迟

iptables -I DOCKER-USER -i enp0s3 -p tcp -m conntrack --ctorigdstport 55555 ! -s 1.2.3.4 -j  DROP
这样只有你梯子的ip能够访问你的容器了，其他人会被drop掉

如果使用了netfilter-persistent（甲骨文给的原生ubuntu镜像就带着呢）或者firewall-cmd，请使用下方命令，保存防火墙规则防止重启系统后规则消失：

netfilter-persistent save
firewall-cmd --runtime-to-permanent
七. netch代理adb程序连接该容器

这里不需要打开防火墙，因为docker会自动配置

下载 scrcpy-win64，解压
使用目录里的open_a_terminal_here.bat 在本目录打开一个窗口
> adb.exe connect 你的ip:你的端口
connected to 你的ip:你的端口
 
 #连接成功后会提示成功，如果没有，而且配置过程中没错误的话，说明是网络问题，请自己加密转发这个端口，或者使用境外服务器做跳板连接，adb连接也是被gfw阻断的一部分
如果你是用我如上第六点增加了防火墙规则，这里直接连接是不会连上的，因为只允许你梯子ip访问：

D:\tools\scrcpy-win64>adb.exe connect 1.2.3.4:55555
failed to connect to 1.2.3.4:55555
我们使用v2N和netch做栗子，其他软件（clxsh）同理，这里只做参考

具体做法就是，用v2N成功启动你梯子节点后（推荐hysteria，这种tcp over udp的形式，尽可能减少延迟卡顿），它自动给出一个socks5代理，然后再netch里添加这个socks5代理，选择只代理程序scrcpy目录的adb.exe，然后你的adb流量就会通过梯子转发走过墙了。（也可以将节点在netch里直接启动，但是我这里用的是hysteria它不支持，我只能这样做了，adb是无法直接over socks的）下面具体步骤：







到这里你就完成adb连接和安全性保障了，记得netch模式选择你刚才创建的模式，接下来启动桌面。

先通过adb连接到容器，才能使用scrcpy使用桌面环境：

D:\tools\scrcpy-win64>adb connect 1.2.3.4:55555
connected to 1.2.3.4:55555
 
#出现这个说明连接成功，根据上面防火墙规则，这里只能通过你的梯子连接，也就保障安全性
下面通过scrcpy.exe连接桌面

如果觉得桌面不流畅，说明延迟高。可选参数：

-b384k :比特率 384kbps。严重影响画质
-m960 :将屏幕大小调整为正常大小的一半（假如你分辨率为1920）。极大地影响延迟
例如这样用scrcpy.exe -b384k -m960

 
D:\tools\scrcpy-win64>scrcpy.exe
scrcpy 1.24 
D:\tools\scrcpy-win64\scrcpy-server: 1 file pushed, 0 skipped. 43.6 MB/s (41159 bytes in 0.001s) 
[server] INFO: Device: Xiaomi redroid13_arm64 (Android 13)
INFO: Renderer: direct3d
INFO: Initial texture: 720x1280
 
#adb连接成功后，直接cmd窗口运行,这个程序就会打开一个安卓的窗口
或者使用AutumnBox连接adb管理(同样需要将AutumnBox程序目录加入netch，这里就不重复了，觉得麻烦直接用netch接管所有流量也行)：


八. 成功截图




安装via浏览器测试

什么都不运行3cpu空载使用率

登录老家网站使用率
八. 停止

#client
D:\tools\scrcpy-win64>adb disconnect
disconnected everything
#server
docker stop <CONTAINER ID>#因为加了rm，所以停止即删除，数据在映射目录里
常见问题
为什么我直连运行成功了却连不上？/为什么我直连连接上这么卡？/为什么直连连接一段时间就断开卡住了？

这些问题都有一个答案，就是老大哥正在看着你，你应该感谢没有出口白名单，而不是在这里抱怨。

为什么我启动失败？

不知道，如果你仔细阅读如上简介，应该能知道问题在哪儿，如果不知道再读一遍

有封号风险吗？

有，就算你吃灰也有，还不如尽可能用用呢，请适度食用，只要不是作死cpu占用率七八十24小时不间断，甲骨文还是很慈悲的。

资源占用过高，怕封号怎么办？

我的建议是别高强度挂机，封号和我无关，我只是介绍，并没有按着你的手去让你启动容器并运行高资源占用程序，更没有去封了你的号。

结语说明
如果你单单运行一个容器不挂游戏的话，基本是不吃资源的，这也与这个镜像优化好有关，最好使用4gb以上的Arm运行，防止内存不够，这点内存对甲骨文来说是很简单的啦。

但这里也有不足的地方，比如这个容器无法加密，无法对adb连接进行授权，需要通过iptables等防火墙进行限制只有你本地的ip进行访问，或者使用VPN的形式，设置这个容器只能内网访问，通过VPN登入内网，这就靠大家自行发挥了，不然被扫到就是纯纯肉鸡。

转载请说明出处

参考
https://hub.docker.com/r/redroid/redroid

某位不知名的群友
