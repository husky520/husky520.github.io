---
layout: master
title:  "MongoDB + Express 搭建电影网站笔记"
categories: node.js
---

> imooc 网项目源码: https://github.com/shinytang6/imoocMovie
> 项目地址: https://github.com/husky520/movie-site.git

## 0. 准备工作

### homebrew 安装 mongodb
{% highlight text %}
brew install mongodb
{% endhighlight %}
*注意：*
*如果出现以下问题*
{% highlight text %}
mongodb: A full installation of Xcode.app 8.3.2 is required to compile this software.
Installing just the Command Line Tools is not sufficient.
Xcode 8.3.2 cannot be installed on macOS 10.11.
You must upgrade your version of macOS.
Error: An unsatisfied requirement failed this build.
{% endhighlight %}
*可以直接安装低版本的 mongodb, 例如 mongodb@3.4*

创建默认数据库存放位置:
{% highlight text %}
sudo mkdir /data
sudo mkdir /data/db
{% endhighlight %}

启动数据库
{% highlight text %}
sudo mongod
{% endhighlight %}


***

## 1. 项目简介
项目 UI 使用 bootstrap, 后台使用 node.js 搭建, 具体为 express 搭建服务器, moogoose 连接 mongodb 数据库, jade 作为模版引擎, 另外还有 moment.js 用来格式化时间等 ...

### 目录结构预览
{% highlight text %}
- movie-site         // 项目目录
  - schemas          // mongoose schemas
    movie.js         // scheme
  - models           // mongoose models
    movie.js         // model
  - views            // 静态资源目录
    - includes       // 页面所包含的公共部分模版
      head.jade      // bootstrap 和 jquery 引入
      header.jade    // 页面基本信息
    - pages          // 不同页面内容模版
      detail.jade    // 电影详情页
      list.jade      // 电影列表页
      index.jade     // 电影网站首页
      admin.jade     // 后台录入页
    layout.jade      // 布局框架
  app.js             // 入口文件
{% endhighlight %}

### 页面路由
{% highlight text %}
首页     http://localhost:3000/
电影列表  http://localhost:3000/list
电影详情  http://localhost:3000/movie/:id
后台录入  http://localhost:3000/admin/movie
后台更新  http://localhost:3000/admin/update/:id
{% endhighlight %}

***

## 2. 基本页面和初步的入口文件

- **新建 app.js 入口文件**
{% highlight javascript %}
const express = require('express')
const path = require('path')
const bodyParser = require('body-parser')

const port = process.env.PORT || 3000
const app = express()

app.set('views', './views')
app.set('view engine', 'jade')

app.use(bodyParser.urlencoded({extended: false}))
app.use(bodyParser.json())

app.use(express.static(path.join(__dirname, 'bower_components')))
app.listen(port)

console.log('Movie Site 已启动 ! 端口为: ', port)

// 配置路由
// index page
app.get('/', (req, res) => {
    res.render('pages/index', {
        title: '电影首页',
        movies: movies  // 传入 mock 数据
    })
})

// movie page
app.get('/movie/:id', (req, res) => {
    res.render('pages/detail', {
        title: '电影详情: ' + movie.title,
        movie: movie
    })
})

// list page
app.get('/list', (req, res) => {
    res.render('pages/list', {
        title: '电影列表',
        movies: movies
    })
})

// admin page
app.get('/admin/movie', (req, res) => {
    res.render('pages/admin', {
        title: '后台录入',
        movie: {  // 录入时传入空数据给页面表单
            doctor: '',
            country: '',
            title: '',
            year: '',
            poster: '',
            language: '',
            flash: '',
            summary: ''
        }
    })
})

// admin update movie 和录入同页面
app.get('/admin/update/:id', function (req, res) {
    res.render('pages/admin', {
        title: '更新电影：' + movie.title,
        movie: movie  // 更新时传入原来数据, 在此基础上修改
    })
})

// admin post movie 该页面为接收录入, 更新页提交的数据
app.post('/admin/movie/new', function (req, res) {
    // ...
})
{% endhighlight %}

***

