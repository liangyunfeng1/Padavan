新增了对xtls的支持，源码来自于xumng123  
https://github.com/xumng123/rt-n56u  
感谢vb1980持续对代码做出的测试和改进，他在原本的基础上新增了对xray的编译，而我之前就直接替换了v2ray的bin，所以是殊途同归啦  
https://github.com/vb1980/Padavan-KVR  
  
# Padavan
基于hanwckf,chongshengB以及padavanonly的源码整合而来，支持kvr  
编译方法同其他Padavan源码，主要特点如下：  
1.采用padavanonly源码的5.0.4.0无线驱动，支持kvr  
2.添加了chongshengB源码的所有插件  
3.其他部分等同于hanwckf的源码，有少量优化来自immortalwrt的padavan源码  
4.添加了MSG1500的7615版本config  
  
以下附上他们四位的源码地址供参考  
https://github.com/hanwckf/rt-n56u  
https://github.com/chongshengB/rt-n56u  
https://github.com/padavanonly/rt-n56u  
https://github.com/immortalwrt/padavan  

已测试的机型为MSG1500-7615，JCG-Q20，CR660x  
  
固件默认wifi名称PDCN及PDCN_5G  
wifi密码1234567890  
管理地址192.168.123.1  
管理账号密码都是admin  

新增了对xtls的支持，源码来自于xumng123
https://github.com/xumng123/rt-n56u
感谢@vb1980 持续对代码做出的测试和改进，他在原本的基础上新增了对xray的编译，而我之前就直接替换了v2的bin，所以是殊途同归啦
https://github.com/vb1980/Padavan-KVR

本文涉及的Padavan源码如下：
https://github.com/hanwckf/rt-n56u
https://github.com/chongshengB/rt-n56u
https://github.com/padavanonly/rt-n56u
https://github.com/immortalwrt/padavan
其中hanwckf的源码最先支持了7915无线芯片，也就是支持了wifi6的机型比如CR660x和JCG Q20/Q10 Pro
padavanonly在hanwckf的基础上增加修改出了7615/7915对kvr的支持
chongshengB的源码具有一些别人没有的插件，使用比较方便
immortalwrt在一些细节上有优化

将他们四个人的源码融合起来，虽然是一件复杂且工作量大的事，毕竟有75000+个文件，但这件事里面并非有多少技术含量
主要是要感谢hanwckf在无线驱动和机型适配方面，以及chongshengB在插件方面，还有padavanonly在kvr方面的探索与开源

刚开始融合的时候想的比较简单，以为源码各个部分的关系比较分明，可能就是user文件夹下添加插件源码更改总的Makefile，然后对应的在www里添加前端的asp就行，
结果经历了几次古怪的失败后（无法启动，页面显示异常，无线异常），这才开始静下心来仔细看每一份源码之间的区别

然后发现不同源码之间的差异可真是大。。有些是写法不同目的相同，有些则是实现的方法都不一样了
我们最终的目的是要有padavanonly的kvr，要有chongshengB的插件，以及hanwckf的其他部分

工具链都是一样的，区别都在trunk文件夹：
configs文件夹完全采用padavanonly，因为config文件里包含了对kvr的编译开关
libc文件夹完全相同
libs文件夹完全采用hanwckf，因为他所采用的各个lib的版本都最新，我比较喜欢追新
linux-3.4.x文件夹完全采用immortalwrt，新增了闪存型号的支持和MMC/SD卡的支持
proprietary文件夹完全采用padavanonly，此处是无线驱动部分，因为要支持kvr就需要修改无线驱动，这里只能用padavanonly的
vendors文件夹比较特别，chongshengB的源码里这个文件夹包含了很多无线驱动方面的内容，比如各种lna和pa搭配的eeprom文件，但是hanwckf/padavanonly是没有的，无线驱动方面一概以padavanonly的为准
但是希望有对vendors这个文件夹比较了解的朋友能给分析一下，我还没更细致的去理解这部分
build_firmware_modify需要采用padavanonly版本（指定回退的无线驱动版本）并从chongshengB版本复制插件添加部分
trunk的Makefile采用chongshengB版本，因为包含了go的编译
trunk文件夹下其他文件均可采用padavanonly版本

