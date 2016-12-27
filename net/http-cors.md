###1、CORS概述
---
跨源资源共享标准通过新增一系列 HTTP 头，让服务器能声明那些来源可以通过浏览器访问该服务器上的各类资源（包括CSS、图片、JavaScript 脚本以及其它类资源）。另外，对那些会对服务器数据造成破坏性影响的 HTTP 请求方法（特别是 GET 以外的 HTTP 方法，或者搭配某些MIME类型的POST请求），标准强烈要求浏览器必须先以 OPTIONS 请求方式发送一个预请求(preflight request)，从而获知服务器端对跨源请求所支持 HTTP 方法。在确认服务器允许该跨源请求的情况下，以实际的 HTTP 请求方法发送那个真正的请求。服务器端也可以通知客户端，是不是需要随同请求一起发送信用信息（包括 Cookies 和 HTTP 认证相关数据）。

###1、CORS原理
例如：域名A(http://a.example)的某 Web 应用程序中通过<img>标签引入了域名B(http://b.foo)站点的某图片资源(http://b.foo/image.jpg)。这就是一个跨域请求，请求http报头包含Origin: http://a.example，如果返回的http报头包含响应头 Access-Control-Allow-Origin: http://a.example （或者Access-Control-Allow-Origin: http://a.example），表示域名B接受域名B下的请求，那么这个图片就运行被加载。否则表示拒绝接受请求。