# 关于ts项目中如何合理的为后端接口定义参数类型

    在传统业务的ts项目中，通常会调用很多后端的api，而几乎每个api的入参和出参都不一样，通常的做法是人为的为每个api定义对呀的参数类型，不仅工作量大，而且重复率高

市面上一些常见的方案是这样的：
1. 为api入参或者出参定义为any类型
2. 简单粗暴的为每个api定义为ts类型
3. 使用工具去提前调用接口，然后根据接口返回参数生成对呀的ts类型
4. 后端使用node+ts开发，或者增加node+ts的中间层，这在微服务架构中叫BFF层，其作用就是聚合基础api字段，返回业务数据，因为在后端使用ts提供接口的时候，必然需要主动去声明api的入参和出参类型，因此就可以通过代码共享的方式和后端共用ts数据类型

其中1、2点不过多解释；

第3点应该有对应实现工具，但是缺点很明显
* 一是接口返回的类型可能不全，而且调用接口传递模拟参数比较麻烦
* 二是类型的重复率还是很高，框架在做类型检查的时候还是会比较消耗性能
* 三是工具生成的类型代码在源码中不具备任何意义

第4点是比较好的做法，但是也有一些缺点
* 一是通常在规模比较大的项目中实现效果更好，因为比较规范的微服务架构以及DDD架构思维事很难落地的，其中包括领域的合理划分，以及服务职责权限等，更别提BFF在项目中的实现
* 二是即使在后端架构中存在BFF层，也不能保证BFF层会涉及到ts代码


## 将数据库字段类型定义在ts类型中

    一些思考：前端项目中的三架马车css、html、js，前两者暂不谈，js开发的项目和其它所有开发语言开发的项目一样，都是由变量、常量、关键字这三部分组成，常量和关键字都是不可变的，剩下的变量都具有继承或者类继承的特性，总能在变量上找到它的原型或者构造它的函数，像js中所有对象最终的原型都指向null一样，似乎在业务代码中也应该使用这样理想的开发模式；

想要定义合理的出入参类型的前提是先合理划分后端微服务子域类型，只有在保证后端逻辑清晰的情况下才能更好的在前端定义对应的参数类型，以普通商城项目为例，后端可能划分有商品、物料、用户、订单等子域，对应的前端也应该划分相应的数据类型，下面为简单例子

~~~typescript
/* user.ts */
export type Id = number
export type Name = string
export type Age = number
export interface User {
  id: Id //用户id
  name: Name
  age: Age
}

~~~

~~~typescript
/* product.ts */
export type Id = string
export type ImgUrl = string
export type Price = number
export type Title = string
export interface Product {
  id: Id//商品id
  imgUrl: ImgUrl//商品图片
  price: Price//价格
  title: Title//商品标题
}

~~~

~~~typescript
/* order.ts */
import { Id as UserId } from "./user.ts"
import { Id as ProductId } from "./product.ts"

export type Id = number
export type CreateTime = string
export type Payment = number
export interface Order {
  id: Id//订单id
  createTime: CreateTime//创建时间
  payment: Payment//付款金额
  userId: UserId//用户id
  productId: ProductId//商品id
}

~~~

api `/getOrderById` 通过orderId来获取用来获取订单数据

~~~ts
import { Id, Order } from "./order.ts"
//入参类型
type E = Id
//出参类型
type P = Order
~~~

如果返回的类型是订单信息和商品信息的集合，则只用将Product和Order类型合并即可

~~~ts
//如果返回的类型为下面这种
const data = {
  id: 1,//订单id
  createTime: "2022-10-22",//创建时间
  payment: 33,//付款金额
  userId: 111,//用户id
  productId: "qwertyuiop",//商品id
  imgUrl: "",//商品图片
  title: ""//商品标题
}

//参数类型也可以是先排除product id 以后和order类型交叉合并
type T = Omit<Product, 'id' | 'price'> & Order
~~~

