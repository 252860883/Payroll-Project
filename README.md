# 大牛工资条项目问题总结
总结在大牛工资条移动端项目中出现的问题

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
挖坑了好久，因为业务逻辑里面有一个302跳转，所以返回键失效，需要调取企业微信的接口写事件，逻辑写通以后，安卓端调通，ios无效。找了好久，兼容性研究许久无果。结果大佬来了一看是引入的文件错了。  
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

