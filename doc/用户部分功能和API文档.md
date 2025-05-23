# 用户部分功能及API文档

API 格式说明，API设计遵循RESHful 原则，所有服务端返回的格式为：

```json
{
    code : int, // 0 表示错误，1表示正确
    msg : "这里是客户端返回的信息",
   	data : any //客户端返回的信息
}
```

## 1. 用户登录/注册/找回密码/第三方登录/解绑

### 1.1 邮箱验证 API

#### 1.1.1 发送邮箱验证码

> 邮箱验证码存储在缓存中，不直接存储在文件中，对于超时的邮箱验证码，可以定期删除

- 请求路径：`/api/user/email/verify-code`
- 请求方法：POST
- 请求参数：

```json
{
    "email": "string",
    "type": "string"  // REGISTER, RESET_PASSWORD
}
```

- 响应格式：

```json
{
    "code": 1,
    "msg": "验证码已发送",
    "data": {
        "expireTime": "string"  // 验证码过期时间
    }
}
```

#### 1.1.2 验证邮箱验证码

> 要求后端校验验证码是否正确以及是否过期， 同时生成临时token表示邮箱校验成功
>
> 临时token存储在缓存中,当token被校验后可以删除

- 请求路径：`/api/user/email/verify`
- 请求方法：POST
- 请求参数：

```json
{
    "email": "string",
    "code": "string",
    "type": "string"  // REGISTER, RESET_PASSWORD
}
```

- 响应格式：

```json
{
    "code": 1,
    "msg": "验证成功",
    "data": {
        "verified": true,
        "token": "string"  // 用于注册或重置密码的临时令牌
    }
}
```

### 1.2 用户直接注册

> 前端在用户未验证成功验证码的时候，不准让用户点击注册按钮，须成功验证邮箱后，将临时token携带进行校验
>
> 后端需要校验 email，phone时候有相同的，并且检验邮箱token是否正确
>
> 用户刚刚注册完成的时候：头像 avator 直接置为NULL， is_third_party 为 0， updated_at = created_at

- 请求路径：`/api/user/register`
- 请求方法：POST
- 请求参数：

```json
{
    "username": "string",
    "password": "string",
    "email": "string",
    "phone": "string",
    "verifyToken": "string"  // 邮箱验证后的临时令牌
}
```

- 响应格式：

```json
{
    "code": 1,
    "msg": "注册成功",
    "data": {
        "userId": "string",
        "username": "string",
        "email": "string",
        "phone": "string",
        "role": "string",
        "isVip": false,
        "remainingAiCount": 15,
        "remainingLatexCount": 20,
        "isNewUser": true,  // 是否新用户
        "lastLoginTime": "string",  // 上次登录时间
        "currentLoginTime": "string"  // 当前登录时间
    }
}
```

### 1.3 用户直接登录

- 请求路径：`/api/user/login`
- 请求方法：POST
- 请求参数：

```json
{
    "account": "string",  // 用户名或邮箱
    "password": "string"
}
```

- 响应格式：

```json
{
    "code": 1,
    "msg": "登录成功",
    "data": {
        "token": "string",
        "userId": "string",
        "username": "string",
        "email": "string",
        "phone": "string",
        "avatar": "string", // 头像
        "role": "string", // 权限 USER ADMIN
        "isVip": bool,
        "vipStartTime": "string",
        "vipEndTime": "string",
        "remainingAiCount": int,
        "remainingLatexCount": int,
        "isNewUser": false,  // 是否新用户
        "lastLoginTime": "string",  // 上次登录时间
        "currentLoginTime": "string"  // 当前登录时间
    }
}
```

### 1.4 用户找回密码

#### 1.4.1 找回密码邮箱认证

> 见 1.1 邮箱验证 API

#### 1.4.2 重置密码

- 请求路径：`/api/user/reset-password`

- 请求方法：POST
- 请求参数：

```json
{
    "email": "string",
    "verifyToken": "string",  // 邮箱验证后的临时令牌
    "newPassword": "string"
}
```

- 响应格式：

```json
{
    "code": 1,
    "msg": "密码重置成功",
    "data": null
}
```

### 1.5 第三方账号登录

> 这里你自己去学习以下,我也不是很清楚第三方登陆的流程, 问了一下AI, 你参考一下
>
> 1. 前端流程：
>
> - 用户点击第三方登录按钮（微信/Google/GitHub）
> - 前端生成随机state参数，并保存到localStorage
> - 跳转到对应第三方平台的授权页面
> - 用户授权后，第三方平台回调前端页面，并返回code和state
> - 前端验证state是否匹配，防止CSRF攻击
> - 调用后端登录API，传递code和state
>
> 2. 后端流程：
>
> - 验证state参数
> - 使用code向第三方平台请求access_token
> - 使用access_token获取用户信息
> - 检查用户是否已存在：
>   - 已存在：直接登录
>   - 不存在：创建新用户
> - 检查是否需要绑定邮箱：
>   - 第三方平台返回了邮箱：自动绑定
>   - 未返回邮箱：标记需要绑定
> - 生成系统token并返回用户信息

- 请求路径：`/api/user/bind-oauth`
- 请求方法：POST
- 请求参数：

```json
{
    "platform": "string",  // WECHAT, GOOGLE, GITHUB
    "code": "string",
    "state": "string"
}
```

- 响应格式：

```json
{
    "code": 1,
    "msg": "绑定成功",
    "data": {
        "token": "string",
        "userId": "string",
        "username": "string",
        "email": "string",
        "phone": "string",
        "avatar": "string",
        "role": "string",
        "isVip": false,
        "vipStartTime": "string",
        "vipEndTime": "string",
        "remainingAiCount": 15,
        "remainingLatexCount": 20,
        "platform": "string",  // 登录平台：WECHAT, GOOGLE, GITHUB
        "platformUserId": "string",  // 第三方平台用户ID
        "isThirdParty": true,  // 是否第三方登录
        "isNewUser": bool,  // 是否新用户
        "lastLoginTime": "string",  // 上次登录时间
        "currentLoginTime": "string"  // 当前登录时间
    }
}
```

#### 1.6 解绑第三方账号

- 请求路径：`/api/user/unbind-oauth`
- 请求方法：POST
- 请求头携带 token `'Authorization' : token`
- 请求参数：

```json
{
    "userId": "string",  // 用户ID
    "platform": "string",  // WECHAT, GOOGLE, GITHUB (解绑的第三方)
    "platformUserId": "string",  // 第三方平台用户ID
}
```

- 响应格式：

```json
{
    "code": 1,
    "msg": "解绑成功",
    "data": {
        "userId": "string",
        "username": "string",
        "email": "string",
        "phone": "string",
        "unbindTime": "string",
        "remainingBindings": [
            {
                "platform": "string",
                "platformUserId": "string",
                "bindTime": "string"
            }
        ],
        "loginMethods": ["PASSWORD", "WECHAT", "GOOGLE", "GITHUB"]  // 剩余可用的登录方式
    }
}
```
