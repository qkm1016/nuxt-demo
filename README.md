# RealWorld

```sh
# 安装依赖
npm install

# 启动开发服务
npm run dev
```

## [Vercel](https://vercel.com)

- 部署文档

  - https://nuxtjs.org/faq/now-deployment
  - https://www.nuxtjs.cn/faq/now-deployment

- 在 Vercel 的网站使用 github 账号登录

- 全局安装 vercel

  ```bash
  yarn global add vercel
  ```

- 创建配置文件 now.json or vercel.json

  ```json
  {
    "version": 2,
    "builds": [
      {
        "src": "nuxt.config.js",
        "use": "@nuxtjs/now-builder"
      }
    ]
  }
  ```

- .nowignore

  忽略构建的文件夹 .nuxt

  ```
  .nuxt
  ```

- 发布

  - 登陆

  ```bash
  vercel login
  ```

  ![image-20200826102755348](assets/image-20200826102755348.png)

  - 然后发邮件到你的邮箱中确认，确认之后

  ![image-20200826103044427](assets/image-20200826103044427.png)

  - 发布

  ```bash
  vercel
  
  vercel --prod
  ```

  

  发布完毕后，首页是静态的，可以直接查看到，但是需要请求接口的地方，比如：登录 会出错。

  点击登录按钮的时候会在控制台提示，在https的主域名下访问接口的时候，接口地址也需要使用 https

- 修改 baseUrl，把接口地址更改成 https 的

  在 plugins/request.js

  ```js
  export const request = axios.create({
    // baseURL: 'http://realworld.api.fed.lagounews.com'
    baseURL: 'https://conduit.productionready.io'
  })
  ```

  https://github.com/gothinkster/realworld-starter-kit/blob/master/FRONTEND_INSTRUCTIONS.md

  https://conduit.productionready.io/api/tags

## Serverless

- Serverless 是一种架构模式，无服务器架构
  - 对于使用 Serverless 架构进行开发的项目，开发者最明显的感受就是更关注应用的业务本身，不必再去过多关心服务器和运行平台的一系列问题
- 无服务器，并不是真的没有服务器，只是开发人员眼中不需要关注服务器。开发人员只需要按照一定的要求完成开发工作，剩下的所有事情全部交给 Serverless 容器完成。
- 我们的应用主要由两大块组成，分别是逻辑与存储。Serverless 中就通过两种方式解决了这两块的需求，分别是：
  - 函数即服务，Function as a Service，FaaS；
  - 后端即服务，Backend as a Service，BaaS。

- Serverless 的优势
  - 不需要再考虑什么物理机/虚拟机，结合工作流的情况下，代码提交自动部署，直接运行；
  - 没有服务器，维护成本自然大大降低，安全性稳定性更高；
  - 都是弹性伸缩云，硬件资源需要多少分配多少，不用担心性能问题；
  - 大多数 Serverless 服务商的计价方式都是按使用情况（如流量、CPU 占用）来收费；

- Vercel Serverless 文档
  - https://vercel.com/docs
  - https://vercel.com/docs/runtimes#official-runtimes/node-js/node-js-dependencies
- demo1
  - 本地测试 vercel dev

```js
module.exports = (req, res) => {
  const { name = 'World' } = req.query
  res.status(200).send(`Hello ${name}!`)
}
```

- demo2

```js
import axios from 'axios'
module.exports = async (req, res) => {
  // const { name = 'World' } = req.query
  const { data } = await axios.get('https://conduit.productionready.io/api/tags')
  let html = '<ul>'
  data.tags.forEach(item => {
    html += `<li>${item}</li>`
  })
  html += '</ul>'
  res.status(200).send(html)
}
```

- demo3

```js
module.exports = (req, res) => {
  const data = require('../data.json')
  res.json(data)
}
```

- demo4

  now.json/vercel.json

```json
{
  "version": 2,
  "routes": [
    { "src": "/api/server/(.*)", "dest": "/api/server.js" }
  ]
}
```

```js
const path = require('path')
const jsonServer = require('json-server')
const server = jsonServer.create()
const router = jsonServer.router(path.join(__dirname, '../data.json'))
const middlewares = jsonServer.defaults()

server.use(middlewares)
server.use(router)

module.exports = server
```



```js
const path = require('path')
const os = require('os')
const fs = require('fs')
const jsonServer = require('json-server')
const server = jsonServer.create()

const middlewares = jsonServer.defaults()

const dbFilename = path.join(os.tmpdir(), 'db.json')

// 判断一下 dbFilename 是否存在，如果不存在才创建
if (!fs.existsSync(dbFilename)) {
  fs.writeFileSync(dbFilename, JSON.stringify({
    "posts": [
      { "id": 1, "title": "json-server", "author": "typicode", "apiId": "server" },
      { "id": 2, "title": "iis", "author": "ms", "apiId": "server" }
    ],
    "comments": [
      { "id": 1, "body": "some comment", "postId": 1, "apiId": "server" }
    ],
    "profile": { "name": "typicode", "apiId": "server" }
  }))
}

const router = jsonServer.router(dbFilename)
server.use(middlewares)
server.use(router)

module.exports = server
```

