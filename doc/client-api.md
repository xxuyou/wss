[![荆秀实时数据推送服务](https://xxuyou.com/static/screenshot/logo-160.png)](https://xxuyou.com)

# WSS Real-time Data Push Service - Javascript(Socket.IO) API

[![Gitter](https://img.shields.io/badge/Powered%20by-ThinkJS%20Framework-brightgreen.svg)](http://thinkjs.org) [![Gitter](https://img.shields.io/badge/Powered%20by-Socket.IO-brightgreen.svg)](http://socket.io) [![Gitter](https://img.shields.io/badge/Powered%20by-JWT.IO-brightgreen.svg)](http://jwt.io)

## 概述

WSS 服务提供了开放于广域互联网的 URI，使得任何互联网终端均可连接和订阅数据池变动事件。事件订阅基于标准 **WebSocket** 协议而并非私有协议，使用者无须担心对未来软件系统升级的潜在的兼容性问题。

> 目前所有的主流浏览器均支持 WebSocket 协议，细节参见：
> * http://websocket.org
> * http://caniuse.com/#feat=websockets
> * https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API

## 特性

WSS 服务在客户端使用具有如下特性

* 完全基于 WebSocket 协议，开发人员可自行封装代码、也可使用第三方 Socket.IO 类库、也可使用 WSS 提供的类库
* 可以建立多个连接实例，可用于多个 app 的监听场景
* 可以在一个连接实例上监听多个 pool
* 监听 pool 的动作可以在实例化时做，也可以在实例化之后做

## 依赖

DEMO 提供的 Javascript 类库是在 Socket.IO [官网](https://socket.io) 基础上针对 WSS 服务的特性进行的封装，实现过程非常简单。

> 为叙述简洁，以下文档说明也以 Socket.IO 类库为基础撰写。类库只是提高编程效率的一种方法，开发人员不必过于拘泥，明白了基本原理完全可以自行实现 WebSocket 协议连接。

## WebSocket SSL 地址

地址可在管理后台首页“控制面板”中找到。

## 代码流程

一个完整的代码流程应当是
1. 引用第三方类库
1. 创建连接实例
1. 连接 WSS 服务
1. 主动身份认证
1. 建立多个事件监听

## Script Src 类库引用

在 Web 页面中引用类库，建议直接引用公开的 CDN 地址，免去下载到本地维护的麻烦。

```html
<!-- Socket.IO class -->
<script src="https://cdn.bootcss.com/socket.io/1.7.3/socket.io.min.js"></script>
```

###### 更详细的 API 参见：[Socket.IO 官网](https://socket.io)。

## Connect 连接

使用内置的 io 函数即可便捷的创建一个连接实例。

需要注意的是，此实例仅仅是一个可操作的指针，并未连接上 WSS 服务。

```js
var wssUrl = 'WSS WebSocket SSL Service Url';
var wssOption = {
  "transports": ['websocket', 'polling']
}; // 显式指定连接方式的顺序，websocket 协议优先
var wssInstance = io(wssUrl, wssOption); // 获取一个 websocket 连接实例
if (!wssInstance) {
  throw new Exception('Realtime Service connect fail!');
};
// 连接到 WSS 服务
wssInstance.on('connect', function () {
  // 如果连接成功，立即监听 connect_error
  wssInstance.on('connect_error', function (err) {
    throw new Exception('[ERR]connect_error' + err);
    console.log('[ERR]connect_error: ', err);
  });
  // 如果连接成功，立即监听 error
  wssInstance.on('error', function (err) {
    throw new Exception('[ERR]error' + err);
    console.log('[ERR]error: ', err);
  });
  // Auth 身份认证
});
```

## Auth 身份认证

#### 服务端：生成身份鉴权令牌

生成身份鉴权令牌的数据包需要以下三个参数，均为必填项。可增加其他业务参数，WSS 在鉴权成功后会返回全部参数。

参数 | 类型 | 长度 | 说明
----|-----|----:|------
app | string | 128 | 应用名，例如：`my_test_app`
uid | string | 64 | 业务中客户端唯一ID，通常是注册用户ID，例如：`71`
exp | integer | 8 | 标记令牌在此时间戳后过期，例如：`1497278280`


```php
<?php
$payload = array(
  'app' => 'my_test_app',
  'uid' => 71,
  'exp' => time() + 86700
);
$secretKey = "1Ufd******PitR"; // app auth key 超级密钥
$token = \JWT::encode($payload, $secretKey, {"algorithm": "HS256"}); // JWT 加密参数恒定为 HS256
?>
```

###### 更详细的 API 参见：[JWT 官网](https://jwt.io)。

#### 客户端：递交令牌申明合法身份建立连接

在连接过程中，系统无法接收额外的参数提交，因此采用的是“先连接，再表明身份”的做法，连接成功的客户端主动进行身份认证动作。连接后不做身份认证动作的，连接将会被强制关闭，也无法监听到 pool 数据变动。

身份认证方法：客户端需要主动 emit 一个名为 auth 的事件，并递交认证数据给 WSS 服务。

```js
// emit 函数
instance.emit(eventName, eventData, callback)
```

###### emit 函数参数
参数 | 类别 | 说明
----|------|-----
eventName | string | 事件名，恒定为 `auth`
eventData | json | 认证数据，JSON 格式
callback(verify) | function | 认证结果回调函数，WSS 服务会调用此函数，传入认证结果到 verify 对象

###### 认证数据参数
参数 | 类别 | 说明
----|------|-----
eventData.url | string | WSS Service Url
eventData.app | string | app 应用名
eventData.jwt | string | jwt 令牌

###### 认证结果参数
参数 | 类别 | 说明
----|------|-----
verify.err | int | 大于 0 的数值表示认证结果有错误
verify.msg | string | 认证结果描述
verify.data | json | 认证成功的，存放 jwt 令牌解签原文

```js
// 认证数据，jwt 数据应当在服务端制作完成
var authData = {
  "url": "demo_app",
  "app": "demo_app",
  "jwt": "****************"
};
// 主动认证
wssInstance.emit('auth', authData, function (verify) {
  if (!verify) {
    alert('Authentication fail. Service not response.');
    return false;
  };
  if (!verify.hasOwnProperty('err')) {
    alert('Authentication fail. Response data format incorrect!');
    return false;
  };
  if (verify['err'] > 0) {
    alert('Authentication fail. ' + verify['msg']);
    return false;
  };
  // 上面三个检测都通过了才表明身份认证成功
  // 下面可以注册对应 pool 的事件监听
});
```

> 需要注意的是：`wssInstance.emit` 方法推荐在前面成功建立连接后的回调函数中执行，参见前述代码中的注释语句。

## Regist Listeners 注册监听事件

#### 事件清单及触发顺序

RESTful API 的操作会产生不同的事件名。

##### DELETE 触发事件及顺序
1. `poolName:change`
1. `poolName:delete`

##### PUT 触发事件及顺序
1. `poolName:change`
1. `poolName:add` 或者 `poolName:update`

##### POST 触发事件及顺序
1. `poolName:change`
1. `poolName:add` 或者 `poolName:append`

#### 监听一个或者多个事件

```js
instance.on(eventName, callback)
```

###### on 函数参数
参数 | 类别 | 说明
----|------|-----
eventName | string | 事件名，格式为 `数据池名:事件名`，例如：`myTestPool:update` 表示监听数据池 myTestPool 的 PUT 事件，并得到变动数据
callback(data) | function | 如果匹配数据池有变化，WSS 服务会调用此函数，传入变化结果到 data 对象

```js
wssInstance.on('myTestPool:update', function (res) {
  console.log('[INFO] 监听到 myTestPool 资源的 PUT 事件', res);
});
// 可以建立多个监听事件
wssInstance.on('myTestPoolNew:append', function (res) {
  console.log('[INFO] 监听到 myTestPoolNew 资源的 POST 事件', res);
});
// 建立不分具体动作的事件
wssInstance.on('myTestPoolNew:change', function (res) {
  console.log('[INFO] 监听到 myTestPoolNew 资源的变动事件', res);
});
```

> 需要注意的是：`wssInstance.on` 方法推荐在前面成功建立连接后的回调函数中执行，参见前述代码中的注释语句。

## Error 错误

在连接、身份认证、注册监听事件的时候，可能会产生错误，建议使用回调函数针对错误进行处理。

## Debug 调试

WSS 服务提供的 Javascript 类库含有此开关，可方便进行调试，详见 [wss.class.js 源码](https://github.com/xxuyou/wss/blob/master/example/client/js/wss.class.js)。

## 完整的代码示意

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <!-- Socket.IO class -->
  <script src="https://cdn.bootcss.com/socket.io/1.7.3/socket.io.min.js"></script>
  <script>
  var wssUrl = 'WSS WebSocket SSL Service Url';
  var wssOption = {
    "transports": ['websocket', 'polling']
  }; // 显式指定连接方式的顺序，websocket 协议优先
  var wssInstance = io(wssUrl, wssOption); // 获取一个 websocket 连接实例
  if (!wssInstance) {
    throw new Exception('Realtime Service connect fail!');
  };
  // 连接到 WSS 服务
  wssInstance.on('connect', function () {
    // 如果连接成功，立即监听 connect_error
    wssInstance.on('connect_error', function (err) {
      throw new Exception('[ERR]connect_error' + err);
      console.log('[ERR]connect_error: ', err);
    });
    // 如果连接成功，立即监听 error
    wssInstance.on('error', function (err) {
      throw new Exception('[ERR]error' + err);
      console.log('[ERR]error: ', err);
    });
    // Auth 身份认证
    // 认证数据，jwt 数据应当在服务端制作完成
    var authData = {
      "url": "demo_app",
      "app": "demo_app",
      "jwt": "****************"
    };
    // 主动认证
    wssInstance.emit('auth', authData, function (verify) {
      if (!verify) {
        alert('Authentication fail. Service not response.');
        return false;
      };
      if (!verify.hasOwnProperty('err')) {
        alert('Authentication fail. Response data format incorrect!');
        return false;
      };
      if (verify['err'] > 0) {
        alert('Authentication fail. ' + verify['msg']);
        return false;
      };
      // 上面三个检测都通过了才表明身份认证成功
      // 下面可以注册对应 pool 的事件监听
      wssInstance.on('myTestPool:update', function (res) {
        console.log('[INFO] 监听到 myTestPool 资源的 PUT 事件', res);
      });
      // 可以建立多个监听事件
      wssInstance.on('myTestPoolNew:append', function (res) {
        console.log('[INFO] 监听到 myTestPoolNew 资源的 POST 事件', res);
      });
      // 建立不分具体动作的事件
      wssInstance.on('myTestPoolNew:change', function (res) {
        console.log('[INFO] 监听到 myTestPoolNew 资源的变动事件', res);
      });
    });
  });
  </script>
  </head>
<body></body></html>
```

_Fin._
