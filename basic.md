- [x] #### const引用是什么在栈什么在堆
- [x] #### 箭头函数prototype
- [x] #### 跨域 Cross-Origin Resource Sharing

origin:   Protocol://Domain:Port

简单请求不会触发CORS预检: `get`, `head`, `post`. 
Content-Type

* text/plain
* multipart/form-data

复杂请求会触发preflight request: 为 OPTION 请求, 包含Access-Control-Allow-Headers/ Methods/ Origin/ Max-Age.

response:200后在 Access-Control-Max-Age 时间段内无需重复 preflight request.

> Access-Control-Max-Age (secend) will takes precedence when Access-Control-Max-Age exceed it.

withCredentials: true
Access-Control-Allow-Origin: http://foo.example
Access-Control-Allow-Credentials: true

jsonp:

```js

// 封装 JSONP函数
function jsonp({ url, params, callback }) {
    return new Promise((resolve, reject) => {
        let script = document.createElement('script');
        params = JSON.parse(JSON.stringify(params));
        let arrs = [];
        for (let key in params) {
            arrs.push(`${key}=${params[key]}`);
        }
        arrs.push(`callback=${callback}`);
        script.src = `${url}?${arrs.join('&')}`;
        document.body.appendChild(script);
        window[callback] = function (data) {
            resolve(data);
            document.body.removeChild(script);
        }
    })
}
// 前端调用
jsonp({
    url: 'http://localhost:3000/say',
    params: {
        wd: 'I Love you'
    },
    callback: 'show'
}).then(data => {
    console.log(data)
})

// 后端响应
// 这里用到了 express
var express = require('express');
var router = express.Router();
var app = express();
router.get('/say',function(req,res,next) {
 //要响应回去的数据
  let data = {
    username : 'zs',
    password : 123456
  }

  let {wd , callback} = req.query;
  console.log(wd);
  console.log(callback);
  // 调用回调函数 , 并响应
  res.end(`${callback}(${JSON.stringify(data)})`);
})
app.use(router);
app.listen(3000);
```

nginx简易配置:https://juejin.cn/post/6844904094973296654

- [ ] #### 1px
- [x] #### cookie用来存什么(session id)   storage存什么

**Session:**
* **session 是基于 cookie 实现的**，session 存储在服务器端,格式自定义，sessionId 会被存储到客户端的cookie 中, cookie只能是字符串
* session 是另一种记录服务器和客户端会话状态的机制
* 需要查询数据库, 如果加快, 多半是用redis存在内存中
* **session也可以不在cookie中, 把sessionId放在url参数中**, 移动端不常用

**Token:**

* 通常由access token 和 refresh token组成, 时效长短不同.  如果要重新申请refresh token需要重新登录
* Refresh Token 及过期时间是存储在服务器的数据库中，只有在申请新的 Acesss Token 时才会验证，不会对业务接口响应时间造成影响，也不需要向 Session 一样一直保持在内存中以应对大量的请求
* 客户端收到 token 以后，用 cookie 里或者 localStorage把它存储起来
* token 完全由应用管理，所以它可以避开同源策略
* **token 可以避免 CSRF 攻击** (因为不需要 cookie 了)
* 移动端对 cookie 的支持不是很好，而 session 需要基于 cookie 实现，所以移动端常用的是 token

**JWT:**
* JWT： 将 Token 和 Payload 加密后存储于客户端，服务端只需要使用密钥解密进行校验（校验也是 JWT 自己实现的）即可，不需要查询或者减少查询数据库，因为 JWT 自包含了用户信息和加密的数据。
* Token：服务端验证客户端发送过来的 Token 时，还需要查询数据库获取用户信息，然后验证 Token 是否有效。
* 可以储存在 Cookie 里面，也可以储存在 localStorage.  发送时可以在 POST 的 body 中, 也可以在 request header 的 Authorization: 中
* 是特定格式的字符串 Header(头部).Payload(负载).Signature(签名算法)
* 由于包含了认证信息, 有效期需要短一些

