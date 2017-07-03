手摸手，带你用vue撸后台 系列一（基础篇）

完整项目地址：vue-element-admin
系类文章二： 手摸手，带你用vue撸后台 系列二(登录权限篇)
系类文章三： 手摸手，带你用 vue 撸后台 系列三 (实战篇))

前言

说好的教程终于来了，第一篇文章主要来说一说在开始写业务代码前的一些准备工作吧，但这里不会教你webpack的基础配置，热更新怎么做，webpack速度优化等等，有需求的请自行google。

目录结构

├── build                      // 构建相关    
├── config                     // 配置相关  
├── src                        // 源代码  
│   ├── api                    // 所有请求    
│   ├── assets                 // 主题 字体等静态资源    
│   ├── components             // 全局公用组件     
│   ├── directive              // 全局指令     
│   ├── filtres                // 全局filter     
│   ├── mock                   // mock数据     
│   ├── router                 // 路由     
│   ├── store                  // 全局store管理     
│   ├── styles                 // 全局样式     
│   ├── utils                  // 全局公用方法     
│   ├── view                   // view     
│   ├── App.vue                // 入口页面     
│   └── main.js                // 入口 加载组件 初始化等     
├── static                     // 第三方不打包资源     
│   ├── jquery     
│   └── Tinymce                // 富文本     
├── .babelrc                   // babel-loader 配置     
├── eslintrc.js                // eslint 配置项     
├── .gitignore                 // git 忽略项     
├── favicon.ico                // favicon图标     
├── index.html                 // html模板     
└── package.json               // package.json     
这里来简单讲一下src文件

api 和 views

简单截取一下公司后台项目，现在后台大概有三十多个api模块


Paste_Image.png

如图可见，模块是很多的，而且随着业务的迭代，模块会越来越多。
所以这里建议根据业务模块来划分views，并且将views 和 api 两个模块一一对应，方便维护.如下图

Paste_Image.png
如article模块下放的都是文章相关的api，这样不管项目怎么累加，api和views的维护还是清晰的，当然也有一些全区公用的api模块，如七牛upload，remoteSearch等等，这些单独放置就行。

components

这里的components放置的都是全局公用的一些组件，如上传组件，富文本等等。一些页面级的组件建议还是放在各自views文件下，如图


Paste_Image.png
store

这里我个人建议不要为了用vuex而用vuex。就拿我司的后台项目来说，它虽然比较庞大，二三十个业务模块，十几种权限，但业务之间的耦合度是很低的，文章模块和评论模块几乎是俩个独立的东西，所以根本没有必要使用vuex来存储data，每个页面里存放自己的data就行。当然有些数据还是需要用vuex来统一管理的，如登录token,用户信息，或者是一些全局个人偏好设置等，还是用vuex管理更加的方便，具体当然还是要结合自己的业务场景的。总之还是那句话，不要为了用vuex而用vuex！

webpack

这里是用vue-cli为基础模板构建的，如果你对这个有什么疑惑请自行google，相关的配置介绍文章已经很详细了，这里就不再展开了。简单说一说需要注意到地方。
jquery

管理后台不同于前台项目，会经常用到一些第三方插件，但有些插件是不得不依赖jquery的，如市面上好的富文本基都是依赖jquery的，所以干脆就直接引入到项目中省事(gzip之后只有34kb，而且常年from cache,不要考虑那些吹毛求疵的大小问题，这几kb和提高的开发效率根本不能比)。但是如果第三方库的代码中出现$.xxx或jQuery.xxx或window.jQuery或window.$则会直接报错。要达到类似的效果，则需要使用webpack内置的ProvidePlugin插件，配置很简单，只需要

     new webpack.ProvidePlugin({
       $: 'jquery' ,
       'jQuery': 'jquery'
     })
这样当webpack碰到require的第三方库中出现全局的$、jQeury和window.jQuery时，就会使用node_module下jquery包export出来的东西了。

alias

当项目逐渐变大之后，文件与文件直接的引用关系会很复杂，这时候就需要使用alias了。
有的人喜欢alias 指向src目录下，再使用相对路径找文件

resolve: {
    alias: {
        '~': resolve(__dirname, 'src')
    }
}
//使用
import stickTop from '~/components/stickTop'
我习惯于

alias: {
  'src': path.resolve(__dirname, '../src'),
  'components': path.resolve(__dirname, '../src/components'),
  'api': path.resolve(__dirname, '../src/api'),
  'utils': path.resolve(__dirname, '../src/utils'),
  'store': path.resolve(__dirname, '../src/store'),
  'router': path.resolve(__dirname, '../src/router')
}

