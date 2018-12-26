# 在Ubuntu上安装中文输入法

> 之前尝试使用Chrome内置输入法，结果真的酸爽  
> 每次打完字等半小时什么水平？ 这得是脾气多好才能用的下去？

### 安装搜狗输入法的惨痛经历

> OS: Ubuntu 16.04 - LTS - GNOME

反正也是按照网上教程走的，安装完毕以后还是不能用，不能用就算了，大家当作无事发生过，然后就系统开始不断弹窗报错，让我删除一个叫什么config的文件，删除完了以后terminal直接不能打开

尝试卸载terminal后重新安装GNOME-terminal，失败，似乎是因为GNOME的全是一整套，OK尝试重新安装GNOME-desktop，这下不仅GNOME-terminal不能打开，然后file也不能打开了。

最后的解决办法是先进Ubuntu官网查找terminal, 然后下载一个deb文件，打开以后是在Ubuntu软件中心，点击安装以后terminal可以重新打开，但是file还是不能打开，算了我就用命令行进入file也没问题

第二天重新开机直接开不了机，重装系统，然后重新配配环境基本一天过去了，我要是早能意识到这件事或许能省下来不少时间。

### 安装IBUS输入法

> OS: Ubuntu 16.04 - LTS

{% hint style="warning" %}
这一段内容因为之前安装已经过了一段时间了，因此具体的安装细节已经记得不是很清楚了，包括IBUS是怎么安装上的，日后如果需要重新安装回来填坑
{% endhint %}

System Settings -&gt; Language Support -&gt; Install Remove Languages -&gt; 选中Chinese先安装上

System Settings -&gt; Text Entry -&gt; 点击左下角的加号 -&gt;  选中Chinese\(Pinyin\)\(IBUS\) -&gt; 退出重新登录



