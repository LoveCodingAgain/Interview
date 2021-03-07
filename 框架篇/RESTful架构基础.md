## 1.1基础概念
> REST（Representational State Transfer）架构⻛格是⼀种世界观，把信息提升为架构中的⼀等公⺠。 通过 REST 可以实现系 统的⾼性能、可伸缩、通⽤性、简单性、可修改性和可扩展等特性。
> REST 表⽰什么含义？以⽆状态⽅式传输、访问和操作⽂本数据。当正确部署后，REST 为互联⽹上不同应⽤程序之间提供了⼀致 的互操作性。⽆状态（stateless）这个术语⾄关重要，它使得应⽤程序可以⽤不可知的⽅式进⾏通信。RESTful API 通过统⼀资 源定位符地址（URL）公开服务。URL 名称将资源的区分为接受内容或返回内容。
> URI：表示资源，资源一般对应服务器端领域模型中的实体类.
> URI表示资源的两种方式：资源集合、单个资源.
> - /zoos  //所有动物园
> - /zoos/1/animals  //id为1的动物园中的所有动物
> - /zoos/1  //id为1的动物园
> - /zoos/1;2;3  //id为1，2，3的动物园

> GET:查询单个资源或者资源集合
> - GET /zoos  //查询所有动物园
> - GET /zoos/1  //查询id为1的动物园
> - GET /zoos/1/employees  //查询id=1的动物园的所有雇员

> POST:创建单个资源。POST一般向"资源集合"型uri发起
> - POST /animals  //新增动物
> - POST /zoos/1/employees //为id为1的动物园雇佣员工

> PUT:更新单个资源（全量）,客户端提供完整的更新后的资源。与之对应的是 PATCH，PATCH 负责部分更新，客户端提供要更新的那些字段。PUT/PATCH一般向“单个资源”型uri发起.
> - PUT /animals/1 // 更新id为1的动物
> - PUT /zoos/1 //更新id为1的动物园

> DELETE:删除资源
> - DELETE /zoos/1/employees/2 //删除id为1的动物园的id为2号的管理员
> - DELETE /zoos/1/employees/2;4;5 //删除id为1的动物园的id为2,4,5号的管理员
> - DELETE /zoos/1/animals  //删除id为1的动物园内的所有动物

> 安全性和幂等性
> - 安全性：不会改变资源状态，可以理解为只读的。
> - 幂等性：幂等性：执行1次和执行N次，对资源状态改变的效果是等价的。GET和PUT是幂等的.GET是安全的.
 