[使用这几者需要考虑的问题](https://juejin.cn/post/6844904034181070861#heading-21)

**Cookie 和 Session 的区别**

* 安全性： Session 比 Cookie 安全，Session 是存储在服务器端的，Cookie 是存储在客户端的。
* 存取值的类型不同：Cookie 只支持存字符串数据，想要设置其他类型的数据，需要将其转换成字符串，Session 可以存任意数据类型。
* 有效期不同： Cookie 可设置为长时间保持，比如我们经常使用的默认登录功能，Session 一般失效时间较短，客户端关闭（默认情况下）或者 Session 超时都会失效。
* 存储大小不同： 单个 Cookie 保存的数据不能超过 4K，Session 可存储数据远高于 Cookie，但是当访问量过多，会占用过多的服务器资源。

**Token 和 Session 的区别**
* Session 是一种记录服务器和客户端会话状态的机制，使服务端有状态化，可以记录会话信息。而 Token 是令牌，访问资源接口（API）时所需要的资源凭证。Token 使服务端无状态化，不会存储会话信息。
* **Session 和 Token 并不矛盾**，作为身份认证 Token 安全性比 Session 好，因为每一个请求都有签名还能防止监听以及重放攻击，而 Session 就必须依赖链路层来保障通讯安全了。如果你需要实现有状态的会话，仍然可以增加 Session 来在服务器端保存一些状态。
* Session 认证只是简单的把 User 信息存储到 Session 里，因为 SessionID 的不可预测性，暂且认为是安全的。而 Token ，如果指的是 OAuth Token 或类似的机制的话，提供的是 认证 和 授权 ，认证是针对用户，授权是针对 App 。其目的是让某 App 有权利访问某用户的信息。这里的 Token 是唯一的。不可以转移到其它 App上，也不可以转到其它用户上。Session 只提供一种简单的认证，即只要有此 SessionID ，即认为有此 User 的全部权利。是需要严格保密的，这个数据应该只保存在站方，不应该共享给其它网站或者第三方 App。所以简单来说：如果你的用户数据可能需要和第三方共享，或者允许第三方调用 API 接口，用 Token 。

- [x] #### jsonp  屏蔽c小网站  request header里的refer? origin? host?

* host 请求头表示请求服务器的域名/IP地址和端口号。 
  域名 + 端口号.
  例子：test.com:1998
  Host 请求头决访问哪个虚拟主机。

  

* referer 为当前请求的来源页面的地址，即表示当前请求是通过此referer页面里的链接进入的。
  协议+域名+端口号+路径+参数（不包含 hash值）
  例子：test.com:1998/home?key=value
  用途: 图片防盗链

  

* origin: 跨域时会带上, 表示请求源.
协议+域名+端口号

- [x] #### 不写key值  bug?
bug似乎没有. 写key值对于react只是当前元素无需删除, 只需要就地复用, 有助于提高diff速度.
* key相同，若组件属性有所变化，则react只更新组件对应的属性；没有变化则不更新。
* key值不同，则react先销毁该组件(有状态组件的componentWillUnmount会执行)，然后重新创建该组件（有状态组件的constructor和componentWillUnmount都会执行）

- [x] #### 打包优化用的插件   dll  cache  thread
thread-loader将耗时的单独一个线程.
cache: { type: 'filesystem' }
dllPlugin 已经无了, webpack5的持久换缓存比dll更优. create-react-app和vue-cli也已经去除了.
cache-loader 也不需要了
terser-plugin官方集成

- [ ] #### 深拷贝

- [ ] #### TLS

- [x] #### XMLHttpRequest
open方法参数: ( METHOD, url, isAsync = true );
`send( requestBody )`
`abort() `取消
`xhr.withCredentials = true` 带cookie.
onreadystatechange, 五个值, XMLHttpRequest.固定值0-4分别是:   
UNSENT, OPENED (open已调用), HEADERS_RECEIVED (send已调用), LOADING (下载中), DONE (下载完成)

- [ ] #### Server Component

- [ ] #### SSR注水脱水

- [ ] #### 项目经经历吹点小作文

- [ ] #### TCP 和 UDP 的区别

- [ ] #### fiber的作用是什么

- [ ] #### react Diffing算法

render函数执行会产生react元素树，下次render会产生另外一个元素树，react需要对比两个元素树差别，来更新同步真实DOM，使用最简单的广度优先遍历，时间复杂度达到O(n^3)
react使用O(n)启发式算法，提出以下两个假设：

1. 不同的元素肯定会产生不同的树
2. 可以使用key来指定哪一些元素不用更新

**元素树的更新有以下几种情况：**

1. 无需更新
2. 全新的节点，新增一个DOM阶段
3. 需要修改属性的节点，更新样式或者其他属性
4. 不需要的节点，需要删除

- [ ] #### react的优化方法

- [ ] #### 性能优化  react和webpack
- [ ] #### plugin和loader
- [ ] #### 迁移ts遇到的问题
- [ ] #### 打开url发生了什么
- [ ] #### eventBus 观察者模式
- [ ]  以上在今天完成

