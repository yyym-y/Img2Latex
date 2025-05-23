# Img2Latex 项目文档

## 1. 项目概述

Img2Latex是一个将数学公式图片转换为LaTeX代码的在线服务。用户可以通过上传图片或提供图片URL，获取对应的LaTeX代码。支持AI辅助优化和直接LaTeX识别两种模式。

## 2. 功能需求

### 2.1 核心功能

- 图片上传转换：支持用户上传数学公式图片，转换为LaTeX代码
- URL转换：支持通过图片URL进行转换
- AI辅助优化：通过AI对话方式优化LaTeX代码
- 历史记录：保存用户的转换历史，保留AI对话记录
- 代码复制：一键复制生成的LaTeX代码

### 2.2 用户功能

- 用户注册/登录（支持用户名或邮箱登录）
- 第三方登录（微信、Google、GitHub）
- VIP会员系统
- 个人中心
- 转换历史查看

### 2.3 VIP特权

- 更多的AI对话次数
- 更多的LaTeX识别次数

## 3. API文档

### 3.1 用户相关API

#### 3.1.1 发送邮箱验证码

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

#### 3.1.2 验证邮箱验证码

> 要求后端校验验证码是否正确以及是否过期

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

#### 3.1.3 用户注册

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
        "remainingLatexCount": 20
    }
}
```

#### 3.1.4 用户登录

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
        "avatar": "string",
        "role": "string",
        "isVip": false,
        "vipStartTime": "string",
        "vipEndTime": "string",
        "remainingAiCount": 10,
        "remainingLatexCount": 10,
        "permissions": ["string"],
        "lastLoginTime": "string",  // 上次登录时间
        "currentLoginTime": "string"  // 当前登录时间
    }
}
```

#### 3.1.5 找回密码

- 请求路径：`/api/user/forgot-password`
- 请求方法：POST
- 请求参数：

```json
{
    "email": "string"
}
```

- 响应格式：

```json
{
    "code": 1,
    "msg": "验证邮件已发送",
    "data": {
        "expireTime": "string"  // 验证码过期时间
    }
}
```

#### 3.1.6 重置密码

- 请求路径：`/api/user/reset-password`
- 请求方法：POST
- 请求参数：

```json
{
    "email": "string",
    "verifyCode": "string",
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

#### 3.1.7 第三方登录

- 请求路径：`/api/user/oauth/{platform}`
- 请求方法：POST
- 请求参数：

```json
{
    "code": "string",  // 第三方平台授权码
    "state": "string"  // 状态码，用于防止CSRF攻击
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
        "avatar": "string",
        "role": "string",
        "permissions": ["string"],
        "platform": "string",  // 登录平台：WECHAT, GOOGLE, GITHUB
        "platformUserId": "string",  // 第三方平台用户ID
        "isNewUser": false,  // 是否新用户
        "needBindEmail": false  // 是否需要绑定邮箱
    }
}
```

#### 第三方登录流程说明

1. 前端流程：
   - 用户点击第三方登录按钮（微信/Google/GitHub）
   - 前端生成随机state参数，并保存到localStorage
   - 跳转到对应第三方平台的授权页面
   - 用户授权后，第三方平台回调前端页面，并返回code和state
   - 前端验证state是否匹配，防止CSRF攻击
   - 调用后端登录API，传递code和state

2. 后端流程：
   - 验证state参数
   - 使用code向第三方平台请求access_token
   - 使用access_token获取用户信息
   - 检查用户是否已存在：
     - 已存在：直接登录
     - 不存在：创建新用户
   - 检查是否需要绑定邮箱：
     - 第三方平台返回了邮箱：自动绑定
     - 未返回邮箱：标记需要绑定
   - 生成系统token并返回用户信息

3. 安全考虑：
   - 使用state参数防止CSRF攻击
   - 验证第三方平台返回的用户信息
   - 限制第三方登录的请求频率
   - 记录第三方登录的日志
   - 敏感操作需要二次验证

4. 错误处理：
   - 授权失败：返回具体错误信息
   - 网络超时：提示用户重试
   - 用户取消授权：返回友好提示
   - 账号绑定冲突：提示用户解绑或使用其他方式登录

5. 特殊情况处理：
   - 用户已存在但未绑定第三方账号：自动绑定
   - 第三方账号已绑定其他用户：提示用户解绑
   - 邮箱已被其他账号使用：提示用户更换邮箱
   - 用户信息不完整：引导用户补充信息

#### 3.1.8 绑定第三方账号

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
        "platform": "string",
        "platformUserId": "string"
    }
}
```

#### 3.1.9 解绑第三方账号

- 请求路径：`/api/user/unbind-oauth`
- 请求方法：POST
- 请求参数：

