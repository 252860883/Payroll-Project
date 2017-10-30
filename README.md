# 大牛工资条项目问题总结
总结在大牛工资条移动端项目中出现的问题

## 1.默认电话号码字段点击会触发系统拨号
从网上查到可以通过添加meta头部标签来取消默认的识别电话号、邮箱等事件
<pre>
    &ltmeta name="format-detection" content="telephone=no" />  
    &ltmeta http-equiv="x-rim-auto-match" content="none">
</pre>
## 2.safari浏览器有默认的样式效果
safari浏览器对于input标签设定有默认的事件，该浏览器实质上使用的是 webkit 内核，所以在 input标签的css样式里面添加语句
<pre>
  -webkit-appearance:none; 
</pre>
这样就可以取消默认的事件了，不过如果只是用来 button 作用时，可以通过设置 a 标签来实现相同效果
## 3.移动端click点击事件会有延迟
从点击屏幕上的元素到触发元素的 click 事件，移动浏览器会有大约 300 毫秒的等待时间。为什么这么设计呢？ 因为它想看看你是不是要进行双击（double tap）操作。所以引入<b>fastclick.js</b>

## 4. vue 异步请求之 axios
