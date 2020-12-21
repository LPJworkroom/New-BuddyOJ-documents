

# Java类

## JWTFilter

JWTFilter继承AuthenticationFilter，需要登录的接口都会经过JWTFilter，该filter负责检查http请求头authorization段中的token并取出，调用SecurityManager.getSubject().login(token)，也就是调用ShiroRealm的自定义JWT验证方法后，才允许访问，否则返回401。
refresh token接口不需要经过JWTFilter。

## ShiroRealm

doGetAuthorizationInfo方法根据用户principle参数从数据库查出用户网站权限和团队权限。
doGetAuthenticationInfo方法根据JWTFilter调用传递的token，验证token合法性及用户id是否存在，从而验证登录。

## JWTUtil

该类提供验证token和对token签名的方法。

## JWTToken

虽然类名叫做JWTToken，但该类并不作真正的JWTToken用，仅仅是替代ShiroRealm的认证方法使用的参数AuthenticationToken。该类会实现AuthenticationToken类，保存真正的token。真正的JWTToken具体形式是一个String，通过JWT相关库对此String操作。

# 事务流程

## 登录

向login接口传username和password，服务器验证正确后，生成access token 和 refresh token 返回给前端。

## 需登录的请求

请求接口时，前端在http请求的Authurization段带上access token。
请求会经过以下处理：
1.经过JWTFilter，该filter检查http请求头authorization段中的token并取出，调用SecurityManager.getSubject().login(token)，也就是调用ShiroRealm的自定义JWT验证方法。
2.ShiroRealm调用doGetAuthentication方法验证token。首先从token取出userid，从数据库取出该用户的jwt_secret，验证token未被篡改。随后检查过期时间，若过期则返回“need_refresh_token”，告诉前端调用refresh接口获取新access token。若能通过以上验证，则说明登录成功，返回SimpleAuthenticationInfo给JWTFilter；否则抛出AuthenticationException。
3.JWTFilter接受来自ShiroRealm的验证登录结果，成功则请求通过。若catch到exception则返回相应的失败原因。

## refresh请求

前端将refresh token传来，请求新的access token。

- todo

## 用户权限发生变化
当用户团队网站权限、团队权限发生变化，为了防止用户继续使用旧权限的token，服务器会保存用户权限最后发生变化的时间jwt_updated_at。所有请求都会检查token的updated_at，若与数据库不同则要求前端进行refresh请求。



# Token 格式

## Access token:

```
{
	userId,
	tokenType:'access',
	role,
	teamRole,
	expiresAt,
	updatedAt,
}
```

## Refresh token:

```
{
	userId,
	tokenType:'refresh',
	expiresAt,
}
```

