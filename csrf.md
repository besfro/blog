## 什么是CSRF 
CSRF (Cross-site request forgery) 跨站请求伪造。 利用浏览器的同源策略, 盗用用户身份凭证, 对目标网站进行操作。

### CSRF 攻击过程
- 用户登录了 a.com, 浏览器保留了登录凭证
- 攻击者引诱用户访问攻击网站 b.com
- b.com 向 a.com 发出了一个请求, 例如: a.com?add-user=xxxxxx
- a.com 接收到请求, 进行验证, 有有效凭证, a.com 认为是用户发送的请求
- a.com 执行了操作 add-user
- 攻击完成, 用户并不知道

### 攻击类型
- Get类型
```
<img src="https://bank.com?act=xxx">
```
用户访问页面带有如上图片, 浏览器自动请求地址, 攻击完成
- Post类型
```
<form action="https://bank.com" method="POST">
  <input type="hidden" name="account" value="xiaoming" />
  <input type="hidden" name="amount" value="100" />
  <input type="hidden" name="act" value="transfer" />
</form>
<script> document.forms[0].submit(); </script> 
```
用户访问页面, 表单自动提交。
- 链接类型
```
<a href="https://bank.com?act=xxx">点我送100话费</a> (本域攻击)
<a href="https://third.com">点我送100话费</a>
```
点击类型常见于社区类网站, 通过诱导用户点击  
这也是社区类网站或者论坛对外链都是严格控制或者不可以发布外链的原因

### 攻击场景
有一天, 小明登录了 Gmail 邮箱, 悠闲的看着邮件。 突然收到了一封陌生邮件, 特价甩卖, 比特币 888 元一个。  
英明神武的小明肯定不相信, 但还是点开了链接进去瞧瞧。进入后是一个空页面, 于是小明关掉网页。  
过了一段时间, 小明突然发现自己某网站的密码被修改了, 里面的金币也被消费完了。  
小明点开邮件链接进入网页, 看似风平浪静, 但其实黑客已经得手了(所以为什么点开邮件的链接会有安全提示)。    
什么都没有的空白页, 其实已经向攻击目标发送了一个请求。设置了过滤规则, 所有的邮件都将转发至 hacku@gmail.com


### 一些历史事件
![Gmail security hijack](https://www.davidairey.com/google-gmail-security-hijack/) 
![Youtube notifications](https://cloud.tencent.com/developer/article/1425657)
![wordpress CSRF to Remote Code Execution](https://blog.ripstech.com/2019/wordpress-csrf-to-rce/)


### CSRF 特点
- 攻击一般从第三方发起, 也有可能本域发起
- 攻击者只能冒用用户凭证进行操作, 无法窃取数据
- 需要诱导用户进入攻击网站

### 防御策略

**CSRF token**
针对CSRF无法获取数据的特点, 生成加密token, 并在调用接口时验证

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
- 后端获取 token, 校验是否过期和一致

**缺点**
- 实施相对麻烦 
- 如果有XSS, 防御无效
- token 更新麻烦
- 后端需要redis集中存储来管理token


**Header token**
这也是使用token进行验证, 和上一种不同在于token放在头部

**实现流程**
- 用户登录后后端生成用户凭证token并返回, 客户端取得 token=eW1kISE= 并存在 Storage
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
- 后端读取header token字段, 解密字段, 校验token有效性, 并与缓存数据进行比较

**优点**
- 无需额外存储
- 可以统一出口
- token 更新相对简单

**缺点**
- 如果有XSS, 防御无效



**Dobule cookies**  
双重cookies验证。 同样针对CSRF无法获取数据的特点, 对cookie进行2次验证。  

**实现流程**
- 后端返回cookie, 带有csrfcookie=eW1kISE=(随机串)
- 客户端发送接口时, 获取csrfcookie, 并加入到接口参数发送(POST https://hostname.comcsrfcookie=eW1kISE=)
- 后端验证参数csrfcookie是否与cookie一致

**优点**
- 相比CSRF token, 实施相对简单
- 无需额外的存储

**缺点**
- 不同域名无法实施, 例如 a.com 的接口地址是 api.a.com。这是非常常见的情况
- 如果有XSS, 防御无效

Csrf token

Same site

从攻击原理来看, 网站本身并没有办法防止攻击发生, 只能防御攻击。  
网站加强防御的同时, 用户也加强安全意识, 那么攻击就无机可乘

### 参考



2020.1.20