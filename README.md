# 大牛工资条项目问题总结
总结在大牛工资条移动端(VUE)项目中出现的问题 

## v1.0上线版 2017/10/13-2017/11/2

## 1.默认电话号码字段点击会触发系统拨号
从网上查到可以通过添加meta头部标签来取消默认的识别电话号、邮箱等事件
<pre>
    &ltmeta name="format-detection" content="telephone=no" />  
    &ltmeta http-equiv="x-rim-auto-match" content="none">
</pre>
## 2.safari浏览器移动端兼容问题
#### safari浏览器对于input标签设定有默认的事件
该浏览器实质上使用的是 webkit 内核，所以在 input标签的css样式里面添加语句
<pre>
  -webkit-appearance:none; 
</pre>
这样就可以取消默认的事件了，不过如果只是用来 button 作用时，可以通过设置 a 标签来实现相同效果  
#### safari 默认点击 a，button，input时会有一层灰色的遮罩
<pre>
a,button,input,textarea{-webkit-tap-highlight-color: rgba(0,0,0,0;)}
</pre>
## 3.移动端click点击事件会有延迟
从点击屏幕上的元素到触发元素的 click 事件，移动浏览器会有大约 300 毫秒的等待时间。为什么这么设计呢？ 因为它想看看你是不是要进行双击（double tap）操作。所以引入<b>fastclick.js</b>

## 4. vue 异步请求之 axios
#### axios 是一个基于Promise 用于浏览器和 nodejs 的 HTTP 客户端，它本身具有以下特征:
从浏览器中创建 XMLHttpRequest  
从 node.js 发出 http 请求  
支持 Promise API  
拦截请求和响应  
转换请求和响应数据  
取消请求  
自动转换JSON数据  
客户端支持防止 CSRF/XSRF  

#### axios的配置
http://blog.csdn.net/sinat_17775997/article/details/69367204
<pre>
import axios from 'axios'
import qs from 'qs'
import * as _ from '../util/tool'
axios.defaults.timeout = 5000; //响应时间
axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded;charset=UTF-8'; //配置请求头
axios.defaults.baseURL = '你的接口地址'; //配置接口地址
//POST传参序列化(添加请求拦截器)
axios.interceptors.request.use((config) => {
	//在发送请求之前做某件事
    if(config.method  === 'post'){
        config.data = qs.stringify(config.data);
    }
    return config;
},(error) =>{
     _.toast("错误的传参", 'fail');
    return Promise.reject(error);
});
//返回状态判断(添加响应拦截器)
axios.interceptors.response.use((res) =>{
	//对响应数据做些事
    if(!res.data.success){
        // _.toast(res.data.msg);
        return Promise.reject(res);
    }
    return res;
}, (error) => {
    _.toast("网络异常", 'fail');
    return Promise.reject(error);
});
//返回一个Promise(发送post请求)
export function fetch(url, params) {
    return new Promise((resolve, reject) => {
        axios.post(url, params)
            .then(response => {
                resolve(response.data);
            }, err => {
                reject(err);
            })
            .catch((error) => {
               reject(error)
            })
    })
}
</pre>

## 5.移动端ios下触发input再点击其他区域无法失去焦点
在移动端的情况下，点击input时输入文字再点击确定按钮无法失去焦点，键盘依然存在.解决办法：
<pre>
//修复ios下点击其他区域 input不会失去焦点问题
    window.onload = function() {  
        document.querySelector('body').addEventListener('touchend', function(e) {  
            if(e.target.tagName.toLowerCase() != 'input') {  
              var inputLists=document.getElementsByTagName('input');
              for(var i=0;i&ltinputLists.length;i++){
                inputLists[i].blur();
              }  
            }  
        });  
    }
</pre>

## 6.移动端 ios下绑定企业微信开发接口无效
挖坑了好久，因为业务逻辑里面有一个302跳转，所以点击返回就会跳转回来，所以这里需要调取企业微信的接口写事件，逻辑写通以后，安卓端调通，ios无效。找了好久，兼容性研究许久无果。结果大佬来了一看是引入的文件错了。  
原引入文件：
<pre>
<script src="http://res.wx.qq.com/open/js/jweixin-1.2.0.js"></script>
</pre>  
大佬一说引入文件错了，恍然大悟，错就错在了这个 http协议啊。因为ios的安全拦截只支持https的安全链接，so...  
更正以后：
<pre>
<script src="//res.wx.qq.com/open/js/jweixin-1.2.0.js"></script>
</pre>

