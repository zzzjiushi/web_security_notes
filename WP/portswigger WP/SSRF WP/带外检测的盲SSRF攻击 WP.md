# 带外检测的盲SSRF攻击 WP

来源：portswigger

题目描述：

本网站使用分析软件，当产品页面加载时，该软件会获取referer标头中指定的URL

要完成该实验，请使此功能像burp collaborator发起HTTP请求

https://portswigger.net/academy/labs/launch/47869588c5b299d864f0e0fb23d97cb0ba5c5a686fcdc0de3a03637daf1ba3af?referrer=%2fweb-security%2fssrf%2fblind%2flab-out-of-band-detection



#### Solution ：

根据题目描述，查看页面加载的请求包：
![image-20251221192851060](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251221192851060.png)

也就是说，修改referer指向的URL，查询collaborator生成的域名。

mka7a621p11xgn4c6qhkqzcp2g87wxkm.oastify.com

![image-20251221193344868](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251221193344868.png)

查看 `burp collaborator` 点击 `poll now`

就可以完成实验

 