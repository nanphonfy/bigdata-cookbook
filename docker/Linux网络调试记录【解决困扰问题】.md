yum search vim;

- centos下的vim安装
https://blog.csdn.net/yexudengzhidao/article/details/70194455

yum install vim-enhanced

- 解决虚拟机可以连网但无法ping通的问题  
https://blog.csdn.net/poetic_vienna/article/details/52690809

- CentOS7网络配置和修改网卡名称及常用服务管理命令
https://www.linuxidc.com/Linux/2017-04/143002.htm

- CentOS7 关闭防火墙
https://blog.csdn.net/Post_Yuan/article/details/78603212

- 【VMware 虚拟机如何连接网络】  
https://blog.csdn.net/qq_37131111/article/details/54000029

【网卡要禁用共享，然后在vmnet8配置与虚拟机静态ip一样的网段即可互ping走外网
eg.虚拟机192.168.25.146，则vmnet8的网段可以配置192.168.25.16，不走DHCP】

出现的问题：可以用xshell连接虚拟机，但是虚拟机ping不通外网，安装不了任何软件。

①计算机点击右键选择管理，进入管理选择VM开头的服务如果没有开启的话就右键开启。（我打开了VMVare workstation service，之前这个漏打）；  
②点击更改适配器，查看虚拟机的虚拟网卡启动没有，没有启动的话右键点击启动；  
③网卡开启后设置ip地址，此处设置的ip和本机的ip没有关系，设置成你虚拟机里面运行的计算机需要的ip地址网段；  
④打开虚拟机，选择你使用的操作系统打开详情页选择网络适配器，选择NAT模式并选择启动时连接。   

这个问题困扰了我好久，配置都是对的，为什么ping不通，而且还能上网，百思不得姐啊，后来google发现了问题所在，原来是前段时间为了用笔记本开wifi，本地连接开了共享给虚拟无线网卡

将本地连接的共享关闭后（控制面板-->网络和 Internet-->网络和共享中心-->本地连接-->属性-->将下图的第一个选项取消掉），虚拟机就可以ping通外网了

其实如果你正在用自己的wifi不想关掉共享，还有另一种解决方法，那就是笔记本连接wifi，此时虚拟机是通过无线网卡而不是本地连接出去的
