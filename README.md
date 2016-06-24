# 使用nodejs 快速搭建服务器
### 1. 什么是nodejs
Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行环境的编程语言。Node.js 使用了一个事件驱动、非阻塞式 I/O 的模型，使其轻量又高效。Node.js 的包管理器 npm，是全球最大的开源库生态系统。（摘自http://nodejs.cn/）

### 2. 发展史
2009年2月，Ryan Dahl在博客上宣布准备基于V8创建一个轻量级的Web服务器并提供一套库。
2009年5月，Ryan Dahl在GitHub上发布了最初版本的部分Node.js包，随后几个月里，有人开始使用Node.js开发应用。
2009年11月和2010年4月，两届JSConf大会都安排了Node.js的讲座。
2010年年底，Node.js获得云计算服务商Joyent资助，创始人Ryan Dahl加入Joyent全职负责Node.js的发展。
2011年7月，Node.js在微软的支持下发布Windows版本。
（摘自百度知道）

### 3. 安装
##### OSX
* 登陆官网：https://nodejs.org/en/
* 点击下载（2个版本，稳定版、最新版）
* 点击安装

##### Linux
* 登陆官网：https://nodejs.org/en/
* 点击 DOWNLOADS
* 点击 Installing Node.js via package manager
* 查看文档 https://nodejs.org/en/download/package-manager/#void-linux
* 执行命令 curl --silent --location https://rpm.nodesource.com/setup_6.x | bash - 最新版本
* 执行命令 yum -y install nodes
* 如果要用相关插件，还需要安装编译环境  执行命令yum install gcc-c++ make
* 由于gcc-build 4.8版本不能直接用yum安装，因此需要手动安装。

##### gcc-build 4.8安装
* 下载gcc 4.8.1源码包：http://ftp.tsukuba.wide.ad.jp/software/gcc/releases/gcc-4.8.1/gcc-4.8.1.tar.bz2
* 解压：tar -jxvf gcc-4.8.1.tar.bz2
* 进入gcc-4.8.1文件夹：cd gcc-4.8.1
* 执行命令：./contrib/download_prerequisites
* 新建文件夹：mkdir gcc-build-4.8.1
* 进入新建文件夹：cd gcc-build-4.8.1
* 执行命令：./gcc-4.8.1/configure --enable-checking=release --enable-languages=c,c++ --disable-multilib
* 编译：make -j4
* 安装：sudo make install

### 4. Nodejs 主流框架
* Express：Express 框架提供了用来开发强壮的 web/移动应用，以及 API 的所有功能。并且开发人员还能够方便地为它开发插件和扩展，从而增加 Express 的能力。
* KOA：KOA 是 node.js mvc 框架的后起之秀，在2013第四个季度才发布了第一个版本。开发 KOA 的人员基本来自 Express 开发团队，TJ Holowaychuk 是 KOA 开发团队的领导者。虽然 KOA 大部分开发人员来自 Express，但是他们使用了完全不同的技术来开发 KOA，并且 KOA 正成为 Express 一个强有力的竞争对手。
* Meteor: Meteor 框架是 Node.js 上最出色的全栈框架。项目在 GitHub 上有 28K+ 的赞，拥有大量的自定义包，庞大的社区支持，非常好的教程和文档。在这个领域 Meteor 毫无疑问是王者，你可以用它构建纯 Javascript 的实时 Web 和 手机应用。

### 5. 技术选型
由于是新项目，不用考虑与老项目兼容，另外服务仅作为前端服务器，不会去做具体业务，仅做数据格式处理、透传、上传、下载等功能，因此选用灵活的KOA作为开发框架。koa安装npm install koa

### 6. KOA中间件选用
* koa-router: koa路由中间件，做路由处理。 npm install koa-router
* koa-static：koa静态文件服务，处理静态文件。 npm install koa-static
* koa-session：koa处理session中间件。 npm install koa-session
* koa-body：koa处理request body中间件。 npm install koa-body
* koa-csrf：koa csrf漏洞预防中间件。 npm install koa-csrf
* koa-convert：koa 语法兼容处理中间件（由于有些中间件不支持asyn wait语法） npm install kia-convert

### 7. 第三方插件选用
* lodash：字符串、数组、对象、集合等数据处理类库。
* canvas：图像处理。npm install canvas
* validator：辅助验证。 npm install validator
* node-xlsx：生成excel文档。 npm install node-xlsx
* co-views：HTML模版处理插件，使用swig模版。 npm install co-views
* path: 合并路径
* fs：文件处理

### 8. KOA使用例子
```
const app = require(‘koa')(); //初始化koa
const convert = require('koa-convert'); 
const router = require(‘koa-router’)();
const serve = require(‘koa-static');
const csrf = require(‘koa-csrf');
const session = require('koa-session');
const koaBody = require(‘koa-body’)();
const fs = require(‘fs’);
const Canvas = require('canvas');
app.keys = ['test'];

router.get('/', function *(next) {
    this.body = 'Hello World';
});

router.get('/getCaptcha', function *(){
    let captcha = fs.readFileSync('captcha.png'); //读取图片
    let canvas = new Canvas(105, 30); //初始化canvas
    let Image = Canvas.Image;
    let img = new Image;
    img.src = captcha;
    let ctx = canvas.getContext(‘2d');
    ctx.drawImage(img, 0, 0);
    ctx.fillText(‘hello’ ,5, 24,50);
    ctx.fillRect();
    this.body = canvas.toBuffer();
})

router.post(‘/post’, koaBody, function *(){
    try {
        this.assertCSRF(this.request.body); //检测csrf攻击
        this.body = this.session; //读session
    }catch(e){
         this.status = 403;
         this.body = "trying csrf?";    
    }
   });

app.use(session(app));
app.use(csrf());
app.use(convert(serve('static/')));
app
    .use(convert(router.routes()))
    .use(convert(router.allowedMethods()));
app.listen(3000);

```

