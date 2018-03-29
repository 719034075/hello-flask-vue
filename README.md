# hello-flask-vue
a example for learning flask and vue.

[实现修改自此](https://www.zcfy.cc/article/full-stack-single-page-application-with-vue-js-and-flask)

# 1. 客户端

## 1.1 安装vue-cli

我用`vue-cli`命令行工具搭建起Vue.js的基础框架。如果你还没有安装，可以运行：

```cmd
> npm install -g vue-cli
```

## 1.2 建立客户端

因为要做到前后端的分离，所以前后端的代码要分开在两个文件夹下，互不影响。

```cmd

# 整个项目的文件夹
> mkdir hello-flask-vue

# 进入文件夹
> cd hello-flask-vue

# 用vue-cli建立客户端部分
> vue init webpack frontend

```

## 1.3 安装客户端依赖

这里是用npm安装，如果你安装了淘宝镜像也可以用cnpm

```
> npm install
```

## 1.4 新建单页面

把`Home.vue`和`About.vue`添加到`frontend/src/components`文件夹中。

如果在新建之后出现类似这样的错误。

```
!!vue-style-loader!css-loader?{“minimize”:false,“sourceMap”:false}!../node_modules/vue-loader/lib/style-compiler/index?{“vue”:true,“id”:“data-v-05e9942a”,“scoped”:false,“hasInlineConfig”:true}!sass-loader?{“includePaths”:["./src/styles"],“data”:"@import “./src/styles/app”;",“sourceMap”:false}!../node_modules/vue-loader/lib/selector?type=styles&index=0!./App.vue in ./src/App.vue
!!vue-style-loader!css-loader?{“minimize”:false,“sourceMap”:false}!../…/node_modules/vue-loader/lib/style-compiler/index?{“vue”:true,“id”:“data-v-1399b181”,“scoped”:true,“hasInlineConfig”:true}!sass-loader?{“includePaths”:["./src/styles"],“data”:"@import “./src/styles/app”;",“sourceMap”:false}!../…/node_modules/vue-loader/lib/selector?type=styles&index=0!./Home.vue in ./src/components/Home.vue

To install them, you can run: npm install --save !!vue-style-loader!css-loader?{“minimize”:false,“sourceMap”:false}!../node_modules/vue-loader/lib/style-compiler/index?{“vue”:true,“id”:“data-v-05e9942a”,“scoped”:false,“hasInlineConfig”:true}!sass-loader?{“includePaths”:["./src/styles"],“data”:"@import “./src/styles/app”;",“sourceMap”:false}!../node_modules/vue-loader/lib/selector?type=styles&index=0!./App.vue !!vue-style-loader!css-loader?{“minimize”:false,“sourceMap”:false}!../…/node_modules/vue-loader/lib/style-compiler/index?{“vue”:true,“id”:“data-v-1399b181”,“scoped”:true,“hasInlineConfig”:true}!sass-loader?{“includePaths”:["./src/styles"],“data”:"@import “./src/styles/app”;",“sourceMap”:false}!../…/node_modules/vue-loader/lib/selector?type=styles&index=0!./Home.vue
Listening at http://localhost:8080
```

是因为新建的`vue`文件有涉及到`sass`。而`sass`在预置的`HelloWorld.vue`中是没有的。所以你需要额外安装依赖。


```
npm install sass-loader node-sass --save-dev
```

## 1.5 把静态资源包`dist`输出与`frontend`同级

在`frontend/config/index.js`找到

```
index: path.resolve(__dirname, '../dist/index.html'),
assetsRoot: path.resolve(__dirname, '../dist'),
```

改成

```
index: path.resolve(__dirname, '../../dist/index.html'),
assetsRoot: path.resolve(__dirname, '../../dist'),
```

# 2. 后端

## 2.1 新建Flask项目

在根目录`/hello-flask-vue`下，用PyCharm新建一个Flask项目`backend`。自带虚拟环境。

## 2.2 新建run

在根目录`/hello-flask-vue`下,新建一个`run.py`

```run.py
from flask import Flask, render_template

app = Flask(__name__,
            static_folder = "./dist/static",
            template_folder = "./dist")

@app.route('/')
def index():
    return render_template("index.html")
    
if __name__ == '__main__':
    app.run()
```

## 2.3 运行Flask

官方文档中，新版本的 `Flask(>=0.11)` 运行方式和以前有所不同，但是按照官方文档，可能会碰到坑的地方：

问题出在终端上面：

### 2.3.1 Linux下

```
$ export FLASK_APP=run.py
$ flask run
```

### 2.3.2 Windows下

#### 2.3.2.1 cmd下

```
> set FLASK_APP=run.py & flask run
```

#### 2.3.2.2 powershell下

```
$env:FLASK_APP=".\run.py" | flask run
```

### 2.3.3 在PyCharm下

我还是使用PyCharm来运行`run.py`

把`Run/Debug Configurations`中的`Script path`修改成`run.py`的路径即可。

## 2.4 重定向

后台服务启动之后。如果访问`/about`，Flask 会抛出一个找不到请求地址的错误。实际上是因为在 `vue-router`用了 HTML5 的 history 模式, 所以我们需要配置我们的后台服务去重定向所有的路由都跳转到 `index.html` 上。这在 Flask 上可以很简单做到。做如下修改:

```run.py
@app.route('/', defaults={'path': ''})
@app.route('/<path:path>')
def catch_all(path):
    return render_template("index.html")
```
现在地址 `localhost:5000/about` 将会重定向到 `index.html` 和 `vue-router` 将会在它自己内部处理。

## 2.5 添加404页面

因为在我们的后台服务里设置捕捉所有路由是非常困难的，所以我们用 Flask 捕捉 404 错误会重定向 所有 请求到 `index.html`（连同不存在的页面）。在 Vue.js 应用里处理未定义的路由。当然，所有的工作均可在我们的路由文件设置。

在`frontend/src/router/index.js`增加：

```
const routerOptions = [
  { path: '/', component: 'Home' },
  { path: '/about', component: 'About' },
  { path: '*', component: 'NotFound' }
]
```
现在访问不存在的页面就会跳转到`Not Found`

## 2.6 添加后端API接口

最后我们在后端创建一个API接口，然后我们通过前端调用它。

这是一个随机返回数字1到100的简单接口。

### 2.6.1 后端

在run.py上新增

```
from flask import Flask, render_template, jsonify
from random import *

app = Flask(__name__,
            static_folder = "./dist/static",
            template_folder = "./dist")

@app.route('/api/random')
def random_number():
    response = {
        'randomNumber': randint(1, 100)
    }
    return jsonify(response)

@app.route('/', defaults={'path': ''})
@app.route('/<path:path>')
def catch_all(path):
    return render_template("index.html")
    
if __name__ == '__main__':
    app.run()
```

### 2.6.2 客户端

在`Home.vue`中,先前端单独模拟随机数的生成与展示。

```
<template>
  <div>
    <p>Home page</p>
    <p>Random number from backend: {{ randomNumber }}</p>
    <button @click="getRandom">New random number</button>
  </div>
</template>

<script>
export default {
  data () {
    return {
      randomNumber: 0
    }
  },
  methods: {
    getRandomInt (min, max) {
      min = Math.ceil(min)
      max = Math.floor(max)
      return Math.floor(Math.random() * (max - min + 1)) + min
    },
    getRandom () {
      this.randomNumber = this.getRandomInt(1, 100)
    }
  },
  created () {
    this.getRandom()
  }
}
</script>
```

安装axios

```
> npm install --save-dev axios
```

在`Home.vue`中引入axios，并修改

```
import axios from 'axios'

methods: {
  getRandom () {
    // this.randomNumber = this.getRandomInt(1, 100)
    this.randomNumber = this.getRandomFromBackend()
  },
  getRandomFromBackend () {
    const path = `http://localhost:5000/api/random`
    axios.get(path)
    .then(response => {
      this.randomNumber = response.data.randomNumber
    })
    .catch(error => {
      console.log(error)
    })
  }
}
```

然后会出现跨域的情况。

安装`flask-cors`

```
> pip install -U flask-cors
```

修改`run.py`

```
from flask_cors import CORS

app = Flask(__name__,
            static_folder = "./dist/static",
            template_folder = "./dist")
cors = CORS(app, resources={"/api/*": {"origins": "*"}})
```

即可。