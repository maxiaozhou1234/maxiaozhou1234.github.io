---
title: 【vue】 vue 初学问题小记
date: 2022-04-17 15:26:17
categories: vue
tags: [vue]

---

vue 初学遇到问题小记

## 1. 新建项目卡住

`vue init webpack yourProjectName`

Q：无法下载 webpack template 模板，导致 vue 项目初始卡住的问题

可能是无法连接 github 问题，或者下载慢，会导致无法下载模板，长时间卡在 init 步骤。

解决方案，通过本地的模板来初始化项目。在用户根目录新建 `.vue-templates` 文件夹，将 **[模板仓库](https://github.com/vuejs-templates/webpack/)** 打包好的模板下载下来 [zip链接](https://codeload.github.com/vuejs-templates/webpack/zip/refs/tags/1.3.1) 解压，修改文件夹名为 webpack，放入文件夹中。

最后，在命令行后添加 `--offline` 进行初始化项目

`vue init webpack yourProjectName --offline`

## 2. 新建项目最后一步卡住

使用 `vue init webpack projectName` 初始化，按要求填写信息或者回车后

Q：在最后选择 `npm` 或 `yarn` 可能导致初始化迟迟无法结束，长时间卡住。

解决方案，最后一步选择自己手动处理即可。

> Should we run `npm install` for you after the project has been created? (recommended) (Use arrow keys)
>
> Yes, use NPM
> Yes, use Yarn
> \> No, I will handle that myself

选择  No, I will handle that myself 手动处理即可。之后进入项目目录，执行

`npm install` 安装项目依赖

`npm run dev` 启动模板项目

默认地址 **http://localhost:8080** 访问项目。

## 3. vue 项目打包，部署

执行 `npm run build` 打包项目。

在目录中自动生成 **dist**  文件夹，将文件夹中的 **static** 文件夹 和 **index.html** 文件复制到项目目录即可完成部署。

###  自动化部署

将代码推送到服务器后，在服务器拉取最新的代码，并执行打包脚本（可修改打包生成的文件路径为服务器的部署路径），即可完成部署

或者，在本地打包后，触发 webpack 脚本将打包好的文件图推送到服务器指定路径实现项目部署。

## 4. vue 跨域问题

**跨域** 指浏览器不允许页面所在源去请求另一个源的数据。

**源指 协议、端口、域名。**

所以在本地开发过程中，需要添加代理配置，解决跨域的问题。否则页面在发起网络请求时，会提示跨域，无法请求。

**注：跨域是指协议、端口、域名三者有一个存在不同。**

如当前页面在 localhost:8080，但是里面的网络请求时 localhost:8899 那么就是跨域了，但是如果里面请求 http://www.baidu.com/s?wd=vuex 就不是跨域，因为他们的域名、端口都不一样。

### 开发阶段使用配置代理，允许接口跨域

在 ./config/index.js 添加配置 （vue2.0 版本，vue 3 有变化）

```javascript
module.export = {
	dev:{
        //...省略其他配置
        
       //代理信息
        proxyTable:{
            '/api':{
                target:'http://localhost:8897',//实际请求地址
                changeOrigin:true,//允许跨域
                wx:true,//websocket
                pathRewrite:{
                    '^/api':'',//重写路径，把 /api 这部分替换为后面的内容
                }
            }
        }
    }
}
```

这个代理是这么理解的，如果接口是 /api 开头，那么把 /api 替换为 target 中的内容，如果不是 /api 开头的接口，就保持不变。通过代理就可以保证我们需要跨域的接口能够正常访问，而内部的资源不经过代理保持原有的访问路径。

乌龙操作记录：

1. 第一次做项目，使用 axios 来做网络请求，为了方便就构建 axios 实例并配置了 `baseURL:http://host:port`，这样通过 axios 请求时传入 `url` 接口就可以。想法是后续换正式环境也方便。

2. 但是遇到了跨域问题，因为后端服务跑在本地，一顿操作把后端所有接口的 `response` 都加上允许所有来源，普通接口 get post 都正常请求。

3. 但是，测试到 **上传文件** 接口时，看控制台请求根本没有发出，被拦截提示存在跨域。后面才知道 vue 有代理配置，一顿配置之后还是不行。

   后来了解到代理细节，才明白代理配置的匹配是完整链接。如按照我原先 aixos 配置，那么我的请求链接是 `http://localhost:port/upload`，根本匹配不到代理，即使我请求时接口为 `/api/upload` 那么完整链接是 `http://localhost:port/api/upload` 也不会进入代理，因为代理期待拦截的接口是 `/api/upload`，再转为 `http://localhost:port/upload`

4. 所以，将 axios 实例的 baseURL 改为 /api，代理配置不变，就可以顺利上传文件，其他接口也可以正常访问。如果有需要访问其他域名的接口，发起请求时将 baseURl 重新赋值，就可以跳过代理。

### 正式环境跨域处理

正式环境中是没有代理，所以无法通过配置解决跨域问题。

在正式版本中使用代理，可使用跨资源共享 `CORS` ，具体使用找官网，暂没了解。

[cors](https://www.npmjs.com/package/cors)

```javascript
var koa = requier('koa');
//npm install --save koa2-cors
var cors = require('koa2-cors');
var appp = koa();
//开启
app.use(cors());
```



不过，最好的方式是通过后端配置代理。在 `Nginx` 添加配置端口，解决跨域的问题。

还有一种解决思路，在 vue 中开启服务器代理所有请求，再转发到服务器，也不会有跨域 问题。[http-proxy-middleware](https://www.npmjs.com/package/http-proxy-middleware)

## 5. 文件上传、下载

### 文件上传

和普通 post 请求一致，将文件通过 FormData 进行提交

不过，这 web 中是无法访问本地的文件系统，需要通过 `<input/>` 标签实现文件的选择，才能得到文件进行提交。

### 文件下载

#### 下载文件需要保存到本地

思路：通过动态生成节点挂载到 dom 中，触发 click() 操作，由浏览器接收下载动作，再将动态挂载的节点移除。

```javascript
function download(res){
    let blob = new Blob([res.data]);
    
    let downloadElement = document.createElement("a");
    let href = window.URL.createObjectURL(blob);//创建下载的链接
    downloadElement.href = href;
    downloadElement.download = `test.jpg`;//下载后文件名
    document.body.appendChild(downloadElement);
    downloadElement.click();//点击下载
    document.body.removeChild(downloadElement);//下载完成移除元素
    window.URL.revokeObjectURL(href);//释放blob对象
}
```



#### 下载文件，读取内容

下载接口中的文件（不使用浏览器方法），读取内容

```javascript
function(){
    this.result = 'loading';
    axios.get('/api/download?name=poet.txt',{
        responseType:'blob',//重要！否则识别为其他，而不是文件
    })
    .then((res)=>{
      	let blob = new Blob([res.data]);
        let reader = new FileReader();
        let app = this;
        reader.onload = function(){
            console.log('download end')
            let result = reader.result;
            
            app.poet = result;
        };
        reader.readAsText(blob);
    })
    .catch(err=>{
        console.log(err);
        this.result = '加载失败';
    });
}
```

参考链接：

[javascript Blob究竟是什么](https://segmentfault.com/a/1190000019932841)

## 6. js 使用两个感叹号判断

对与一些变量判断使用 !!，是为了处理 **null / undefined /0 ** 这类值

如：

```javascript
var a;
var b = !!a;//false
```

通过两个 **!!** 将定义的变量读取出来，保证拿到安全的结果。

## 7. js 函数重载

函数重载是其他高级语言具备的特性，指函数/方法有相同的名称，但函数的参数不同。

在 js 中，没有函数签名的概念，它的参数是由包含零或多个值得数组来标识。因为它没有函数签名，所以真正的重载是不可能做到的。

如果在 js 中定于了 **两个相同名称** 的函数/方法，那么后编写的方法会覆盖前一个方法。

但是，在函数中，可以通过对 **arguments** 来读取所有的入参

```javascript
function test(param1,param2){
	console.log(arguments.length);
}
```

通过对入参的判断，可以在 js 实现 **伪重载** 功能

```javascript
function test(){
    if(arguments.length==1){
        console.log('do logic 1.');
    }else{
        console.log('do logic 2.');
    }
}
```

参考：[javascript可以实现函数重载吗](https://www.php.cn/website-design-ask-487716.html)