//使用
import stickTop from 'components/stickTop'
import getArticle from 'api/article'
没有好与坏对与错，纯看个人喜好和团队规范。

eslint

不管是多人合作还是个人项目，代码规范是很重要的。这样做不仅可以很大程度地避免基本语法错误，也保证了代码的可读性。这所谓工欲善其事，必先利其器，个人推荐eslint+vscode来写vue，绝对有种飞一般的感觉。效果如图：


eslintGif.gif

每次保存，vscode就能标红不符合eslint规则的地方，同时还会做一些简单的自我修正。安装步骤如下：
首先安装eslint插件


eslint1.png
安装并配置完成 ESLint 后，我们继续回到 VSCode 进行扩展设置，依次点击 文件 > 首选项 > 设置 打开 VSCode 配置文件,添加如下配置


    "files.autoSave":"off",
    "eslint.validate": [
       "javascript",
       "javascriptreact",
       "html",
       { "language": "vue", "autoFix": true }
     ],
     "eslint.options": {
        "plugins": ["html"]
     }
这样每次保存的时候就可以根据根目录下.eslintrc.js你配置的eslint规则来检查和做一些简单的fix。这里提供了一份我平时的eslint规则地址，都简单写上了注释。每个人和团队都有自己的代码规范，统一就好了，去打造一份属于自己的eslint 规则上传到npm吧，如饿了么团队的config,vue的config。

vscode 插件和配置推荐

封装axios

我们经常遇到一些线上的bug，但测试环境很难模拟。其实可以通过简单的配置就可以在本地调试线上环境。
这里结合业务封装了axios ，线上代码

import axios from 'axios';
import { Message } from 'element-ui';
import store from '../store';
// import router from '../router';

// 创建axios实例
const service = axios.create({
  baseURL: process.env.BASE_API, // api的base_url
  timeout: 5000                  // 请求超时时间
});

// request拦截器
service.interceptors.request.use(config => {
  // Do something before request is sent
  if (store.getters.token) {
    config.headers['X-Token'] = store.getters.token; // 让每个请求携带token--['X-Token']为自定义key 请根据实际情况自行修改
  }
  return config;
}, error => {
  // Do something with request error
  console.log(error); // for debug
  Promise.reject(error);
})

// respone拦截器
service.interceptors.response.use(
  response => response
  /**
  * 下面的注释为通过response自定义code来标示请求状态，当code返回如下情况为权限有问题，登出并返回到登录页
  * 如通过xmlhttprequest 状态码标识 逻辑可写在下面error中
  */
  // const code = response.data.code;
  // // 50014:Token 过期了 50012:其他客户端登录了 50008:非法的token
  // if (code === 50008 || code === 50014 || code === 50012) {
  //   Message({
  //     message: res.message,
  //     type: 'error',
  //     duration: 5 * 1000
  //   });
  //   // 登出
  //   store.dispatch('FedLogOut').then(() => {
  //     router.push({ path: '/login' })
  //   });
  // } else {
  //   return response
  // }
  ,
  error => {
    console.log('err' + error);// for debug
    Message({
      message: error.message,
      type: 'error',
      duration: 5 * 1000
    });
    return Promise.reject(error);
  }
)

export default service;
//使用
export function getInfo(params) {
  return _fetch({
    url: '/user/info',
    method: 'get',
    params
  });
}
比如后台项目，每一个请求都是要带token来验证权限的，这样封装以下的话我们就不用每个请求都手动来塞token，或者来做异常处理，一劳永逸。
而且因为我们的api是更具env环境变量动态切换的，如果以后线上出现了bug，我们只要将配置dev.env.js

module.exports = {
    NODE_ENV: '"development"',
    BASE_API: '"https://api-dev"', //修改为'"https://api-prod"'就行了
    APP_ORIGIN: '"https://wallstreetcn.com"' //为公司打个广告 pc站为vue+ssr
}
妈妈再也不用担心我调试线上bug了。
当然这里只是简单举了个例子，axios还可以执行多个并发请求，拦截器什么的，大家自行去研究吧。

多环境

vue-cli 默认只提供了dev和prod两种环境。但其实正真的开发流程可能还会多一个sit环境，就是所谓的测试环境。所以我们就要简单的修改一下代码。其实很简单就是设置不同的环境变量

"build:prod": "NODE_ENV=production node build/build.js",
"build:sit": "NODE_ENV=sit node build/build.js",
之后在代码里自行判断，想干就干啥

