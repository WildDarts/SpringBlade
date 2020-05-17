# 1. 首页登录
- 首页登录认证且创建token的流程
- 点击首页登录，会有下面的请求
```
http://localhost:1888/api/blade-auth/token?tenantId=000000&account=admin&password=admin&type=account
```
- 对应的Controller处理是 AuthController中的`<AuthInfo> token`（至于为什么是这个类，api/blade-auth/token目前还没有弄清楚）
# 2. 过程
## 2.1. 获取用户类型，默认是web
## 2.2. 设置参数
## 2.3. 获取授权类型（比如是密码验证，验证码等）
- ITokenGranter（接口）------>>>ITokenGranter getGranter(String grantType)
- - PasswordTokenGranter
- - CaptchaTokenGranter
- - RefreshTokenGranter
        以上三个类都实现ITokenGranter，会根据授权的类型进入对应的类，如密码授权的就会进入PasswordTokenGranter
## 2.4. 获取用户相关信息并判断
- UserInfo grant(TokenParameter tokenParameter)
        该方法里面会调用到了Blade-user模块下的模块（Feign远程调用），具体代码如下
```
        result = userClient.userInfo(tenantId, account, DigestUtil.encrypt(password));
```
- IUserClient
```
@GetMapping(API_PREFIX + "/user-info")
	R<UserInfo> userInfo(@RequestParam("tenantId") String tenantId, @RequestParam("account") String account, @RequestParam("password") String password);
```
- 上面的代码会调用到blade-user模块下的UserClient中的办法
```
@GetMapping(API_PREFIX + "/user-info")
	public R<UserInfo> userInfo(String tenantId, String account, String password) {
		return R.data(service.userInfo(tenantId, account, password));
	}
```
## 2.5. 如果用户信息正确就创建认证的token，不正确则返回错误信息
```
	return R.data(TokenUtil.createAuthInfo(userInfo));
```