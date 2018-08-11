name:inverse
class: center, middle,inverse
layout: true

---

# HTTPS

Fighting

---

> HTTPS（全称：Hyper Text Transfer Protocol over Secure Socket Layer），是以安全为目标的 HTTP 通道，简单讲是 HTTP 的安全版。

> HTTPS = HTTP + SSL/TLS

---

# 几个问题

???

带着问题出发

---

- 为什么需要 HTTPS

--

- HTTPS 是如何保证安全的

???

为何不能被劫持

--

- Fiddler/Charles 能抓 https 的包么？

--

- HTTPS 是否 100%安全？

???

Strict-Transport-Security: max-age=31536000

添加到 HSTS 列表 https://hstspreload.org/ google 维护

[HTTP Strict Transport Security](https://developer.mozilla.org/zh-CN/docs/Security/HTTP_Strict_Transport_Security)

## 本地添加测试 chrome://net-internals/#hsts

# HTTP

???

超文本传输协议（HTTP，HyperText Transfer Protocol)是互联网上应用最为广泛的一种网络协议。

http 弊端：不安全，等同于裸奔

类似的，我们常用来接收的短信验证码，如果是用的 2G 或 3G 网络，那么也完全是裸奔的。
而 4G 的 TD-LTE FDD-LTE 是比较安全的

当然，目前是有技术手段可以让支持 4G 的手机降级的

---

# HTTP 劫持

---

![MITM](./images/mitm.jpg)

???

如果让你来改进 http 协议，如何让其变安全？

---

# 加密

---

- 对称加密

--

- 非对称加密

???

编码不是加密： base64

