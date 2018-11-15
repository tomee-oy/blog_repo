---
title: 第一个vuejs项目：他山之石
date: 2017-01-05 22:28:23
tags: [vue,技术]
categories: [项目,总结]
---
## 路由：vue-router（2.0版本中使用axios，vue-router已停止维护）
&emsp;&emsp;本项目中，路由配置作为一个单独的模块被放routers.js中，在路由配置中一定要通过import引入所有路由需要的组件：
```bash
    import Vue from 'vue'
    import VueRouter from 'vue-router'
    import MainPage from './components/Main'
    import SlideNav from './components/SlideNav'
    import TeaProduct from './components/TeaProduct'
    Vue.use(VueRouter);
    var routes=[
        {path:"/",component:MainPage},
        {
            path:"/index",
            component:SlideNav,
            children:[{
                path:'product',
                component:TeaProduct
            }]
        }
    ]
    var router=new VueRouter({
        routes
    })
    export default router
```
&emsp;&emsp;<font color=red>**Vue.use(VueRouter)**</font>这一句是将插件注册到Vue全局对象上去，如果没有这句，将会报插件未定义的错。在路由嵌套方面，vue2.0比1.0稍有改动，1.0用的是subRoutes，2.0则用的是<font color=red>**children**</font>进行子路由的定义。另外，子路由中的path属性前面不需要再加“/”，如果加了“/”，路由会报错。
&emsp;&emsp;**路由的使用**：在main.js中引入`router.js`，然后将router注册到要使用路由的Vue对象中：
```bash
    import router from './routers.js'
    new Vue({
        el: '#app',
        router,
        render: h => h(App)
    })
```
&emsp;&emsp;**组件中的路由跳转**：在组件中要进行页面跳转，首先要把router对象import进来，然后使用router的push方法进行跳转：
```bash
    import router from '../routers.js'
    router.push({path:'/index/productDetail',query:{teaId:this.teaList[groupIndex][index].teaId}});
```
&emsp;&emsp;要强调的是，在1.0版本中使用的是go方法进行路由跳转，2.0版本用的push。push方法的参数是一个对象，对象中包含路由路径、要传递的参数等参数。<font color=red>参数对象的名称是query而不是params。</font>
&emsp;&emsp;更多信息在[这里](http://router.vuejs.org/en/)

## 数据请求：vue-resource
&emsp;&emsp;数据请求模块封装在`utils.js`文件中作为项目公用模块，在main.js中以`Vue.prototype.utils`的方式将工具对象注册到Vue的原型对象中，这样，在整个项目中就可以直接对过Vue对象调用工具方法：
```bash
    import util from './utils.js'
    Vue.prototype.utils=util;
```
&emsp;&emsp;数据请求单独封装在ajax方法中，分别处理跨域和非跨域请求：
```bash
    ajax:function(host,url,params,callback){

        if(host==''){
            Vue.http.post(host+url,params).then(function(response){
                    callback(response);
                })
        }else{
            Vue.http.jsonp(host+url,params).then(function(response){
                    callback(response);
                })
        }
    }
```
&emsp;&emsp;参数解释：
&emsp;&emsp;&emsp;&emsp;host：请求的主机
&emsp;&emsp;&emsp;&emsp;url：请求的文件路径
&emsp;&emsp;&emsp;&emsp;params：请求参数
&emsp;&emsp;&emsp;&emsp;callback：请求成功后的回调函数
&emsp;&emsp;<font color=red>**误区：**</font>在http请求方法的参数中，第二个参数为请求参数对象，往往容易写成`{params:params}`的形式，这种方式是错的！应该是参数对象本身直接作为http请求的第二个参数。
&emsp;&emsp;更多信息在[这里](https://github.com/pagekit/vue-resource)

## 关于插件
&emsp;&emsp;在本项目中要求某些页面滚动条滚动到底部动态加载部分数据。起初，我在网上搜到了一个叫做`vue-scroll`的插件。但这个插件的滚动事件只能加在App.vue中的根节点上，如此，便不能在滚动事件中调用`router-view`渲染的路由组件中的方法，思维进入了死胡同，一直没能找到解决方案。最后只能转变思路：使用另外的滚动插件！终于迎来了曙光！
&emsp;&emsp;`vue-infinite-scroll`：一个很灵活的滚动插件，组件初始化的时候就会调用一次滚动回调函数。所以，如果在滚动回调函数里需要处理一些数据，而这些数据需要向后台请求回来后才能用，那么在刚加载组件的时候要注意禁用scroll插件。也就是说，把`infinite-scroll-disabled="busy"`中busy的值设为true，等到回调函数中需要的数据全部请求回来以后，在http的回调函数中再把busy的值设为false，即解除禁用。
&emsp;&emsp;更多关于`vue-infinite-scroll`的信息在[这里](https://github.com/ElemeFE/vue-infinite-scroll)

## 其他要点
&emsp;&emsp;***** 组件style加上<font color=red>scoped</font>标签是限制style中的样式只在本组件中起作用，不会污染其他组件。  
&emsp;&emsp;***** 在做侧边栏滑动动画的时候，使用了@keyframes，也做了各种兼容，在ios系统上可以正常运行，但在安卓机上始终不起作用。后来把animation换成了transiton实现就解决了。animation要兼容安卓系统是否需要其他特殊的处理方式，有待考证！
