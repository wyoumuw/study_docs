#JSONP的原理
##同源策略
这里不介绍，大概就是ajax请求获取资源数据的时候会跨域，浏览器会禁止跨域获取资源。
##解决方案
1.在跨域的前端建立个后台，这样可以同域下调用

2.后台写入cors的header
    
     Access-Control-Allow-Origin：http://test.blds.zwjk.com
     Access-Control-Allow-Methods：POST,GET,DELETE,PUT
     Access-Control-Allow-Headers：x-requested-with,content-type
     Access-Control-Allow-Credentials：true

3.jsonp

在同源策略里，是允许src属性跨域的，那么\<script>\<img>这种标签其实是可以跨域的例如你可以用img引用其他服务器的图片一样。

jsonp其实就是走了这样一个方式，先来看个例子：

        //a.js
        say("hello");
        
        //b.html
        ....
        <script>
            function say(data){
            alert(data)
            }
        </script>
        <script src="./a.js"></script>
        ....
        
上面代码写完后执行b.html会直接跳出alert.输出hello。

jsonp其实就是把数据封装成函数，我们换个想法，如果把接口的数据看成js会怎么样。比如：

接口action，它的返回值是{"data":"hello"}，如果我们用ajax在跨域去取，因为同源策略，那么其实我们是取不到的。

我们换个想法，我们直接写

        <script src="host:port/action"></script>

那么页面上就会有一个对象是{"data":"hello"},但是因为没有引用，我们根本没法拿到。但是我们再换个想法，把服务器返回的数据改一改成:

        //action
        callback({"data":"hello"})
        
        //b.htm
        <script src="host:port/action"></script>
        function callback(data){
            alter(data.data)
        }
这样就可以调用自己的方法了。以上就是jsonp的原理了，其实就是巧妙的利用了src不遵循同源策略出现的。jquery里面帮我实现的jsonp他是自生成的callback，并且生成对应的function然后在function里调用success方法。