# 利用Shellshock漏洞实现盲SSRF攻击 WP

来源：portswigger

题目描述：

本网站使用分析软件，当产品页面加载的时候，该软件会获取referer标头中的指定的URL

要完成本实验，请利用此功能对内部服务器（位于192.168.0.x特定范围内的8080端口）发起盲SSRF攻击。在盲攻击中，使用shellshock payload针对内部服务器，以窃取操作系统用户的名称。

https://portswigger.net/academy/labs/launch/d453c6b39efc515eb918548d6ff29ddde86d0d0361ab5453b626309de0e3a958?referrer=%2fweb-security%2fssrf%2fblind%2flab-shellshock-exploitation



### Solution：