var env = process.env.NODE_ENV === 'production' ? config.build.prodEnv : config.build.sitEnv
新版的vue-cli也内置了webpack-bundle-analyzer 一个模块分析的东西，相当的好用。使用方法也很简单，和之前一样封装一个npm script 就可以。

//package.json
"build:sit-preview": "NODE_ENV=sit npm_config_preview=true  npm_config_report=true node build/build.js"

//之后通过process.env.npm_config_report来判断是否来启用webpack-bundle-analyzer

var BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
webpackConfig.plugins.push(new BundleAnalyzerPlugin())
效果图


analyzer.png

webpack-bundle-analyzer这个插件还是很有用的，对后期的代码优化什么的，最重要的是它够装逼~
前后端交互

每个公司都有自己一套的开发流程，没有绝对的好与坏。这里我来讲讲我司的前后端开发流程。

跨域问题

首先前后端交互不可避免的就会遇到跨域问题，我司现在全是用cors来解决的，如果你司后端嫌麻烦不肯配置的话，dev环境也可以通过
webpack-dev-server的proxy来解决，开发环境用nginx反代理一下就好了，具体配置这里就不展开了。

前后端的交流问题

其实大家也知道，平时的开发中交流成本占据了我们很大一部分时间，但前后端如果有一个好的协作方式的话能解决很多时间。我司开发流程都是前后端和产品讨论项目，之后 后端mock api 生成好文档，我们前端才是对接接口的。这里推荐一个文档生成器swagger
swagger是一个REST APIs文档生成工具，可以在许多不同的平台上从代码注释中自动生成，开源，支持大部分语言，社区好，总之就是一个神奇，给大家透露一下我司的api 文档(swagger自动生成，ui忽略)


Paste_Image.png

url地址，需要传是没参数，需要的传参类型，返回的数据格式什么都一清二楚了，我们大前端终于不用再看后端的脸色了~
前端自行mock

如果后端不肯来帮你mock数据的话，前端自己来mock也是很简单的。你可以使用mock server 或者使用mockjs+rap也是很方便的。
不久前出的easy-mock也相当的不错，还能结合swagger。

iconfont

element-ui 默认的icon不是很多，这里要安利一波阿里的iconfont简直是神器，不管是公司项目还是个人项目都在使用。它提供了png,ai,svg三种格式，同时使用也支持unicode，font-class，symbol三种方式。由于是管理后台对兼容性要求不高，楼主平时都喜欢用symbol，就是用svg的方式引入iconfont.js,之后就能欢快的去选icon了，还能自己上传icon。晒一波我司后台的图标(都是楼主自己发挥的)。


iconfont.png
router-view

different router the same component vue。真实的业务场景中，这种情况很多。比如


router-view.png

我创建和编辑的页面使用的是同一个component,默认情况下当这两个页面切换时并不会触发vue的created或者mounted钩子，官方说你可以通过watch $route的变化来做处理，但其实说真的还是蛮麻烦的。后来发现其实可以简单的在 router-view上加上一个唯一的key，来保证路由切换时都会重新渲染触发钩子了。这样简单的多了。
<router-view :key="key"></router-view>

computed: {
    key() {
        return this.$route.name !== undefined? this.$route.name + +new Date(): this.$route + +new Date()
    }
 }
优化

有些人会觉得现在构建是不是有点慢，我司现在技术栈是容器服务，后台项目会把dist文件夹里的东西都会打包成一个docker镜像，基本步骤为

npm install
npm run build:prod
加打包镜像，一共是耗时如下

Paste_Image.png
还是属于能接受时间的范围。
主站PC站基于nodejs、Vue实现服务端渲染，所以不仅需要依赖nodejs，而且需要利用pm2进行nodejs生命周期的管理。为了加速线上镜像构建的速度，我们利用taobao源 https://registry.npm.taobao.org 进行加速, 并且将一些常见的npm依赖打入了基础镜像，避免每次都需要重新下载。
这里注意下 建议不要使用cnpm install或者update 它的包都是一个link，反正会有各种诡异的bug，这里建议这样使用

npm install --registry=https://registry.npm.taobao.org
如果你觉得慢还是有可优化的空间如使用webpack dll 或者把那些第三方vendor单独打包 external出去，或者我司现在用的是http2 可以使用AggressiveSplittingPlugin等等，这里有需求的可以自行优化。

占坑

常规占坑，这里是手摸手，带你用vue撸后台系列。
完整项目地址：vue-element-admin
系类文章二： 手摸手，带你用vue撸后台 系列二(登录权限篇)
