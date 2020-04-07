---
title: 前端安全-CSRF
date: 2019-01-21 20:12:26
category: posts
tags: [前端安全,CSRF]
---

随着 Web 高速发展, 越来越多的线下业务都转移到线上。Web 程序变得越来越复杂同时安全也成为重中之重。本文将带你认识CSRF, 和其带来的严重危害, 应该如何来进行防御  
   
<!-- more -->

#### 什么是 CSRF 
CSRF (Cross-site request forgery) 跨站请求伪造。 冒用用户身份凭证, 对目标网站进行设置。

#### CSRF 的攻击过程
- 用户登录了 a.com, 浏览器保留了登录凭证
- 攻击者引诱用户访问攻击网站 b.com
- b.com 向 a.com 发出了一个请求, 例如: a.com?add-user=xxxxxx
- a.com 接收到请求, 进行验证, 有有效凭证, a.com 认为是用户发送的请求
- a.com 执行了操作 add-user
- 攻击完成, 用户并不知道

<!-- more -->

#### 攻击类型
- Get 类型
```
<img src="https://bank.com?act=xxx">
```
用户访问页面带有如上图片, 浏览器自动请求地址, 攻击完成  

- Post 类型
```
<form action="https://bank.com" method="POST">
  <input type="hidden" name="account" value="xiaoming" />
  <input type="hidden" name="amount" value="100" />
  <input type="hidden" name="act" value="transfer" />
</form>
<script> document.forms[0].submit(); </script> 
```
用户访问页面, 表单自动提交, 攻击完成    

- 链接类型
```
<a href="https://bank.com?act=xxx">点我送100话费</a>
```
链接类型比较少见, 比起前两种一打开页面就中招, 这种还需要用户进行点击, 才能完成攻击  
这种类型比较可能出现在社区网站或者论坛, 直接发起同域攻击或者跳到攻击页面  
这也是社区网站对外链都是严格控制或者不可以发布外链的原因