- **各页面模版**

layout.jade
{% highlight jade %}
doctype
html
    head
        meta(charset="utf-8")
        title #{title}
        include ./includes/head
    body
        include ./includes/header
        block content
{% endhighlight %}

head.jade  - **( 此处使用绝对路径, 否则子页面无法正确访问到静态资源 ! )**
{% highlight jade %}
link(href="/bootstrap/dist/css/bootstrap.css", rel="stylesheet")
scritp(src="/jquery/dist/jquery.js")
script(src="/bootstrap/dist/js/bootstrap.js")
{% endhighlight %}

header.jade
{% highlight jade %}
.container
    .row
        .page-header
            h1= title
{% endhighlight %}

index.jade
{% highlight jade %}
extends ../layout

block content
    .container
        .row
            each item in movies
                .col-6.col-md-3
                    .thumbnail
                        a(href="/movie/#{item._id}")
                            img(src="#{item.poster}", alt="#{item.title}")
                        .caption
                            h3 #{item.title}
                            p: a.btn.btn-primary(href="/movie/#{item._id}", role="button") 欢迎观看预告片
{% endhighlight %}

list.jade
{% highlight jade %}
extends ../layout

block content
    .container
        .row
            table.table.table-hover.table-bordered
                thead
                    tr
                        th 电影名字
                        th 导演
                        th 国家
                        th 上映年份
                        th 录入时间
                        th 查看
                        th 更新
                        th 删除
                tbody
                    each item in movies
                        tr(class="item-id-#{item._id}")
                            td #{item.title}
                            td #{item.doctor}
                            td #{item.country}
                            td #{item.year}
                            td #{moment(item.meta.createdAt).format('MM/DD/YYYY')}
                            td: a(target="_blank", href="../movie/#{item._id}") 查看
                            td: a(target="_blank", href="../admin/update/#{item._id}") 修改
                            td
                                button.btn.btn-danger.del(type="button", data-id="#{item._id}") 删除

{% endhighlight %}

detail.jade
{% highlight jade %}
extends ../layout

block content
    .container
        .row
            .col-12
                video(src="#{movie.flash}", width="100%", controls="controls")
            .col-12
                dl.dl-horizontal
                    dt 电影名字
                    dd= movie.title
                    dt 导演
                    dd= movie.doctor
                    dt 国家
                    dd= movie.country
                    dt 语言
                    dd= movie.language
                    dt 上映年份
                    dd= movie.year
                    dt 简介
                    dd= movie.summary
{% endhighlight %}

admin.jade
{% highlight jade %}
extends ../layout

block content
    .container
        form(method="post", action="/admin/movie/new")
            //- 使用隐藏的 input 标签提交电影的 id
            input(type="hidden", name="_id", value="#{movie._id}")
            .form-group.row
                label.col-sm-2.col-form-label(for="inputTitle") 电影名字
                .col-sm-10
                    input#inputTitle.form-control(type="text", name="title", value="#{movie.title}")
            .form-group.row
                label.col-sm-2.col-form-label(for="inputDoctor") 电影导演
                .col-sm-10
                    input#inputDoctor.form-control(type="text", name="doctor", value="#{movie.doctor}")
            .form-group.row
                label.col-sm-2.col-form-label(for="inputCountry") 国家
                .col-sm-10
                    input#inputCountry.form-control(type="text", name="country", value="#{movie.country}")
            .form-group.row
                label.col-sm-2.col-form-label(for="inputLanguage") 语种
                .col-sm-10
                    input#inputLanguage.form-control(type="text", name="language", value="#{movie.language}")
            .form-group.row
                label.col-sm-2.col-form-label(for="inputPoster") 海报地址
                .col-sm-10
                    input#inputPoster.form-control(type="text", name="poster", value="#{movie.poster}")
            .form-group.row
                label.col-sm-2.col-form-label(for="inputFlash") 片源地址
                .col-sm-10
                    input#inputFlash.form-control(type="text", name="flash", value="#{movie.flash}")
            .form-group.row
                label.col-sm-2.col-form-label(for="inputYear") 上映年份
                .col-sm-10
                    input#inputYear.form-control(type="text", name="year", value="#{movie.year}")
            .form-group.row
                label.col-sm-2.col-form-label(for="inputSummary") 电影简介
                .col-sm-10
                    textarea#inputSummary.form-control(name="summary")
            .form-group.row
                .offset-sm-2.col-sm-10
                    button.btn.btn-primary(type="submit") 录入
{% endhighlight %}

