---
layout: post
title: "Rails中跨域请求的解析"
date: "2014-10-11 15:44:31 +0800"
description: 简单介绍了CORS和JSONP的一些概念，以及Rails中如何实现cors和jsonp的支持。
comments: true
---

#### 如何进行跨域资源请求

可以通过以下两种方式进行跨域资源请求：

* JSONP
* CORS(cross-origin resource sharing: 跨来源资源共享)

#### 什么是JSONP

关于JSONP,以下是来自[维基百科](http://zh.wikipedia.org/wiki/JSONP)的介绍：

> JSONP（JSON with Padding）是资料格式 JSON 的一种“使用模式”，可以让网页从别的网域要资料。另一个解决这个问题的新方法是跨来源资源共享。

> 由于同源策略，一般来说位于 server1.example.com 的网页无法与不是 server1.example.com 的服务器沟通，而 HTML 的 `<script>` 元素是一个例外。利用 `<script>` 元素的这个开放策略，网页可以得到从其他来源动态产生的 JSON 资料，而这种使用模式就是所谓的 JSONP。用 JSONP 抓到的资料并不是 JSON，而是任意的 JavaScript，用 JavaScript 直译器执行而不是用 JSON 解析器解析。

所有支持Javascript的浏览起都支持[同源策略]
(http://zh.wikipedia.org/wiki/%E5%90%8C%E6%BA%90%E7%AD%96%E7%95%A5)
(来自维基百科)，而html中的img，script等标签是不用遵循同源策略的。JSONP相当与利用了script的这个特点，去加载不同origin的资源。

JSONP的原理，简单来说就是加载不同origin的脚本，然后执行。一般来说，一些资源都是一些数据，比如JSON格式，而script加载的是js文件并执行，那么要求返回的是一个可执行的js文件，而服务器端只提供数据，并不知道客户端要如何处理这个数据。所以一般客户端可以写一个回调函数，然后把回调函数的名字告诉服务器端，而服务器端会将json数据作为参数，包含上回调函数的名字，然后将wrapper后的内容作为js文件返回。

具体流程可以参照下图：

![jsonp处理流程图](/images/2014-10-11-cross-origin-resource-sharing/jsonp处理流程图.png "jsonp处理流程图")

<!-- more -->

#### 什么是CORS

关于CORS,以下是来自[维基百科](http://zh.wikipedia.org/wiki/%E8%B7%A8%E4%BE%86%E6%BA%90%E8%B3%87%E6%BA%90%E5%85%B1%E4%BA%AB)的介绍：

> 跨来源资源共享（CORS）是一份浏览器技术的规范，提供了 Web 服务从不同网域传来沙盒脚本的方法，以避开浏览器的同源策略[1]，是 JSONP 模式的现代版。与 JSONP 不同，CORS 除了 GET 要求方法以外也支援其他的 HTTP 要求。用 CORS 可以让网页设计师用一般的 XMLHttpRequest，这种方式的错误处理比 JSONP 要来的好。另一方面，JSONP 可以在不支援 CORS 的老旧浏览器上运作。现代的浏览器都支援 CORS[2]。

CORS是W3C的一个[规范](http://www.w3.org/TR/cors/)，规定了各个用户agent（浏览器）以及Web服务器端按照什么流程来支持跨域请求。

关于CORS规范的内容，可以参照w3c的[链接](http://www.w3.org/TR/cors/)，也可以参照
[【孟子E章】](http://blog.csdn.net/net_lover)
的
[这篇博客](http://blog.csdn.net/net_lover/article/details/5172509)，
里面介绍了CORS的跨域资源请求的细节。

CORS规范对跨域请求分为两种：

* simple request(简单请求)
* preflight request(预检请求)

针对simple request，只需要浏览器请求一次，而针对preflight是需要浏览器事先进行一次OPTION method请求。如果OPTION请求的结果是OK的话，才会发送真正的请求。


#### JSONP和CORS的区别

JSONP是比较老的解决跨域请求的方式，CORS是比较新的一种方式。从实现的原理上来讲完全是两个不同的东西。

* JSONP只能支持GET请求，而CORS可以支持所有的HTTP请求。
* CORS在一些老式的浏览器不被支持，而JSONP可以在老式的浏览器上使用。关于CORS浏览器的支持情况，可以参照维基百科的链接。比如至少知道的Opera10.61为止没有实现CORS。

#### Rails中支持JSONP

Rails本身就是支持jsonp的，在render的时候，可以通过传递`callback`option来直接返回jsonp的内容。示例代码如下：

``` ruby
render :json => {echo: "hello world"}, :callback => "foo"
```

返回的请求结果js内容如下：

```
/**/foo({"echo":"hello world"})
```

其实就是调用foo函数处理json数据，这样自己在client端书写自己的foo函数，就可以实现跨域资源请求了。

还有另外一种实现jsonp的方式就是使用
[rack-jsonp-middleware](https://github.com/robertodecurnex/rack-jsonp-middleware)
这个gem。这个gem是一个rack middleware，通过查看他的源代码，他的处理流程如下：

1. 判断是否是jsonp的请求。（通过判断后缀是否为jsonp）
2. 如果是jsonp的请求，那么会把请求path中的jsonp，修改为json后缀，这样就变成了一个json请求。
3. 调用正常的rack call处理，得到[status, header, body]三元组。这里的body内容其实就是json数据。
4. 使用callback参数的值包装body内容。`body = ["/**/#{callback}(#{json});"]`，这样body内容就成了一个正常js文件的内容。
5. 修改header的Content-Type为`application/javascript`。

rack-jsonp-middleware（v0.0.10）关键源码如下：  
<https://github.com/robertodecurnex/rack-jsonp-middleware/blob/rack-jsonp-middleware-0.0.10/lib/rack/jsonp.rb#L12>

``` ruby
def call(env)
  request = Rack::Request.new(env)
  requesting_jsonp = Pathname(request.env['PATH_INFO']).extname =~ /^\.jsonp$/i
  callback = request.params['callback']

  return [400,{},[]] if requesting_jsonp && !self.valid_callback?(callback)

  if requesting_jsonp
    env['PATH_INFO'] = env['PATH_INFO'].sub(/\.jsonp/i, '.json')
    env['REQUEST_URI'] = env['PATH_INFO']
  end

  status, headers, body = @app.call(env)

  if requesting_jsonp && self.json_response?(headers['Content-Type'])
    json = ""
    body.each { |s| json << s }
    body = ["/**/#{callback}(#{json});"]
    headers['Content-Length'] = Rack::Utils.bytesize(body[0]).to_s
    headers['Content-Type'] = headers['Content-Type'].sub(/^[^;]+(;?)/, "#{MIME_TYPE}\\1")
  end

  [status, headers, body]
end

```




#### Rails中支持CORS

Rails应用中可以使用rack-cors
[(github地址)](https://github.com/cyu/rack-cors)
这个gem来处理。

他的部分源码（`David add:`开始的行是我自己添加的注释）:

[github源代码链接(版本：v0.2.9)](https://github.com/cyu/rack-cors/blob/v0.2.9/lib/rack/cors.rb#L29)

``` ruby 	

def call(env)
  env['HTTP_ORIGIN'] = 'file://' if env['HTTP_ORIGIN'] == 'null'
  env['HTTP_ORIGIN'] ||= env['HTTP_X_ORIGIN']

  cors_headers = nil
  if env['HTTP_ORIGIN']
    
    # David add: 如果http头部有origin的话，那么被认为是跨域的请求
    debug(env) do
      [ 'Incoming Headers:',
        "  Origin: #{env['HTTP_ORIGIN']}",
        "  Access-Control-Request-Method: #{env['HTTP_ACCESS_CONTROL_REQUEST_METHOD']}",
        "  Access-Control-Request-Headers: #{env['HTTP_ACCESS_CONTROL_REQUEST_HEADERS']}"
        ].join("\n")
    end
    
    # David add: 这里判断是否是OPTIONS，因为preflight请求会由浏览器先发送一个option方法的请求
    if env['REQUEST_METHOD'] == 'OPTIONS' and env['HTTP_ACCESS_CONTROL_REQUEST_METHOD']
    
      # David add: 这里是处理preflight request
      if headers = process_preflight(env)
        debug(env) do
          "Preflight Headers:\n" +
              headers.collect{|kv| "  #{kv.join(': ')}"}.join("\n")
        end
        
        # David add: 这里如果cors的配置中针对访问的资源，允许跨域请求的话，那么返回200（OK）
        return [200, headers, []]
      end
    else
      # David add: 这里是处理simple request
      cors_headers = process_cors(env)
    end
  end
  status, headers, body = @app.call env
  if cors_headers
    headers = headers.merge(cors_headers)

    # http://www.w3.org/TR/cors/#resource-implementation
    unless headers['Access-Control-Allow-Origin'] == '*'
      vary = headers['Vary']
      headers['Vary'] = ((vary ? vary.split(/,\s*/) : []) + ['Origin']).uniq.join(', ')
    end
  end
  [status, headers, body]
end

```

其实从rack-cors的源代码可以看出，他基本上就是在遵循w3c的规范流程进行实现。他就是一个middleware，在`ActionDispatch::Static`这个middleware前进行拦截处理。下面是rack-cors在rails项目中的配置代码：

``` ruby
config.middleware.insert_before "ActionDispatch::Static", "Rack::Cors" do
  allow do
    origins '*'
    resource '*', :headers => :any, :methods => [:get, :post, :options]
  end
end
```

另外，关于rack-cors的注意点：

* 针对simple request，该gem只判断origin和resource是否存在，如果resource存在，那么所有的simple request都可以处理。resource配置项后面的参数headers和methods的配置是针对preflight request的，和simple request无关。
比如即使配置`methods=>[:get]`，然后发送一个POST的simple request，也是能够正常处理的。

#### 参考链接

我的测试cors项目的github地址：
<https://github.com/MakeItEasy/CrossDomainDemo>

在学习以及写这篇文章的过程中参考了以下链接：

* <http://blog.csdn.net/net_lover/article/details/5172509>
* <http://www.w3.org/TR/cors/>
* <http://www.ibm.com/developerworks/cn/web/wa-aj-jsonp1/>
* <http://blog.csdn.net/hfahe/article/details/7730944>


#### 感悟

* 对于一些规范，有时候直接查看w3c的规范文档，会更加深入的了解所以然。（虽然英文看起来很崩溃。。。）
* 对于一些gem，可以适当看一下他们的源码实现，这样可以大概了解他们的处理流程，但是也不必过于深究一些源码细节，除非有足够的精力或者时间。

<br>
<br>
The End