散列也不是加密：[md5](https://md5jiami.51240.com/)、SHA 家族

对称加密：AES、DES、3DES 速度快、密钥唯一

非对称加密：RSA 速度慢、有一对密钥

---

![对称加密](./images/duicheng.png)

???

对称加密很方便，速度也很快，但由于密钥唯一，安全性得不到保障

[《模仿游戏》](https://movie.douban.com/subject/10463953/)

---

![非对称加密](./images/feiduicheng.png)

???

非对称加密有利于单向通信及签名。

1. 公钥加密的信息，只有私钥可以解，中间人无法查看
2. 私钥加密的信息，所有人都可以通过公钥解，但无法伪造

---

![对称加密的局限](./images/duicheng1.png)

???

[大图](https://user-images.githubusercontent.com/8401872/28902322-c1ac4bdc-7830-11e7-88d7-0da60428cd19.png)

---

![对称加密的局限](./images/duicheng2.png)

???

[大图](https://user-images.githubusercontent.com/8401872/28902323-c1b60708-7830-11e7-8280-03d2bb80b370.png)

---

# 中间人攻击

---

![中间人攻击](./images/zhongjianren.png)

???

[大图](https://user-images.githubusercontent.com/8401872/28902455-8afb96b4-7831-11e7-9a53-2c8474a963af.png)

---

![数字签名](./images/ca.png)

???

公钥放在数字证书中

[大图](./images/ca1.png)

---

数字证书

???

数字证书是什么？为什么可以证明服务器身份？

---

> 政府有一对公钥与私钥，现在先对姓名、性别、年龄、住址、签发单位、证件有效期、公钥做一次 hash，然后使用私钥对该 hash 值做一次加密得到最后一项签名

![国家证书](./images/zhengshu0.png)

???

政府签了这张证书给你，你能信任这个证书是中国政府签的么？

国家政府的证书必须无条件被任何国家信任，国家的信誉做保障，离线保存私钥

---

![省级证书](./images/zhengshu1.png)

???

这个签名是由国家私钥加密得来，叫中间证书

---

![个人证书](./images/zhengshu2.png)

???

这个签名是由湖南省私钥加密得来

---

![handshake](./images/handshake.png)
???

[大图](./images/handshake1.png)

这个握手过程是否安全？

时序图源码

```
title HTTPS Handshake

Client->Server: 1.协议版本\n2.加密方式列表\n3.支持的压缩算法\n4.random_C
Server->Client: 1.确认协议\n2.确认加密方式\n3.确认压缩方式\n4.random_S\n5.数字证书
note left of Client: 验证证书
note left of Client: 产生 Pre-master
note left of Client: enc_key=Fuc(random_C, random_S, Pre-Master)
note left of Client: enc_key 加密之前的握手信息得到 encrypted_handshake_message
Client->Server: 1.用证书携带的公钥加密后的 Pre-Master \n2.参数协商完成通知\n3.encrypted_handshake_message
note right of Server: 用私钥解密得到 Pre-Master
note right of Server: enc_key=Fuc(random_C, random_S, Pre-Master)
note right of Server: 用 enc_key 解密 Client 传来的 encrypted_handshake_message 并验证
note right of Server: 用 enc_key 加密之前的握手信息得到新的 encrypted_handshake_message
Server->Client: 1.参数协商完成通知\n2.encrypted_handshake_message
note left of Client: 用 enc_key 解密 Server 传来的 encrypted_handshake_message 并验证
```

https://www.websequencediagrams.com/

---

如何抓包 HTTPS

???

我们经常用 Fiddler 抓包，那么它是如何抓取 https 包的呢？

---

![https代理](./images/proxy.png)

???

[大图](./images/proxy1.png)

- cnnic 根证书被吊销 [月光博客](http://www.williamlong.info/archives/4192.html)
- github 被劫持事件 （wosign 证书被吊销一段时间）

- 12306 的自签证书

---

证书类型

---

DV、OV、EV

---

- g 全局搜索
- i 不区分大小写
- m 多行模式
- u unicode
- y 粘性匹配

???

```javascript
'123abc456def789'.match(/\d+/); // 全局搜索案例

'aaaaa'.match(/A/); // 忽略大小写
`aa 
`.match(/^\w+$/); // 多行匹配

// 粘性匹配
var re = /\d{2}/y;
re.lastIndex = 2;
re.exec('12!34');

// 对比全局匹配
var re = /\d{2}/g;
re.lastIndex = 2;
re.exec('12!34');
```

---

# Backtrack

--

# 回溯

???

```javascript
/ab{1,3}bbc/.test('abbbc');
```

```javascript
/".\*"/.test('"abc"de');
```

```javascript
/^\d{1,3}?\d{1,3}$/.test('12345');
```

- [示例 1](./images/huisu.png)

- [示例 2](./images/huisu1.png)

- [示例 3](./images/huisu2.png)

---

### 回到问题

- 匹配手机号
- 验证密码强度
- 驼峰转连字符风格
- 添加千分位分隔符
- 匹配形如 abba 的字符串

--

- 不匹配形如 abba 的字符串
- 验证一个字符串是否至少包含三个数字

???

```javascript
// 匹配手机号
/^((?!86-)\d+-\d{6,}|(86-)?1\d{10})$/.test('12456454542');
// 验证密码强度
const reg = /((?=.*\d)(?=.*[a-z])|(?=.*\d)(?=.*[A-Z])|(?=.*[a-z])(?=.*[AZ]))^[0-9A-Za-z]{6,12}$/;
const reg1 = /(?!^\d{6,12}$)(?!^[a-z]{6,12}$)(?!^[A-Z]{6,12}$)^[0-9A-Za-z]{6,12}$/;
// 驼峰转连字
'AsdfsdBsdasadC'.replace(/\B(?=[A-Z])/g, '-').toLowerCase();
'AsdfsdBsdasadC'.replace(/\B([A-Z])/g, '-$1').toLowerCase();
// 添加千分位
'123212312312312'.replace(/(?!^)(?=(\d{3})+$)/g, ',');
```

[不匹配 abba 类型 demo](./demo/1.html)

```javascript
/^(?!.*(.)(.)\2\1)/.test('312312abbdewr23423');
```

#### 验证字符串是否至少包含 3 位数字

```javascript
/(\D*\d){3}/.test('dasdas12');
```

---

```
    console.log("Thanks!");
```