```json
{
    "userId": "string",  // 用户ID
    "platform": "string",  // WECHAT, GOOGLE, GITHUB
    "platformUserId": "string",  // 第三方平台用户ID
    "verifyType": "string",  // PASSWORD, EMAIL, PHONE
    "verifyCode": "string",  // 验证码（如果使用邮箱或手机验证）
    "password": "string"  // 密码（如果使用密码验证）
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

- 错误码说明：
  - 1001: 用户不存在
  - 1002: 第三方账号未绑定
  - 1003: 验证失败
  - 1004: 不能解绑唯一的登录方式
  - 1005: 验证码已过期
  - 1006: 验证码错误
  - 1007: 密码错误

- 安全要求：
  1. 必须验证用户身份
  2. 不能解绑唯一的登录方式
  3. 解绑操作需要二次验证
  4. 记录解绑操作日志
  5. 解绑后清除相关token

#### 3.1.10 开通/续费VIP

- 请求路径：`/api/user/vip/subscribe`
- 请求方法：POST
- 请求参数：

```json
{
    "plan": "string",  // MONTHLY, QUARTERLY, YEARLY
    "paymentMethod": "string"  // WECHAT, ALIPAY, CREDIT_CARD
}
```

- 响应格式：

```json
{
    "code": 1,
    "msg": "开通成功",
    "data": {
        "orderId": "string",
        "paymentUrl": "string",
        "expireTime": "string"
    }
}
```

### 3.2 转换相关API

#### 3.2.1 图片上传转换

- 请求路径：`/api/convert/upload`
- 请求方法：POST
- 请求参数：multipart/form-data
  - file: 图片文件
  - type: string  // AI 或 LATEX
- 响应格式：

```json
{
    "code": 1,
    "msg": "转换成功",
    "data": {
        "latexCode": "string",
        "imageUrl": "string",
        "convertId": "string",
        "remainingAiCount": 10,
        "remainingLatexCount": 10
    }
}
```
#### 3.2.2 AI对话优化

- 请求路径：`/api/convert/ai-chat`
- 请求方法：POST
- 请求参数：

```json
{
    "convertId": "string",
    "message": "string"
}
```

- 响应格式：

```json
{
    "code": 1,
    "msg": "对话成功",
    "data": {
        "conversationId": "string",
        "messageCount": 1,
        "maxMessages": 10,
        "response": "string",
        "updatedLatexCode": "string"
    }
}
```

#### 3.2.3 获取对话历史

- 请求路径：`/api/convert/ai-chat/history`
- 请求方法：GET
- 请求参数：
  - conversationId: string
- 响应格式：

```json
{
    "code": 1,
    "msg": "获取成功",
    "data": {
        "conversationId": "string",
        "messageCount": 1,
        "maxMessages": 10,
        "messages": [
            {
                "sequenceNumber": 1,
                "role": "USER",
                "content": "string",
                "createTime": "string"
            }
        ]
    }
}
```

#### 3.2.4 URL转换

- 请求路径：`/api/convert/url`
- 请求方法：POST
- 请求参数：

```json
{
    "imageUrl": "string"
}
```

- 响应格式：

```json
{
    "code": 1,
    "msg": "转换成功",
    "data": {
        "latexCode": "string",
        "imageUrl": "string",
        "convertId": "string"
    }
}
```

#### 3.2.5 批量转换

- 请求路径：`/api/convert/batch`
- 请求方法：POST
- 请求参数：multipart/form-data
  - files: 多个图片文件
- 响应格式：

```json
{
    "code": 1,
    "msg": "批量转换成功",
    "data": [
        {
            "latexCode": "string",
            "imageUrl": "string",
            "convertId": "string"
        }
    ]
}
```

### 3.3 历史记录相关API

#### 3.3.1 获取转换历史

- 请求路径：`/api/history/list`
- 请求方法：GET
- 请求参数：
  - page: int
  - size: int
  - type: string  // ALL, AI, LATEX
- 响应格式：

```json
{
    "code": 1,
    "msg": "获取成功",
    "data": {
        "total": "int",
        "list": [
            {
                "convertId": "string",
                "imageUrl": "string",
                "latexCode": "string",
                "conversionType": "string",
                "hasAiChat": false,
                "createTime": "string"
            }
        ]
    }
}
```

## 4. 技术栈建议

### 4.1 前端

- React/Vue.js
- Ant Design/Element UI
- Axios
- MathJax/KaTeX (用于LaTeX预览)
- WebSocket (用于AI对话实时通信)

### 4.2 后端

- Python FastAPI/Flask
- OCR引擎：Mathpix API
- AI服务：OpenAI API
- 数据库：MySQL
- 文件存储：阿里云OSS/腾讯云COS
- 第三方登录集成：
  - 微信开放平台
  - Google OAuth2.0
  - GitHub OAuth
- 支付集成：
  - 微信支付
  - 支付宝
- 缓存：Redis
- 消息队列：RabbitMQ

### 4.3 部署

- Docker容器化
- Nginx反向代理
- HTTPS证书

## 5. 安全考虑

- 用户密码加密存储
- API访问频率限制
- 文件上传大小限制
- 图片格式验证
- XSS防护
- CSRF防护
- OAuth2.0安全配置
- 第三方登录状态验证
- 支付安全
- 敏感操作二次验证
- 邮箱验证码有效期控制（5分钟）
- 邮箱验证码使用次数限制（1次）
- 邮箱验证码发送频率限制（同一邮箱1分钟内最多发送1次）

## 6. 性能要求

- 单张图片转换响应时间 < 3秒
- 批量转换（10张以内）响应时间 < 30秒
- AI对话响应时间 < 2秒
- 系统并发支持 > 100
- 图片大小限制 < 10MB
- 支持的图片格式：JPG, PNG, GIF
- AI对话限制：每个对话最多10次交互