插件是都集中在user文件夹的，所以user文件夹以chongshengB为基础添改：
        chnroute修改Makefile不需要每次重新下载（可以不改，我只是为了自己编译不同固件方便）
        dnsmasq可替换为hanwckf的升级版本
        dropbear可替换为hanwckf的升级版本
        frp修改Makefile不需要每次重新下载编译（可以不改，我只是为了自己编译不同固件方便）
        htop可替换为hanwckf的升级版本

        httpd需要以chongshengB的为基础按照hanwckf+padavanonly的修改
                \user\httpd\ralink.c采用hanwckf的
                \user\httpd\variables.c添加7915部分及两个80211KV，80211R
                \user\httpd\web_ex.c添加7915部分

        iptables可替换为hanwckf的升级版本，同时要替换miniupnpd，有指定依赖关系

        rc需要以padavanonly的为基础按照chongshengB修改
                \user\rc\rc.c增加插件脚本运行
                \user\rc\rc.h增加插件定义
                \user\rc\services.c增加插件服务
                \user\rc\watchdog.c增加插件看门狗

        scripts需要以padavanonly的为基础按照chongshengB修改
                \user\scripts\autostart.sh从chongshengB添加
                \user\scripts\copyscripts.sh从chongshengB添加
                \user\scripts\dev_init.sh增加对ld.so.conf的定义
                \user\scripts\ld.so.conf从chongshengB添加
                \user\scripts\Makefile增加autostart.sh,copyscripts.sh,ld.so.conf
                \user\scripts\mtd_storage.sh注释掉gfwlist部分

       share需要以padavanonly的为基础按照chongshengB修改
                \user\shared\cflags.mk增加插件部分
                \user\shared\defaults.c增加插件部分
                \user\shared\notify_rc.h使用chongshengB的版本

        v2修改Makefile直接跳过编译采用二进制文件，可用xray（可以不改，我只是为了自己编译不同固件方便）

       www需要以chongshengB的为基础按照padavanonly修改
                \user\www\n56u_ribbon_fixed\Advanced_WAdvanced_Content.asp增加7915和kvr
                \user\www\n56u_ribbon_fixed\Advanced_WAdvanced2g_Content.asp增加7915和kvr
                \user\www\n56u_ribbon_fixed\Advanced_Wireless_Content.asp采用padavanonly的
                \user\www\n56u_ribbon_fixed\Advanced_Wireless2g_Content.asp采用padavanonly的
                \user\www\n56u_ribbon_fixed\Advanced_WMode_Content.asp采用padavanonly的
                \user\www\n56u_ribbon_fixed\Advanced_WMode2g_Content.asp采用padavanonly的
                \user\www\n56u_ribbon_fixed\wireless.js采用padavanonly的
                \user\www\n56u_ribbon_fixed\wireless_2g.js采用padavanonly的
                其他的js文件有互相的关联，所以轻易不要替换或者改动，很容易出错导致显示问题

       Makefile需要以chongshengB的为基础按照padavanonly增加ralinkiappd，也就是控制kvr的程序

最后再来回顾一下，如果要添加有前端页面的插件，就需要修改httpd，rc，scripts，share，www和总的Makefile，因为一个插件包括了前端显示的参数，系统注册的服务，运行时的脚本，系统内的参数，前端的asp页面以及编译开关，是这么多内容构成一个可视化插件的整体

增加适配机型的config就简单的多了，从chongshengB的config里复制出来MSG1500-7615的部分，然后跟类似机型对比修改，这里比较麻烦的就是闪存定义这里要对比一个nand闪存的，而无线部分又要对照一个7615的，还有usb部分，基本上都是些不是很要紧的增改，前面的事都做了，这里简直是毫无难度

融合出来的源码已经上传到github，也有三四个坛友都编译通过并测试使用了，想必没有什么大问题，有兴趣的朋友可以自己融合自己需要的部分，或者直接用融合后的代码
https://github.com/keke1023/Padavan
另：之前我说B70编译出来5G有问题，可能问题出在我这台机子本身的硬件，也许7612是可以用的，等待我再找个机子验证吧