#### 攻击场景
有一天, 小明登录了 Gmail 邮箱, 悠闲的看着邮件。 突然收到了一封陌生邮件, 特价甩卖, 比特币 888 元一个。  
英明神武的小明肯定不相信, 但还是点开了链接进去瞧瞧。进入后是一个空页面, 于是小明关掉网页。  
过了一段时间, 小明突然发现自己某网站的密码被修改了, 里面的金币也被消费完了。  
修改密码需要通过邮箱, 小明回想起之前的奇怪邮件, 找回那个空白页面, 打开源码一看 
```
<form method="POST" action="https://mail.google.com/mail/h/ewt1jmuj4ddv/?v=prf" enctype="multipart/form-data"> 
    <input type="hidden" name="cf2_emc" value="true"/> 
    <input type="hidden" name="cf2_email" value="evilinbox@mail.com"/> 
    <input type="hidden" name="cf1_from" value=""/> 
</form> 
<script> 
    document.forms[0].submit();
</script>
```
这个页面提交了一个请求, 添加了一个过滤规则, 所有邮件都将转发至 evilinbox#mail.com
看似风平浪静的页面, 其实黑客已经得手了(所以为什么点开邮件的链接会有安全提示)。 
这是一个真实的攻击页面 [Google Gmail E-mail Hijack Technique](https://www.gnucitizen.org/blog/google-gmail-e-mail-hijack-technique/)   


#### CSRF 特点
- 攻击一般从第三方发起, 也有可能本域发起
- 攻击者只能冒用用户凭证进行操作, 无法窃取数据
- 需要诱导用户进入攻击网站

#### 防御策略
从攻击原理看, 我们并没有办法阻止CSRF攻击的发生, 但是我们可以针对CSRF的特点进行防御  

**1. Referer**

CSRF 攻击一般从第三方发起, 针对这一特点, 可以通过阻止外域调用来进行防御  
浏览器在发起请求时会在头部添加 referer 字段, 表示来源。可以通过判断 referer 来判断请求是否从外域发起, 并拒绝请求。例如这样一个接口 POST bank.com?transfer=1000, 黑客从第三方 hacku.com 发起时, referer 是指向该域名的  

这种方法只需要后端进行过滤, 前端不用处理, 实施起来比较简单。但这并不是一个可靠的办法, referer 字段是由浏览器添加上去的, 我们不能将安全都寄托在浏览器  

大部分现代浏览器都支持了 Referrer Policy ([Referrer-Policy - HTTP | MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Referrer-Policy)) 策略, 允许去除 referrer 字段  
```
<meta name="referrer" content="no-referrer"> // 全局
<a href="http://example.com" rel="noreferrer"> // 单条
```
如果攻击页面除移了 referer, 后端就得不到该值无法进行过滤。

{% note info %}
referer 会记录来源地址, 存在用户隐私问题。可能存在网站自身除移掉该字段的情况 [Referer 隐私问题](https://75.team/post/everything-you-could-ever-want-to-know-and-more-about-controlling-the-referer-header-fastmail-blog.html)
{% endnote %}


**2. Dobule cookies**  
CSRF 通过冒用凭证进行攻击, 但无法获取到数据, 针对这一特点, 对 cookie 进行2次验证。  

**实现流程**
- 用户登录后返回 cookie, 并带有csrfcookie=eW1kISE=
- 客户端发送接口时, 需要从 cookies 获取 csrfcookie, 并加入到接口参数发送 (POST //hostname.comc?srfcookie=eW1kISE=)
- 后端验证参数 csrfcookie 是否与 cookie 一致

该方法实施起来相对简单, 也无需额外存储。但只适用与接口是同域, 例如 a.com 的接口地址是 api.a.com, 就实施。这是非常常见的情况。当然如果是 domain=a.com, 将 cookies 存在顶域是可以实现的, 但通常不会这么做

{% note info %}
如果有 XSS 漏洞防御无效
{% endnote %}

**3. CSRF token**
同样针对 CSRF 无法获取数据的特点, 接口需要加入从页面获取数据做为参数  

**实现流程**
- 用户打开页面时, 由后端生成加密token并插入到页面
```
<input type="hidden" value="eW1kISE=" name="token">
```
- 客户端发送请求时, 携带该token
```
const token = document.querySelector('token').value
fetch('https://yourdomain.com', {
  method: 'POST',
  headers: new Headers({
    'Content-Type': 'application/json'
  }),
  body: JSON.stringify({
    token,
    sid: 'xxxx'
  })
})
```
- 后端获取 token, 校验是否一致

该方法可以有效防御, 相比起 referer 更加安全, 但实施起来比较麻烦, 需要后端为每个页面生成 token 并插入到页面。 token 更新也比较麻烦

{% note info %}
如果有 XSS 漏洞防御无效
{% endnote %}


**4. Header token**
这也是使用 token 进行验证, 和上一种不同在于 token 放在头部

**实现流程**
- 用户登录后后端生成加密 token 并返回, 客户端取得 token=eW1kISE= 并存在 Storage
```
// login 取得 token 存储
fetch('https://a.com?a=login')
.then(res => {
  window.localStorage.setItem('token', res.token)
})
```
- 客户端发送接口时, 从 Storage 取得 token, 并加入到自定义头部 
```
// 发送请求时加入 token
fetch('https://a.com?a=send', {
  method: 'POST',
  headers: new Headers({
    'token': window.localStorage.getItem('token')
  }),
})
```
- 后端读取 header token, 解密字段, 校验token是否失效, 并进行对比

这种方法也可以有效的防御, token 更新相对简单，可以在请求接口时统一添加 header。但可能存在有些组件使用 form 提交, 无法加入自定义 header。

和上一种方法一样, 如果你的网站存在 XSS 漏洞的话防御就失效, 所以很多金融类网站要求再次输入密码, 进行2次身份验证  


**5. SameSite Cookies**
SameSite Cookie 允许服务器要求某个 cookie 在跨站请求时不会被发送。可以阻止从第三方发起时发送 cookie
```
Set-Cookie: key=value; SameSite=Strict
```
SameSite 有3个值
- None 同域跨域都会发送
- Strict 只在同域时发送, 从外域跳转进来不发送, 例如从搜索引擎进入时
- Lax 也是只在同域时发送, 相对于 strict 较宽松, Same-site cookies 将会为一些跨站子请求保留, 如图片加载或者frames的调用，但只有当用户从外部站点导航到URL时才会发送。如link链接

{% note info %}
在新版本的浏览器中，SameSite的默认属性是SameSite=Lax。换句话说，当Cookie没有设置SameSite属性时，将会视作SameSite属性被设置为Lax——这意味着Cookies将会在当前用户使用时被自动发送。如果想要指定Cookies在同站、跨站请求都被发送，那么需要明确指定SameSite为None
{% endnote %}

如果设置了 strict, 浏览器任何跨域都不会携带 cookie, 即使跳转进入也不会携带。 那也就没有 CSRF 攻击的机会了。 对于子域的跳转, cookie 也不会保留, 用户必须重新登录, 对于用户来讲体验不好

如果设置 lax, 则外部跳转时会保留 cookie。 用户从外部或则子域进来时不需要重新登录

SameSite不支持子域名, 但项目很常见的一种情况, 站点和接口不同域名, 例如 a.com 接口地址是 api.a.com。对于这种情况就不能通过SameSite来防御, 但如果相同的话 SameSite 实施也比较方便

#### 一些历史事件
[Gmail security hijack](https://www.davidairey.com/google-gmail-security-hijack/) 
[Youtube notifications](https://cloud.tencent.com/developer/article/1425657)
[wordpress CSRF to Remote Code Execution](https://blog.ripstech.com/2019/wordpress-csrf-to-rce/)

#### 总结
CSRF 是一种危害较大的攻击, 以上防御方法可以很大程度的防御攻击。各方法的实施难度, 效果, 适用情况都不同。 没有完美的解决方案, 我们只能通过实际情况选择最合适的方法。

#### 参考文献
[Wiki CSRF](http://en.wikipedia.org/wiki/Cross-site_request_forgery)
[IBM CSRF 攻击的应对之道](https://www.ibm.com/developerworks/cn/web/1102_niugang_csrf/)
[MDN SameSite Cookie](hhttps://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies#SameSite_Cookies)
[MDN Same-origin-policy](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)