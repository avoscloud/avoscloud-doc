<!--# 做个餐厅管理系统没用 LeanStorage，这位程序员被老板开除了-->

# LeanStorage 案例分析 - 餐厅管理系统设计

## 这篇教程讲什么？
- 详解如何设计一个餐厅-座位-预订管理系统在 LeanCloud 如何快速的开发
- 在没有 mongodb 相关知识储备的情况下如何使用传统的 SQL 语句迅速地使用 LeanCloud 存储
- 在表设计的时候如何合理的使用 Relation，Poninter 和自建中间表

## 读完这篇教程之后能学到什么？
如下内容对个人情况不同可能有所偏差，但是大致如此。

假设每一项技能以 10 星计算，10 星可以称为该技能的业界顶尖水平

- mongodb 使用水平提升：1 星
- 逻辑建模能力提升：2 星
- 编程水平提升：2 星
- LeanCloud 知识储备增加：4 星


## 项目背景介绍
本文缘起一个用户在社区的讨论：

我想设计一个表结构，能实现如下需求：

- 展现餐厅列表
- 每个餐厅需要管理餐厅内部的座位
- 每个座位在每天的某一个时段可能会被预定，我需要管理这些预定
- 给老板展现每一个餐厅某一个时段的预订率
- 给老板展现某一个餐厅的某一个座位的翻桌率（这个位置可能靠窗户也可能座位本身的设计很受欢迎）


一开始我们只是讨论，后来我们跟这个客户也多次沟通最后在不断的探讨下我们发现其中很多设计的思路针对其他系统是适用的，因此我们特别提炼出一些精华分享给所有 LeanCloud 开发者，希望减少大家「掉坑」情况的发生，首先我们来针对上述需求一个一个拆解，逐步实现这个系统里面的表结构设计。


## 展现餐厅列表(Restaurant)

首先我们简化餐厅的字段，我们先保存一下基本字段（后文会根据需求补充或者修改字段）：

id|name(餐厅名称:string)
--|--
r1|必胜客
r2|麦当劳
r3|肯德基


然后调用 SDK 的代码创建它们:

```js
'use strict';
var AV = require('leancloud-storage');

exports.newRestaurant = function newRestaurant(restaurantData) {
    let name = restaurantData.name || '';
    if (name == '') {
        throw new Error('餐厅必须得有个名字吧，😜');
    }
    let restaurant = new AV.Object('Restaurant');
    restaurant.set('name', name);
    return restaurant.save();
}

// 调用代码如下:
let restaurantData = {
    name: '必胜客',
};
newRestaurant(restaurantData).then(result => {
    console.log(result.id);
});
```

## 座位(Seat)

id|under(归属餐厅:Pointer)|capacity(座位容量:number)
--|--|--
s1|r1|2
s2|r1|3
s3|r1|4
s4|r2|2
s5|r2|2
s6|r3|5

```js
//filename:booking.js
exports.newSeat = function newSeat(seatData) {
    let restaurant = seatData.restaurant;
    if (restaurant == undefined) throw new Error('一个座位必须属于一个餐厅啊，亲 🏚');
    let capacity = seatData.capacity || 1;
    let seat = new AV.Object('Seat');

    seat.set('under', restaurant);
    seat.set('capacity', capacity);

    return seat.save();
}

// 调用代码如下：
let restaurantData = {
    name: utils.randomString(),
};
newRestaurant(restaurantData).then(restaurant => {
    let seatData = {
        capacity: 5,
        restaurant: restaurant
    };
    return newSeat(seatData);
}).then(result => {
    // 保存成功
}).catch(error => {
    console.log(error);
});
```

## 预订表(Bookings)

objectId|seat(座位:)|from(预订起始时间:Date)|to(预订结束时间:Date)
--|--|--|--
s1|r1|2017-02-01 18:00|2017-02-01 19:00
s2|r1|2017-02-02 15:00|2017-02-02 16:00
s3|r1|2017-02-02 15:00|2017-02-01 16:00
s4|r2|2017-02-01 18:00|2017-02-01 19:00
s5|r2|2017-02-01 18:00|2017-02-01 19:00
s6|r3|2017-02-01 18:00|2017-02-01 19:00


```js
//filename:booking.js
exports.newBooking = function newBooking(bookingtData) {
    let seat = bookingtData.seat;
    if(typeof seat === 'undefined') throw new Error('订座位的时候一定要指定座位...');
    let from = bookingtData.from;
    if(typeof from === 'undefined') throw new Error('订座位的时候一定要指定指定预订起始时间'); 
    let to = bookingtData.to;
    if(typeof to === 'undefined') throw new Error('订座位的时候一定要指定指定结束就餐的时间');

    let booking = new AV.Object('Booking');
    booking.set('seat',seat);
    booking.set('from',from);
    booking.set('to',to);

    return booking.save();
}
```

## 查询某一个餐厅的某一个时段的预订情况

首先我们针对上述的需求拆解查询条件：

1. 查询某一个餐厅里面里面所有的座位
2. 查询一个座位在某一个时段的预订情况
3. 查询一些座位在某一个时段的预订情况

拆解之后，我们来用代码逐步实现。

### 查询某一个餐厅里面里面所有的座位

假设数据库存在如下数据：

objectId|name|createdAt|updatedAt
--|--|--|--
58edc5cea0bb9f006a4d57eb|必胜客|2017-04-12 14:14:38|2017-04-17 15:36:35
58ee366644d9040057784276|麦当劳|2017-04-12 22:15:02|2017-04-13 10:51:38
58ee0124a0bb9f006a4f3f40|肯德基|2017-04-12 18:27:48|2017-04-13 10:51:35







