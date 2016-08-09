# SellerGrowth 网页授权获取用户基本信息

可以通过微信网页授权机制，来获取用户基本信息，进而实现业务逻辑，主要参考[OAuth 2.0 core spec]。
  - 获取用户昵称
  - 获取用户性别
  - 获取用户头像
  - 获取用户uid

关于网页授权回调url的说明
> 回调链接必须是完整的url，包括scheme:[//[domain][/]path，不包括[?query]和[#fragment]，一个网页应用可以包含多个回调链接，例如http://www.example.com/、 https://www.example.com/和https://www.example.com/api/等。

网页授权流程分为四步：
> 1、引导用户进入授权页面同意授权，获取code  
> 2、通过code换取网页授权access_token  
> 3、如果需要，开发者可以刷新网页授权access_token，避免过期  
> 4、通过网页授权access_token获取用户基本信息

# 第一步，用户同意授权，获取code
在确定网页应用拥有授权作用域（scope参数）的权限的前提下（默认user_info:read），引导用户打开如下页面：
```html
https://www.sellergrowth.com/api/oauth2/authorize/?response_type=code&client_id=CLIENT_ID&redirect_uri=REDIRECT_URI&scope=SCOPE&state=STATE
```
若需要请求多个scope，可以将scope用“ ”（空格）号连起来（urlencode后显示为“+”号）
若有<a href="https://github.com/SellerGrowth/unicorn-oauth2-document/blob/master/Page%20authorization/Invalid_Redirect_URI.png" target="_blank">如下</a>提示，请检查参数是否填写错误：

![redirect_uri参数错误](https://github.com/SellerGrowth/unicorn-oauth2-document/blob/master/Page%20authorization/Invalid_Redirect_URI.png?raw=true)


### 参数说明：
| 参数           | 必须         | 说明   |
| ------------  | ------------ | ------ |
| client_id     | 是       | 应用唯一标识 |
| redirect_uri  | 是       | 授权后重定向的回调链接地址，请使用urlencode对链接进行处理 |
| scope         | 否       | 应用授权作用域 |
| state         | 建议     | 重定向后会带上state参数，开发者可以填写a-zA-Z0-9的参数值 |
| response_type | 是       | 返回类型，请填写code |

### 用户同意授权后
如果用户同意授权，页面将跳转至 redirect_uri/?code=CODE&state=STATE。若用户禁止授权，或者请求非法的scope，则重定向后不会带上code参数，仅会带上error参数redirect_uri/?error=ERR_MSG

# 第二步，通过code换取网页授权access_token
尤其注意：由于网页应用的client_secret和获取到的access_token安全级别都非常高，必须只保存在服务器，不允许传给客户端。后续刷新access_token、通过access_token获取用户信息等步骤，也必须从服务器发起。

### 请求方法
```html
获取code后，POST请求以下链接获取access_token：
method：POST
url：https://www.sellergrowth.com/api/oauth2/token/?grant_type=authorization_code&code=CODE&client_secret=CLIENT_SECRET&client_id=CLIENT_ID&redirect_uri=REDIRECT_URI
header：{'Content-Type': 'application/x-www-form-urlencoded'}
```

### 参数说明：
| 参数           | 必须         | 说明   |
| ------------  | ------------ | ------ |
| client_id     | 是       | 应用唯一标识 |
| client_secret | 是       | 应用secret |
| redirect_uri  | 是       | 授权后重定向的回调链接地址，请使用urlencode对链接进行处理 |
| code          | 是       | 填写第一步获取的code参数 |
| grant_type    | 是       | 填写为authorization_code |

### 返回说明
正确时返回的JSON数据包如下：
```javascript
{
  "access_token": "ACCESS_TOKEN", 
  "token_type": "Bearer", 
  "expires_in": 7200, 
  "refresh_token": "REFRESH_TOKEN", 
  "scope": "SCOPE"
}
```
错误时返回JSON数据包如下（示例为Code无效错误）:
```javascript
{"error": "invalid_grant"}
```

# 第三步：刷新access_token（如果需要）
由于access_token拥有较短的有效期，当access_token超时后，可以使用refresh_token进行刷新，refresh_token拥有较长的有效期（6天、7天），当refresh_token失效的后，需要用户重新授权。

### 请求方法
```html
获取第二步的refresh_token后，POST请求以下链接获取access_token：
method：POST
url：https://www.sellergrowth.com/api/oauth2/token/?grant_type=refresh_token&refresh_token=REFRESH_TOKEN&client_secret=CLIENT_SECRET&client_id=CLIENT_ID
header：{'Content-Type': 'application/x-www-form-urlencoded'}
```

### 参数说明：
| 参数           | 必须         | 说明   |
| ------------  | ------------ | ------ |
| client_id     | 是       | 应用唯一标识 |
| client_secret | 是       | 应用secret |
| refresh_token | 是       | 填写通过access_token获取到的refresh_token参数 |
| grant_type    | 是       | 填写为refresh_token |

### 返回说明
正确时返回的JSON数据包如下：
```javascript
{
  "access_token": "ACCESS_TOKEN", 
  "token_type": "Bearer", 
  "expires_in": 7200, 
  "refresh_token": "REFRESH_TOKEN", 
  "scope": "SCOPE"
}
```
错误时返回JSON数据包如下（示例为Refresh_Token无效错误）:
```javascript
{"error": "invalid_grant"}
```

# 第四步：拉取用户信息(需scope为 user_info:read)

### 请求方法
```html
开发者可以通过access_token拉取用户信息：
method：GET
url：https://gaochao.sellergrowth.com/api/oauth2_server/api/oauth_user_info
header：{'Authorization': 'Bearer ACCESS_TOKEN'}
```

### 参数说明：
| 参数           | 必须         | 说明   |
| ------------  | ------------ | ------ |
| assess_token  | 是       | [Bearer Token] |

### 返回说明
正确时返回的JSON数据包如下：
其中gender性别： 0代表未知，1代表男士，2代表女士
```javascript
{
  "username":"USERNAME",
  "gender":1,
  "uid":"UID",
  "avatar":"AVATAR.jpg"
}
```
错误时返回JSON数据包如下（示例为Access_Token无效错误）:
```javascript
{"detail":"Authentication credentials were not provided."}
```

### Version
1.0.0

[OAuth 2.0 core spec]: <https://tools.ietf.org/html/rfc6749>
[Bearer Token]: <https://tools.ietf.org/html/rfc6750>
