## RESTful简介

* RESTful：REpresentational State Transfer，表现层状态转移

* 简单来说：
  * 网络上的每条数据、每个文件等都是资源（Resource）
  * 用URL定位资源，用HTTP动词（GET/POST/DELETE/PATCH）描述操作
  * 看URL就知道要什么；看http method就知道要干什么；看http status code就知道结果如何
  * 对同一个对象的增删改查，使用同一个URL和不同的动词

* 不推荐的接口设计：
  * GET: xxxxxxxxx/getProducts
  * GET: yyyyyyyyy/deleteProduct?id=6
* 推荐的接口设计：
  * GET: xxxxxxxxx/products
  * DELETE: yyyyyyyyy/products