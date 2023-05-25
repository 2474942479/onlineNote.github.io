# JWT应该保存在哪里？

## Cookie

服务端可以将JWT令牌通过Cookie发给浏览器，浏览器在请求服务端接口时会自动在Cookie头中带上JWT令牌，服务端对Cookie头中的JWT令牌进行检验即可实现身份验证。但它容易受到CSRF攻击的影响。

解决的方法是通过设置Cookie的`SameSite`属性为`Strict`。跨站时不会发送 Cookie。换言之，只有当前网页的 URL 与请求目标一致，才会带上 Cookie。

Cookie除了易受CSRF攻击还有XSS攻击。黑客可以通过JS脚本读取Cookie中的信息。为了防止这一点，可以设置Cookie的属性为`HttpOnly`。

```
response.setHeader("Set-Cookie", "jwt=jwt_value;Path=/;Domain=domainvalue;Max-Age=seconds;HttpOnly");
```

> ❝
>
> 你可以通过设置`Max-Age`来设置其生存时间。

## localStorage

localStorage也可以存储JWT令牌，这种方法不易受到 CSRF 的影响。但是和Cookie不同的是它不会自动在请求中携带令牌，需要通过代码来实现。不过这样会受到XSS攻击。另外如果用户不主动清除JWT令牌，它将永远存储到localStorage。

## sessionStorage

sessionStorage大部分特性类似localStorage，不过它的生命周期不同于localStorage，它是会话级存储。**关闭页面或浏览器后会被清除**。

## 总结

您可能会注意到所有 3 种方法都有相同的缺点——“易受 XSS 攻击”。请特别注意 XSS的防护，并始终遵循XSS保护的最佳实践。

## **结论**

三种形式都容易收到XSS攻击，因此如果对安全性要求很高，要特别针对性的配置。在三种方式之中，Cookie 提供了一堆安全选项，例如`SameSite`、`HttpOnly`等。因此最好使用 Cookie。

