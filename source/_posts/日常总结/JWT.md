---
title: JWT原理解析
date: 2019-09-26
categories:
    - 学习
tags:
    - jwt
---

### 什么是JWT
    JSON Web Token(JWT),是一个开放安全的行业标准,用于多个系统之间传递安全可靠的信息

### 为什么使用JWT
* 传统的访问模式 | PC(浏览器)->服务端
    1. 用户登录时发送用户名密码给服务器
    2. 服务器验证通过后服务器会生成一个HttpSession对象，并将对象的ID(sessionID)作为Cookie发送给浏览器
    3. 后面的每次请求浏览器都会带上这个sessionId。服务器通过验证这个sessionId就可以知道客户端是否登录

随着智能手机、微信小程序等用户端越来越多，服务端需要同时支持PC端、APP端、微信小程序的访问。而对于APP端和微信小程序
来说cookie的机制就不太友好，这个时候就需要JWT了

* JWT
    1. 用户登录时发送用户名密码给服务器
    2. 服务器验证通过后服务器会生成一个token串，并将这个token串返回给客户端
    3. 后面的每次请求客户端都要带上这个token串，服务器通过解析这个token串，可以验证请求的合法性
 
 乍一看好像只是用token串替换掉了sessionId,其实主要区别有：
 1. 对应sessionid来说服务端是有保存的，而token服务端是没有保存的。而仅仅是通过算法解析来验证合法性
 2. 传统的sessionid机制实现过于复用户杂，且可能是tomcat等容器默认实现了，要改动也是很不方便的。所以用token

### token串解析
```text
eyJhbGciOiJIUzI1NiJ9
.eyJzdWIiOiJsbEBmY2JveC5jb20iLCJwYXNzd29yZCI6ImUxMGFkYzM5NDliYTU5YWJiZTU2ZTA1N2YyMGY4ODNlIiwiaWQiOjIsImV4cCI6MTU2OTQ5OTc1NSwiaWF0IjoxNTY5NDk3OTU1LCJqdGkiOiIzMGIxZjJiOC1iNDE4LTQ4ODAtYjY4OC02OTY0NmYxMDg3MmIiLCJ1c2VybmFtZSI6ImxsQGZjYm94LmNvbSJ9
.YE-SYjyThmogsF5Xj5sgvE9rNzAt4t3z19iDmi2Q2EI
```
* 第一段为头部信息，指明了加密算法等信息
* 第二段为用户自定义信息，通常放username等信息
* 第三段为签名，使用第一段里面指定的签名算法（默认是 HMAC SHA256）按照下面的公式生成签名。主要用于验证token的合法性
    * HMACSHA256( base64UrlEncode(header) + "." + base64UrlEncode(payload),secret)
    * 由于签名的密钥只有服务端有，所以别人没有办法伪造和篡改token