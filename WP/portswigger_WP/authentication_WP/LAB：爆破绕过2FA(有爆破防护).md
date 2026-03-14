# LAB：爆破绕过2FA(有爆破防护)

来源：portswigger

漏洞原理：因为验证码通常都比较短，四位或者六位，爆破起来难度不算太大，通常可以绕过。但是网站通常对爆破都设有防护，那么就可以采取对应的措施：burp turbo intruder工具，python脚本自动化，多session轮换，多IP爆破，高并发攻击，寻找OTP漏洞。

题目描述：

该实验室的双因素认证漏洞存在暴力破解漏洞，你已获得有效的用户名和密码，但无法获得用户的双因素验证码。

victim's credentials:carlos:montoya

解决方案：



