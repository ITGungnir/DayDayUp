## HTTP请求方法

参考：https://zhuanlan.zhihu.com/p/25441947

* GET（Retrofit）：get方法请求指定的页面信息，返回实体主体。该请求是向服务器请求信息，请求参数会跟在url后面，因此，对传参长度有限制的，而且不同浏览器的上限是不同的（2k, 7~8k及其他）。由于get请求直接将参数暴露在url中，因此对于一些带有重要信息的请求可能并不完全合适。
* POST（Retrofit）：post请求是向指定资源提交数据进行处理请求，例如提交表单或者上传文件等。数据被包含在请求体中，POST请求可能会导致新的资源的建立和/或已有资源的修改。post方法没有对传递资源的大小进行限制，往往是取决于服务器端的接受能力，而且，该方法传参安全性稍高些。
* PUT（Retrofit）：PUT方法是从客户端向服务器传送的数据取代指定的文档的内容。PUT方法的本质是idempotent的方法，通过服务是否是idempotent来判断用PUT 还是 POST更合理，通常情况下这两种方法并没有刻意区分，根据语义使用即可。
* HEAD（Retrofit）：
* PATCH（Retrofit）：
* DELETE（Retrofit）：
* OPTIONS（Retrofit）：
* CONNECT：
* TRACE：