## 7.子组件和父组件
```js
//子组件中
//$emit自定义input事件，传递给父组件
this.$emit("input", this.realYear + "年" + this.realMonth + "月");

//父组件中
<din-date v-model="date"></din-date>
```

## 8.弹框模糊
部分居中的弹窗显示模糊，分析原因是在居中对齐时，使用了transform:translate(-50%,-50%);如果碰到计算50%的结果刚好是.5像素的时候（即像素值为单数），会导致Dom内的内容模糊。由于项目是兼容ie11+,所以这里可以使用flex布局实现居中对齐。

## 9.实现文本双行超出显示省略号
项目里有这样一个需求，用户上传的文本过长时超出部分未做判断，导致显示错乱。接到这样的bug我是崩溃的，因为前期并没有涉及这个需求，后期再改css时真的酸爽极了，因为用户自定义的数据实在太多了，所以就要狂加css来做限制了...  
  
  
首先查询到，单行隐藏的实现很简单
```
.more-text-cut {
  overflow: hidden; /*自动隐藏文字*/
  text-overflow: ellipsis; /*文字隐藏后添加省略号*/
  white-space: nowrap; /*强制不换行*/
}
```
但是双行超出显示省略号真的是很少见啊，不着急，网上一查解决了,因为用的webpack打包，还涉及到了-webkit-box-orient消失的问题：
```
.double-row {
  display: -webkit-box;
  /* autoprefixer: off */
  -webkit-box-orient: vertical; //如果直接写，webpack编译后此属性消失，需要上下两个注释包裹
  /* autoprefixer: on */
  -webkit-line-clamp: 2; //显示两行
  overflow: hidden;
  word-break: break-all; //单词折断
  line-height: 50px;
}
```
因为移动端是在企业微信，本身是webkit内核，兼容性完好，可这到了PC端就不行了。另外想到了另一种办法，直接通过after伪类来实现
```
.double-row {
  overflow: hidden;
  word-break: break-all; //单词折断
  line-height: 20px;
  max-height: 40px;
  position: relative;
  &::after {
    content: "...";
    position: absolute;
    z-index: 2;
    background: #fff;
    top: 0;
    right: 0;
    padding-left: 0.5em;
    margin-top: 20px;
  }
}
```
但是这种情况仅限于长度大于三行的情况，否则两行时都会显示“...”。  
同时想到了另一种情况，添加 filter 进行字节判断，当长度大于某个值时，显示...，但是这种情况还需要判断汉字字母数字的个数，同时可能会对原始数据的显示造成问题。最后重新改需求，达成IE情况下统一单行显示。

## 10.工资表详情页面双击跳到放大页面再返回不强制刷新
因为从放大页面返回到详情页面是不会有变化的所以不必须要created这些生命周期重新渲染。  
首先是如何判断当前是在放大页面跳回到详情页而不是其他页面跳转到详情页呢？  
```js
//app.vue
watch: {
    $route(to, from) {
      // from 对象中要 router 来源信息
      sessionStorage.setItem("fromRouter", from.path);
    }
  },
```
如上，在app.vue中对$route进行watch监听,然后将from.path写入缓存或者直接存入一个全局的变量供其它组件使用。  
接下来解决强制不刷新问题，用到了 vue中的 keep-alive，上代码
```js
//app.vue
<keep-alive>
     <router-view v-if="$route.meta.keepAlive"></router-view>
</keep-alive>
     <router-view v-if="!$route.meta.keepAlive"></router-view>
     
//路由中的配置 
 {
    path: '/salarydetail',
    name: 'salaryDetail',
    component: resolve => require(['@/page/salaryDetail'], resolve),
    meta: { keepAlive: true }//添加meta，keepAlive属性
  }
```
注意：keep-alive下就没有正常的created等等周期了，相反是activated和deactivated了

## 11.路由跳转时设置过渡效果，以及过渡时间的设置
直接在 router-view 外写入transition过渡标签，然后添加mode属性设置，避免过渡时的样式重叠（比如透明度切换时会造成重影）。
```js
   //mode: out-in 旧元素执行完过渡再执行新元素过渡 in-out则相反
   <transition name="jump" mode="out-in">
     <router-view></router-view>
   </transition>
```

## 12.路由绑定query的值时只能接受基本数据类型 Number、String、Boolean等
因为过去时要传递一个对象Object格式的，所以造成了传递过去的内容只是[object]，所以在传递的时候需要进行字符串的转化 JSON.stringfy（...）
