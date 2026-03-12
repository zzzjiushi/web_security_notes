# 伪造xff,referer

```
X-Forwarded-For
X-Real-IP
Client-IP
```

基于 HTTP Header 的访问控制绕过（多条件）

## [原理]

X-Forwarded-For:简称XFF头，它代表客户端，也就是HTTP的请求端真实的IP，只有在通过了HTTP 代理或者负载均衡服务器时才会添加该项

HTTP Referer是header的一部分，当浏览器向web服务器发送请求的时候，一般会带上Referer，告诉服务器我是从哪个页面链接过来的