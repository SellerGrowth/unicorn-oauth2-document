# SellerGrowth 网页授权获取用户基本信息

access_token是网站应用的全局唯一票据，在调用各接口时都需使用access_token。开发者需要进行妥善保存。access_token的有效期目前为2个小时，需定时刷新，重复获取将导致上次获取的access_token失效。

access_token的使用及生成方式说明：
> 1、为了保密client_secrect，第三方需要一个access_token获取和刷新的中控服务器。而其他业务逻辑服务器所使用的access_token均来自于该中控服务器，不应该各自去刷新，否则会造成access_token覆盖而影响业务；
> 2、目前access_token的有效期通过返回的expires_in来传达，目前是7200秒之内的值。中控服务器需要根据这个有效时间去刷新新access_token。在刷新过程中，中控服务器对外输出的依然是老access_token，在刷新短时间内，新老access_token都可用，这保证了第三方业务的平滑过渡；
> 3、access_token的有效时间可能会在未来有调整，所以中控服务器不仅需要内部定时主动刷新，还需要提供被动刷新access_token的接口，这样便于业务服务器在API调用获知access_token已超时的情况下，可以触发access_token的刷新流程。

如果第三方不使用中控服务器，而是选择各个业务逻辑点各自去刷新access_token，那么就可能会产生冲突，导致服务不稳定。

网站应用可以使用client_id和client_secrect调用本接口来获取access_token。注意调用所有接口时均需使用https协议。

### 请求方法
```html
method：POST
url：https://www.sellergrowth.com/api/oauth2/token/?grant_type=client_credentials&client_secret=CLIENT_SECRET&client_id=CLIENT_ID
header：{'Content-Type': 'application/x-www-form-urlencoded'}
```

### 参数说明：
| 参数           | 必须         | 说明   |
| ------------  | ------------ | ------ |
| client_id     | 是       | 应用唯一标识 |
| client_secret | 是       | 应用secret |
| grant_type    | 是       | 填写为client_credentials |

### 返回说明
正确时返回的JSON数据包如下：
```javascript
{
  "access_token": "ACCESS_TOKEN", 
  "token_type": "Bearer", 
  "expires_in": 7200, 
  "scope": "SCOPE"
}
错误时返回JSON数据包如下（示例为Code无效错误）:
```javascript
{"error": "invalid_client"}
```

