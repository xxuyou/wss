[![荆秀实时数据推送服务](https://xxuyou.com/static/screenshot/logo-160.png)](https://xxuyou.com)

# WSS Real-time Data Push Service - RESTful API

[![Gitter](https://img.shields.io/badge/Powered%20by-ThinkJS%20Framework-brightgreen.svg)](http://thinkjs.org) [![Gitter](https://img.shields.io/badge/Powered%20by-Socket.IO-brightgreen.svg)](http://socket.io) [![Gitter](https://img.shields.io/badge/Powered%20by-JWT.IO-brightgreen.svg)](http://jwt.io)

## 概述

WSS 服务提供了基于 HTTP 1.1 的 RESTful API，发布者可使用 GET/DELETE/PUT/POST 指令来操作 WSS 服务提供的资源；采用 RESTful API 的好处是可以抹平服务端使用的开发语言的差异，学习成本基本为 0，技术开发人员可以将更多的精力放在业务数据结构、流程、逻辑等的构建上。

## 限制

如果没有特别说明，使用本接口有如下限制：

* 接口 URI 不使用鉴权令牌（`JWT Token`），直接使用 `AuthKey`
* 接口请求正文和响应正文字符编码均为 `UTF-8` 格式
* 接口请求头参数可显式声明为 `Content-Type: application/json`
* 接口请求正文和响应正文均为 `JSON` 格式（参见：[json.org](http://www.json.org)）
* 接口请求频率限制为 **10次/秒**，大于此频率的请求可能会被系统判定为恶意爬虫
* 接口 **每次** 上行数据包（包含请求头和请求正文）大小限制为 **16M**，超过此大小可能会造成不可预料的结果
* 建议使用 `IPv4 only` 参数调用接口（如：`curl -4 ...`），可提高 DNS 效率

## URI Schema 资源地址

**URI** `WSS RESTful API Url/<AuthKey>/<AppName>/<PoolName>`

参数      | 类别  | 长度  | 说明
:------- |:-----| -----:| :---------
AuthKey  | string | 40 | 应用密钥，明文，例如：`1Ufd**PitR`
AppName  | string | 128 | 应用名，例如：`my_test_app`
PoolName | string | 128 |  数据池名，例如：`client_price`

## GET 获取数据

#### Request Header 请求头

N/A

#### Request Body 请求正文

N/A

#### Response Header 响应头

参数      | 类别  | 说明
:------- |:-----| :---------
url | string | 请求的完整 URI Schema
content_type | string | 恒定为 application/json; charset=utf-8
http_code | int | 返回 200 表示正确响应

> 响应头中的 http_code 参数可用来识别网络请求是否正确被接口响应，建议仅在返回 200 才进行响应正文参数分析动作。

#### Response Body 响应正文

参数      | 类别  | 说明
:------- |:-----| :---------
err | int | 返回 0 表示无错误
msg | string | 如有错误，则是错误描述文字
data | object | 通常是以数组格式来组装数据

> 响应正文是个 JSON 格式的 String，使用者需要将其反序列化为语言可识别的对象，然后才可以进行操作。

```json
{"err":0,"msg":"","data":[{"event":"auction_close","action_id":"341","result":"1","complete_price":"1450000","customer_id":"87","order_id":"629"}]}
```


## DELETE 删除数据

#### Request Header 请求头

N/A

#### Request Body 请求正文

N/A

#### Response Header 响应头

参数      | 类别  | 说明
:------- |:-----| :---------
url | string | 请求的完整 URI Schema
content_type | string | 恒定为 application/json; charset=utf-8
http_code | int | 返回 200 表示正确响应

> 响应头中的 http_code 参数可用来识别网络请求是否正确被接口响应，建议仅在返回 200 才进行响应正文参数分析动作。

#### Response Body 响应正文

参数      | 类别  | 说明
:------- |:-----| :---------
err | int | 返回 0 表示无错误
msg | string | 如有错误，则是错误描述文字
data | object | 包含以下参数
data.affectedRows | int | 表示删除的数据池内资源的条数
data.key | string | 返回被删除的数据池名

> 响应正文是个 JSON 格式的 String，使用者需要将其反序列化为语言可识别的对象，然后才可以进行操作。

```json
{"err":0,"msg":"","data":{"affectedRows":2,"key":"myTestPoolNew1"}}
```


## PUT 更新数据／覆盖数据／重写数据

PUT 操作可以理解为更新数据、替换数据、覆盖数据等，如下示例：

```json
// 原资源
{"name":"小明"}
// PUT {"sex":"男"} 后
{"sex":"男"}
```

#### Request Header 请求头

参数 | 值 | 说明
:------- |:--- |:---------
Content-Type | application/json | 声明本次提交的资源格式

#### Request Body 请求正文

数据类别 | 长度 | 说明
:------- |----:| :---------
json | &lt; 16M | 格式良好的 JSON 字符串，长度小于 16M

> 提交的是 JSON String 必须格式正确，接口如果解析失败将返回不可预料的结果。

#### Response Header 响应头

参数      | 类别  | 说明
:------- |:-----| :---------
url | string | 请求的完整 URI Schema
content_type | string | 恒定为 application/json; charset=utf-8
http_code | int | 返回 200 表示正确响应

> 响应头中的 http_code 参数可用来识别网络请求是否正确被接口响应，建议仅在返回 200 才进行响应正文参数分析动作。

#### Response Body 响应正文

参数      | 类别  | 说明
:------- |:-----| :---------
err | int | 返回 0 表示无错误
msg | string | 如有错误，则是错误描述文字
data | object | 包含以下参数
data.key | string | 返回新增的数据池数据体的 uuid

> 响应正文是个 JSON 格式的 String，使用者需要将其反序列化为语言可识别的对象，然后才可以进行操作。

```json
{"err":0,"msg":"","data":{"key":"r1qxp3sGZ"}}
```


## POST 新增数据／追加数据

POST 操作可以理解为追加数据，如下示例：

```json
// 原资源
{"name":"小明"}
// POST {"sex":"男"} 后
[{"name":"小明"},{"sex":"男"}]
// POST {"name":"小明"} 后
[{"name":"小明"},{"sex":"男"},{"name":"小明"}]
```

POST 操作要点
* 新数据追加在现有数据的尾部
* 追加的新数据不会覆盖已有的相同的数据
* 如果 pool 不存在，本操作等同于 PUT 操作

#### Request Header 请求头

参数 | 值 | 说明
:------- |:--- |:---------
Content-Type | application/json | 声明本次提交的资源格式

#### Request Body 请求正文

数据类别 | 长度 | 说明
:------- |----:| :---------
json | &lt; 16M | 格式良好的 JSON 字符串，长度小于 16M

> 提交的是 JSON String 必须格式正确，接口如果解析失败将返回不可预料的结果。

#### Response Header 响应头

参数      | 类别  | 说明
:------- |:-----| :---------
url | string | 请求的完整 URI Schema
content_type | string | 恒定为 application/json; charset=utf-8
http_code | int | 返回 200 表示正确响应

> 响应头中的 http_code 参数可用来识别网络请求是否正确被接口响应，建议仅在返回 200 才进行响应正文参数分析动作。

#### Response Body 响应正文

参数      | 类别  | 说明
:------- |:-----| :---------
err | int | 返回 0 表示无错误
msg | string | 如有错误，则是错误描述文字
data | object | 包含以下参数
data.key | string | 返回追加的数据池数据体的 uuid

> 响应正文是个 JSON 格式的 String，使用者需要将其反序列化为语言可识别的对象，然后才可以进行操作。

```json
{"err":0,"msg":"","data":{"key":"r1qxp3sGZ"}}
```

_Fin._
