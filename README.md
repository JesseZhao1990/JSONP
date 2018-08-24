# JSONP

要了解jsonp.首先要了解同源策略，什么是同源策略呢？

##同源策略(Same Origin Policy)

浏览器出于安全方面的考虑，只允许与同域下的接口交互。
也就是说javascript只可以操作自己域下的东西，不能操作其他域下的东西。比如百度下javascript是不可操作谷歌下的页面。

### 同域指的是？

同协议：如都是http或者https

同域名：如都是baidu.com

同端口：如都是80端口

比如： 用户打开了 页面: http://baidu.com， 当前页面下的 js 向 http://baidu.com/xxx 的接口发 ajax 请求，浏览器是允许的。但假如向: http://google.com/xxx 发ajax请求则会被浏览器阻止掉。

### 那 JSONP是什么呢？

HTML 中 script 标签可以加载其他域下的js，比如我们经常引入一个其他域下线上cdn的jQuery。那如何利用这个特性实现从其他域下获取数据呢？

这样试试：
```
<script src="http://api.jirengu.com/weather.php?callback=showData"></script>

```

这个请求到达后端后，后端会去解析callback这个参数获取到字符串showData，在发送数据做如下处理：

```
之前后端返回数据： {"city": "hangzhou", "weather": "晴天"}
现在后端返回数据： showData({"city": "hangzhou", "weather": "晴天"})

```

前端script标签在加载数据后会把 「showData({"city": "hangzhou", "weather": "晴天"})」做为 js 来执行，这实际上就是调用showData这个函数，同时参数是 {"city": "hangzhou", "weather": "晴天"}。

用户只需要在加载提前在页面定义好showData这个全局函数，在函数内部处理参数即可。

```
<script>
    function showData(ret){
        console.log(ret);
    }
</script>
<script src="http://api.jirengu.com/weather.php?callback=showData"></script>

```

「原来这就是 JSONP(JSON with padding)，总结一下：」
1. JSONP是通过 script 标签加载数据的方式去获取数据当做 JS 代码来执行

2. 提前在页面上声明一个函数，函数名通过接口传参的方式传给后台，后台解析到函数名后在原始数据上「包裹」这个函数名，发送给前端。换句话说，JSONP 需要对应接口的后端的配合才能实现。


原理很简单，但用起来代码好丑陋，做个封装如何？

```

function jsonp(setting){
  setting.data = setting.data || {}
  setting.key = setting.key||'callback'
  setting.callback = setting.callback||function(){} 
  setting.data[setting.key] = '__onGetData__'

  window.__onGetData__ = function(data){
    setting.callback (data);
  }

  var script = document.createElement('script')
  var query = []
  for(var key in setting.data){
    query.push( key + '='+ encodeURIComponent(setting.data[key]) )
  }
  script.src = setting.url + '?' + query.join('&')
  document.head.appendChild(script)
  document.head.removeChild(script)

}

jsonp({
  url: 'http://api.jirengu.com/weather.php',
  callback: function(ret){
    console.log(ret)
  }
})

jsonp({
  url: 'http://photo.sina.cn/aj/index',
  key: 'jsoncallback',
  data: {
    page: 1,
    cate: 'recommend'
  },
  callback: function(ret){
    console.log(ret)
  }
})

```
演示地址：[点击开始演示](http://codepen.io/zhaojianxin/pen/KgXoAG?editors=0011)

文章来源：

1.https://zhuanlan.zhihu.com/p/22600501

2.http://blog.csdn.net/navy_xue/article/details/40016453