***

- **mongoose 创建数据库模型**

在 schemas 目录下创建 movie.js 文件, 导出 MovieSchema

movie.js
{% highlight javascript %}
const mongoose = require('mongoose')

// 设计数据库结构
const MovieSchema = new mongoose.Schema({
    doctor: String,
    title: String,
    language: String,
    country: String,
    summary: String,
    flash: String,
    poster: String,
    year: Number,
    meta: {
        createAt: {
            type: Date,
            default: Date.now()
        },
        updateAt: {
            type: Date,
            default: Date.now()
        }
    }
})

// Schema 钩子, 以下用以在保存时创建或更新修改时间
MovieSchema.pre('save', function (next) {
    if (this.isNew) {
        this.meta.createAt = this.meta.updateAt = Date.now()
    } else {
        this.meta.updateAt = Date.now()
    }
    next()
})

// 静态方法, fetch 查询所有数据, findById 查询单条数据
MovieSchema.statics = {
    fetch: function(cb) {
        return this
            .find({})
            .sort('meta.updateAt')
            .exec(cb)
    },
    findById: function(id, cb) {
        return this
            .findOne({_id: id})
            .exec(cb)
    }
}

module.exports = MovieSchema

{% endhighlight %}

在 models 下创建 movie.js, 导出 model
{% highlight javascript %}
const mongoose = require('mongoose')
const MovieSchema = require('../schemas/movie')

const Movie = mongoose.model('Movie', MovieSchema)

module.exports = Movie
{% endhighlight %}

***

- **连接数据库**

app.js
{% highlight javascript %}
const mongoose = require('mongoose')
mongoose.connect('mongodb://localhost/movie-site')  // movie-site 为数据库名
{% endhighlight %}

***

- **数据的增删改查**

首先引入创建的 model
{% highlight javascript %}
const Movie = require('./models/movie')
{% endhighlight %}

增
{% highlight javascript %}
// 创建一个数据
const data = new Movie({
    // 数据内容
    // title: 'title'
    // ....
})

// 存入数据
Movie.save(function (err, data) {
    if (err) {
        console.error(err)
    }
    // 其他操作 ...
})
{% endhighlight %}

删
{% highlight javascript %}
// 删除 _id 为 id 的项, 并为结果作出相应的回应
Movie.remove({'_id': id}, function (err) {
    if (err) {
        console.error(err)
        res.json({status: 0})
    } else {
        res.json({status: 1})
    }
})
{% endhighlight %}

改
{% highlight javascript %}
// 这里使用一个插件 ‘underscore’
const underscore = require('underscore')

// 在原数据的基础上拓展新数据 (更新数据)
const data = underscore.extend(oldData, newData)

// 存入数据库
Movie.save(function (err, data) {
    if (err) {
        console.error(err)
    }
    // 其他操作 ...
})
{% endhighlight %}

查
{% highlight javascript %}
// 使用在 ‘/schemas/ movie.js’ 中所封装的静态方法进行查询

// 查询所有数据
Movie.fetch(function (err, data) {
    if (err) {
        console.error(err)
    }
    // 其他对已查询到的 data 的操作 ...
})

// 按 id 查找数据
Movie.findById(function (err, data) {
    if (err) {
        console.error(err)
    }
    // 其他对已查询到的 data 的操作 ...
})
{% endhighlight %}

***

## 3. 注意点
> 1. ***jQuery.slim.js*** 是精简版, 其中不含 $.ajax
> 2. 在列表页删除了一条数据后, 可以**刷新**页面, 以保证列表页和数据库的同步 (无法通过重定向实现刷新)


***

最终的源码参考开头的链接