# 米斯特白帽培训讲义 实战篇 CIDP

> 讲师：[gh0stkey](https://www.zhihu.com/people/gh0stkey/answers)

> 整理：[飞龙](https://github.com/)

> 协议：[CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/)

目标是`http://yjs.cidp.edu.cn/`。

我们扫描一下，找到了这个系统：`http://211.71.233.67/`。

![](http://ww4.sinaimg.cn/large/841aea59jw1fb4as0u6noj20vs0hpq4j.jpg)

这个站点使用手动测试和扫描器扫描都没有发现什么漏洞。于是我们注意到了有个“Android 教务查询系统”，果断把它下载下来安装。

我们在电脑上打开 Burp，使用 Burp 抓手机的包。我们需要保证电脑和手机连一台路由器。然后，电脑上使用`ipconfig`查看内网的 IP：

![](http://ww1.sinaimg.cn/large/841aea59jw1fb4as3rii1j20iu0c93zc.jpg)

IP 为`192.168.1.100`。

然后在手机上设置代理，填写之前的 IP 地址，点击保存。

![](http://ww2.sinaimg.cn/large/841aea59jw1fb4as9jm6jj20e80e0t94.jpg)

打开 APP，主界面是这样。

![](http://ww4.sinaimg.cn/large/841aea59jw1fb4asc7codj20e50nh0tz.jpg)

我们点击“公共查询”，然后找个东西随便测试一下。比如说，我们在里面的班级查询功能里面查询`123456`。

![](http://ww4.sinaimg.cn/large/841aea59jw1fb4asfb1cgj20ea0npab1.jpg)

然后 Burp 中会发现封包，我们把它发送到 Repeater。

![](http://ww2.sinaimg.cn/large/841aea59jw1fb4ashqtq0j20rs0maacm.jpg)

因为我们是随便填写的，它都能返回`true`，我们猜测这里面可能有个 SQL 注入。只不过以前的 HTTP 正文是 urlencode 格式的，现在变成了 XML 格式。

我们改为`123456 and 1=1`，它返回`true`：

![](http://ww2.sinaimg.cn/large/841aea59jw1fb4askwa1ij20qx07pgmy.jpg)

然后改为`123456 and 1=2`，也返回`true`：

![](http://ww4.sinaimg.cn/large/841aea59jw1fb4asnh1irj20qz07qwft.jpg)

然后我们在前面加个单引号，发现它返回了`false`：

![](http://ww4.sinaimg.cn/large/841aea59jw1fb4asq7eh2j20qy0820u3.jpg)

那就说明可能存在注入。之后我们将请求封包复制下来，保存为`xxx.txt`，放到 SQLMap 相同目录下。

之后在命令行中执行：

```
sqlmap.py -r xxx.txt
```

![](http://ww1.sinaimg.cn/large/841aea59jw1fb4astyyfpj20iv0c6ta9.jpg)

![](http://ww1.sinaimg.cn/large/841aea59jw1fb4aszmy06j20ir0cctam.jpg)

![](http://ww1.sinaimg.cn/large/841aea59jw1fb4aswporpj20ir0c6tas.jpg)

是存在注入的。

然后使用`--dbs`列出所有数据库：

![](http://ww2.sinaimg.cn/large/841aea59jw1fb4at3ycqxj20is0by75h.jpg)

![](http://ww1.sinaimg.cn/large/841aea59jw1fb4at9f05pj20iv0c6taw.jpg)

![](http://ww1.sinaimg.cn/large/841aea59jw1fb4atbx3ruj20ir0c674t.jpg)

这样就挖到了这个站点的漏洞。
