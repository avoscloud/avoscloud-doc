# 应用内社交模块
应用内社交，又称「事件流」，在应用开发中出现的场景非常多，包括用户间关注（好友）、朋友圈（时间线）、状态、互动（点赞）、私信等常用功能。


## 基本概念

### Status
是指一条广义上的状态,不只可以表示某个用户更新了状态, 还可以表示任意的一个事件,比如某人发布了一篇文章, 某个图片被赞等.

### Follower/Followee
分别表示用户的粉丝和用户的关注，在控制台中对应着两张表 `_Follower` 和 `_Followee`。

### 特别提示
发了一条状态，并不代表会自动发送了一条 push 消息（也就是推送通知），开发者可以自由控制是否使用 push。关于消息推送请参考[消息推送开发指南](./push_guide.html)

## JavaScript SDK 的使用方法

### 用户关系

AV.User 新增两个方法 `follow` 和 `unfollow` 来建立用户关系，你可以关注某个用户：

```javascript
AV.User.current().follow('52f9be45e4b035debf88b6e2').then(function(){
  //关注成功
}, function(err){
  //关注失败
  console.dir(err);
});
```

follow方法接收一个AV.User对象或者User对象的objectId（通过user.id拿到）。

取消关注使用unfollow方法：

```javascript
AV.User.current().unfollow('52f9be45e4b035debf88b6e2').then(function(){
  //取消关注成功
}, function(err){
  //取消关注失败
  console.dir(err);
});
```

关注后，可以查询自己关注的用户列表，使用`AV.User#followeeQuery`得到一个`AV.Query`对象来查询关注的用户列表：

```javascript
var query = AV.User.current().followeeQuery();
query.include('followee');
query.find().then(function(followees){
  //关注的用户列表followees
});
```

followee是一个Pointer类型，通过include将它的所有信息查询包括进来，否则只会返回用户的id。当查询计数的时候（使用AV.Query#count方法）不建议include。

查询自己的粉丝（他人关注了我，他人就是我的粉丝），可以通过`followerQuery`方法：

```javascript
var query = AV.User.current().followerQuery();
query.include('follower');
query.find().then(function(followers){
  //粉丝列表followers
});
```

followerQuery和followerQuery方法返回的AV.Query对象可以像普通的[AV.Query](https://leancloud.cn/docs/api/javascript/symbols/AV.Query.html)对象那样使用，他们本质上都是查询数据管理平台中的`_Follower`和`_Followee`表，你可以添加order,skip,limit以及其他where条件等信息。


### 状态

关注了用户之后（也就是成为他的粉丝），你就会接收到该用户发送给他的粉丝的状态信息。例如，我喜欢了某个视频，我可以发送这条信息“我喜欢了视频xxxx”给我的粉丝。我的粉丝就可以在他们的收件箱(inbox)里收到这条状态信息。

#### 发布状态

当前登录用户发送一条状态给关注他的粉丝：
的粉丝：

```javascript
var status = new AV.Status('视频url', '我喜欢了视频xxxx.');
status.set('sound', 'sound.wmv');
AV.Status.sendStatusToFollowers(status).then(function(status){
  //发布状态成功，返回状态信息
  console.dir(status);
}, function(err){
  //发布失败
  console.dir(err);
});
```

发布状态成功，粉丝的收件箱不一定马上能看到，因为发布过程是一个异步的过程，会有一定的延迟，通常这个延迟都不大（几秒之内）。发送状态必须处于用户登录状态，使用`AV.Status.sendStatusToFollowers`将发送给登录用户的粉丝。

AV.Status对象包含下列属性：

* id 对象的objectId
* messageId 状态在某个用户收件箱的消息编号，发件箱返回的status则没有此属性，messageId可用于收件箱的分页查询，见下文。
* inboxType 收件箱类型，默认是timeline收件箱，也就是default。SDK默认提供private表示私信，default表示timeline，您也可以自定义。
* data  状态属性，一个JSON对象，可通过get和set方法获取和设置
* createdAt 消息的创建时间

其中data.source属性是保留属性，默认指向状态的发布者。


#### 发送私信给某个用户

我还可以发送一条私信给单独某个用户：

```javascript
var status = new AV.Status(null, '秘密消息');
AV.Status.sendPrivateStatus(status,'52f9be45e4b035debf88b6e2').
  then(function(status){
    //发送成功
    console.dir(status);
  }, function(err){
    //发布失败
    console.dir(err);
});
```

`AV.Status.sendPrivateStatus`的第二个参数指定私信接收的用户或者用户的objectId。

#### 发送给自定义收件箱

通过send方法还可以自定义inboxType:

```javascript
var query = ... //一个AV.Query对象，定义接收者。
var status = new AV.Status(null, '我读了《clojure编程乐趣》');
//定义一个book收件箱
status.inboxType = 'book';
status.query = query;
status.send().then(function(status){
  //发送成功
});
```

#### 查询收件箱

查询我的timeline收件箱，可以通过`AV.Status.inboxQuery`方法：

```javascript
var query = AV.Status.inboxQuery(AV.User.current());
query.find().then(function(statuses){
  //查询成功，返回状态列表，每个对象都是AV.Status
}, function(err){
  //查询失败
  console.dir(err);
});
```

收件箱返回的status列表，每个status都有messageId属性，表示这条status在这个收件箱里的唯一编号，`AV.InboxQuery`有两个方法用于限制messageId的范围：`sinceId`和`maxId`。其中:

* sinceId设定查询返回的status的messageId必须**大于**传入的message id，
* maxId限定查询返回的status的messageId必须小于等于传入的message id。

通过sinceId和maxId可以实现分页查询：

* 查询本次查询之后新增的status（向后翻页刷新）:

```javascript
//假设messageId是上次查询返回的status的最大messageId编号
var messageId = ...
var query = AV.Status.inboxQuery(AV.User.current());
query.sinceId(messageId);
//查询从上次查询返回结果之后的更新status
query.find().then(function(statuses){
  //查询成功，返回状态列表，每个对象都是AV.Status
}, function(err){
  //查询失败
  console.dir(err);
});
```

* 查询本次查询的前一页（也就是更老的status，向前翻页）:

```javascript
//假设messageId是上次查询返回的status的最大messageId编号
var messageId = ...
var query = AV.Status.inboxQuery(AV.User.current());
query.maxId(messageId);
```


`AV.Status.inboxQuery`还可以指定收件箱的类型，默认是查询timeline收件箱，也可以查询私信收件箱：

```javascript
var query = AV.Status.inboxQuery(AV.User.current(), 'private');
```

#### 查询收件箱未读状态数目

使用`AV.Status.countUnreadStatuses`可以查询某个收件箱的未读状态数目和总数目：

```javascript
AV.Status.countUnreadStatuses(AV.User.current()).then(function(result){
  console.dir(result);
  var total = result.total;
  var unread  = result.unread;
}, function(err){
    //查询失败
});
```

同样的，您可以这样查询当前登录用户未读私信的未读数目和总数目：

```javascript
AV.Status.countUnreadStatuses(AV.User.current(),'private').then(function(result){
  console.dir(result);
  var total = result.total;
  var unread  = result.unread;
}, function(err){
    //查询失败
});
```

#### 查询发件箱

查询我发出去的状态信息，可以通过`statusQuery`方法：

```javascript
var query = AV.Status.statusQuery(AV.User.current());
query.find().then(function(statuses){
  //查询成功，返回状态列表，每个对象都是AV.Object
}, function(err){
  //查询失败
  console.dir(err);
});
```

`AV.Status.statusQuery`返回的是一个普通的AV.Query对象，本质上是查询数据管理平台的`_Status`表。您可以自主添加更多查询条件。



## iOS SDK中的使用方法


### 好友关系

#### 关注/取消关注

当前登陆用户可以关注某人：

    NSString *userObjectId  =@"XXXXXX";

    //关注
    [[AVUser currentUser] follow:userObjectId andCallback:^(BOOL succeeded, NSError *error) {
        if (error.code==kAVErrorDuplicateValue) {
            //重复关注
        }

    }];

    //取消关注
    [[AVUser currentUser] unfollow:userObjectId andCallback:^(BOOL succeeded, NSError *error) {

    }];

如果您在应用设置的应用选项里勾选了`自动互相关注（事件流）`，那么在当前用户关注某个人，那个人也会自动关注当前用户。

从 2.6.7 版本开始，我们允许在 follow 的时候同时传入一个 attribute 列表，用于设置关系的属性，这些属性都将在 `_Follower` 和 `_Followee` 表同时存在:

```objc
   NSDictionary * attrs = ……
   [[AVUser currentUser] follow:userObjectId userDictionary:attrs andCallback:^(BOOL succeeded, NSError *error) {
	    //处理结果
    }];

```

#### 获取粉丝/关注列表

有2个特殊的`AVQuery`:

```objc
//粉丝列表查询
AVQuery *query= [AVUser followerQuery:@"USER_OBJECT_ID"];

//关注列表查询
AVQuery *query= [AVUser followeeQuery:@"USER_OBJECT_ID"];
```

`followerQuery` 和 `followeeQuery` 返回的 AVQuery 可以增加其他查询条件，只要在`_Followee`和`_Follower` 表里存在的属性都可以作为查询或者排序条件。

**注：默认的查询得到的AVUser对象仅仅有ObjectId数据，如果需要整个AVUser对象所有属性，则需要调用include方法**。例如

```objc
AVQuery *query= [AVUser followeeQuery:@"USER_OBJECT_ID"];
[query includeKey:@"followee"];
```

是分别获得某个用户的粉丝和关注, 我们也可以同时取得这这两种:

```objc
[[AVUser currentUser] getFollowersAndFollowees:^(NSDictionary *dict, NSError *error) {
    NSArray *followers=dict[@"followers"];
    NSArray *followees=dict[@"followees"];
}];
```

### 状态

#### 发布状态

发布一条时间线状态, 即发一条我的粉丝可以看到的状态

```objc
AVStatus *status=[[AVStatus alloc] init];

status.data=@{@"text":@"data type change"};

[AVUser logInWithUsername:@"travis" password:@"123456"];
[AVStatus sendStatusToFollowers:status andCallback:^(BOOL succeeded, NSError *error) {
    NSLog(@"============ Send %@", [status debugDescription]);
}];
```

其中`status.data`可以任意指定NSDictionary数据. **注意: 这个字典中的source字段不可用，内部保留**


#### 发私信

给某个用户发私信也非常简单

```objc
AVStatus *status=[[AVStatus alloc] init];
status.data=@{@"text":@"this is a private message"};

NSString *userObjectId=@"XXXXXXXXXXXXX";

[AVStatus sendPrivateStatus:status toUserWithID:userObjectId andCallback:^(BOOL succeeded, NSError *error) {
    NSLog(@"============ Send %@", [status debugDescription]);
}];
```

#### 自定义状态

除了上面常见两种场景, 自定义状态可以通过设置**受众群体和发送者** 来实现更加灵活的功能

```objc
AVStatus *status=[[AVStatus alloc] init];
[status setData:@{@"text":@"we have new website, take a look!",@"link":@"http://leancloud.cn"}];

status.type=@"system";

AVQuery *query=[AVUser query];
[query whereKey:@"age" equalTo:@(20)];
[status setQuery:query];

[status sendInBackgroundWithBlock:^(BOOL succeeded, NSError *error) {

}];
```

上面是系统广播的基本实现. 因为指定了一个AVUser查询,所以会发送给所有`age=20`的用户, 指定了`type`是`system`或者任意字符串, 则所有用户会在查询这个类型的状态中看到这一条.

#### 获取状态

下面代码会获取用户时间线上的50条状态

```objc
AVStatusQuery *query=[AVStatus inboxQuery:kAVStatusTypeTimeline];

//限制50条
query.limit=50;

//限制1397这个messageId上次查询的最大messageId, 如果不设置,默认为最新的
query.maxId=1397;

//需要同时附带发送者的数据
[query includeKey:@"source"];

[query findObjectsInBackgroundWithBlock:^(NSArray *objects, NSError *error) {
    //获得AVStatus数组
}];
```

同理, 可以获得用户的私信,只要把参数改为`kAVStatusTypePrivateMessage`。返回的`AVStatus`对象有一个`messageId`属性，用于唯一表示这条Status在这个inbox里的标示符。可以用这个id结合query做分页查询。

`AVStatusQuery`可以设置sinceId和maxId:

* sinceId  设定查询返回的status的messageId必须大于传入的message id，
* maxId  限定查询返回的status的messageId必须小于等于传入的message id。

使用这两个id就可以做分页查询。**`AVStatusQuery`查询不支持skip**

获取收件箱的未读status数目（从上次访问收件箱最新status到现在的未读status数目）可以通过`[AVStatus getUnreadStatusesCountWithType:andCallback]`方法。

下面的代码是某个用户发送出去的状态,**请注意,查询发送出去的状态,是无法用messageId(sinceId,maxId)来做分片查询的,因为messageId只是相对于某个用户的Inbox才有意义, 同时返回的状态中也没有messageId的数据**

```objc
AVStatusQuery *query=[AVStatus statusQuery];

//设置查询某个用户, 默认是查询当前用户
[query whereKey:@"source" equalTo:<AVUser>];

//限制条数
query.limit=20;

//设置消息类型
query.inboxType=kAVStatusTypeTimeline;

[query findObjectsInBackgroundWithBlock:^(NSArray *objects, NSError *error) {
    //获得AVStatus数组
}];
```

## Android SDK中的使用方法

 Android 的应用内社交模块已经正式发布，欢迎尝试！

### 好友关系

#### 关注和取消关注

登陆的用户可以关注其他用户，成为他们的粉丝，例如：

```java
//关注
AVUser.getCurrentUser().followInBackground(userObjectId, new FollowCallback() {
        @Override
        public void done(AVObject object, AVException e) {
            if (e == null) {
                Log.i(TAG, "follow succeed.");
            } else if (e.getCode() == AVException.DUPLICATE_VALUE) {
                Log.w(TAG, "Already followed.");
            }
        }
    });

//取消关注
AVUser.getCurrentUser().unfollowInBackground("the user object id", new FollowCallback() {
        @Override
        public void done(AVObject object, AVException e) {
            if (e == null) {
                Log.i(TAG, "unfollow succeed.");
            } else {
                Log.w(TAG, "unfollow failed.");
            }
        }
    });
```

如果您在应用设置的应用选项里勾选了`自动互相关注（事件流）`，那么在当前用户关注某个人，那个人也会自动关注当前用户。


从 2.6.7 版本开始，我们允许在 follow 的时候同时传入一个 attribute 列表，用于设置关系的属性，这些属性都将在 `_Follower` 和 `_Followee` 表同时存在:

```java
Map<String, Object> attributes = ......
AVUser.getCurrentUser().followInBackground("target user objectId", attributes, new FollowCallback{
            @Override
            public void done(AVObject object, AVException e) {
                  //处理结果
            }
});
```

#### 获取粉丝和关注列表

您可以使用followerQuery/followeeQuery来查询您的粉丝/关注列表，这样可以设置更多的查询条件，比如

```java
// 其中userA是AVUser对象，您也可以使用AVUser的子类化对象进行查询
//vhaxun粉丝
AVQuery<AVUser> followerQuery = userA.followerQuery(AVUser.class);
//AVQuery<AVUser> followerQuery = AVUser.followerQuery(userA.getObjectId(),AVUser.class); 也可以使用这个静态方法来获取非登陆用户的好友关系
followerQuery.findInBackground(new FindCallback<AVUser>() {
    @Override
    public void done(List<AVUser> parseObjects, AVException parseException) {
        // parseObjects包含了userA的粉丝列表
    }
});

//查询关注者
AVQuery<AVUser> followeeQuery = AVUser.followeeQuery(userB.getObjectId(), AVUser.class);
//AVQuery<AVUser> followeeQuery = userB.followeeQuery(AVUser.class);
followeeQuery.findInBackground(new FindCallback<AVUser>() {
    @Override
    public void done(List<AVUser> parseObjects, AVException parseException) {
        //parseObjects就是用户的关注用户列表

    }
});
```

通过AVQuery，您也可以增加skip或者limit操作来分页查询，比如

```java
    AVQuery<AVUser> followerSkipQuery = AVUser.followerQuery(userA.getObjectId(), AVUser.class);
    followerSkipQuery.setLimit(50);
    followerSkipQuery.skip(100);
    followerSkipQuery.findInBackground(new FindCallback<AVUser>() {
        @Override
        public void done(List<AVUser> parseObjects, AVException parseException) {
            // parseObjects.size() == 1
        }
    });
}
```

您也可以查找某个特定的粉丝，比如

```java
AVQuery<AVUser> followerNameQuery = userA.followerQuery(userA.getObjectId(), AVUser.class);
followerNameQuery.whereEqualTo("follower", userC);
followerNameQuery.findInBackground(new FindCallback<AVUser>() {
    @Override
    public void done(List<AVUser> parseObjects, AVException parseException) {
        // parseObjects中应当只包含userC
    }
});
```

总之 `followerQuery` 和 `followeeQuery` 返回的 AVQuery 可以增加其他查询条件，只要在`_Followee`和`_Follower` 表里存在的属性都可以作为查询或者排序条件。


**注：默认的得到的AVUser对象仅仅有ObjectId数据，如果需要整个AVUser对象所有属性，则需要调用include方法**。例如

```java
AVQuery<AVUser> followerNameQuery = AVUser.followerQuery(userA.getObjectId(), AVUser.class);
followerNameQuery.include("follower");

AVQuery<AVUser> followeeNameQuery = AVUser.followeeQuery(userA.getObjectId(), AVUser.class);
followerNameQuery.include("followee");
```

##### 一次性获取粉丝和关注列表

很多用户反映他们需要能够一次性获取性获取粉丝和关注列表的API接口，于是我们加入这个功能。

```java
    AVFriendshipQuery query = AVUser.friendshipQuery(userId, SubUser.class);
    query.include("followee");
    query.include("follower");
    query.getInBackground(new AVFriendshipCallback() {
      @Override
      public void done(AVFriendship friendship, AVException e) {
        List<SubUser> followers = friendship.getFollowers();//获取关注列表
        List<SubUser> followees = friendship.getFollowees();//获取粉丝
	AVUser user = friendship.getUser();//获取用户对象本身
      }
    });
```

### 状态

#### 发布状态

发布一条时间线状态, 即发一条我的粉丝可以看到的状态

```java
AVStatus status= new AVStatus();
// 或者您也可以使用静态方法
// AVStatus status = AVStatus.createStatus("my image", "my message");
status.setImageUrl("myImageUrl");
status.setMessage("myMessage");
// 或者您也可以使用方法
// setData(Map<String, Object> data)

AVUser.logIn("myUserName", "myPassword");
AVStatus.sendStatusToFollowersInBackgroud(status, new SaveCallback() {
    @Override
    public void done(AVException parseException) {
        Log.i(TAG, "Send status finished.");
    }
});
```

其中`status.setData`可以任意指定Map<String, Object>数据. **注意: 这个map中的source字段不可用**


#### 发私信

给某个用户发私信也非常简单


```java
AVStatus status = AVStatus.createStatus("test image", "test message");
AVStatus.sendPrivateStatusInBackgroud(status, "user object id", new SaveCallback() {
    @Override
    public void done(AVException parseException) {
        Log.i(TAG, "Send private status finished.");
    }
});
```

#### 自定义Status

除了上面常见两种场景, 自定义Status可以通过设置**受众群体和发送者**来实现更加灵活的功能

```java
Map<String, Object> data = new HashMap<String, Object>();
data.put("text", "we have new website, take a look!");
data.put("link", "http://leancloud.cn");
AVStatus status = AVStatus.createStatusWithData(data);
status.setInboxType("system");

    status.sendInBackgroundWithBlock(new SaveCallback() {
        @Override
        public void done(AVException e) {
            Log.i(TAG, "Send finished");

        }
    });
```

上面是系统广播的基本实现. 因为指定了一个无条件的AVUser查询,所以会发送给所有的用户, 指定了`inboxType`是`system`或者任意字符串, 则所有用户会在查询这个类型的状态中看到这一条.我们在SDK中间为您准备两种预定的类型:`AVStatus.INBOX_TYPE.TIMELINE`和`AVStatus.INBOX_TYPE.PRIVATE`。

#### 获取收件箱状态

下面代码会获取用户收件箱内时间线上的50条状态

```java
AVStatusQuery<AVStatus> inboxQuery = AVStatus.inboxQuery(AVStatus.class, userB,AVStatus.INBOX_TYPE.TIMELINE.toString());
inboxQuery.setLimit(50);  //设置最多返回50条状态
inboxQuery.setSinceId(0);  //查询返回的status的messageId必须大于sinceId，默认为0
inboxQuery.findInBackground(new InboxStatusFindCallback(){
  @Override
  public void done(final List<AVStatus> parseObjects, final AVException parseException) {

  }
});
```

同理, 可以获得用户收件箱的私信,只要把type参数改为`AVStatus.INBOX_TYPE.PRIVATE.toString()`即可

`AVStatus`有一个`messageId`属性，用于标示这条status在inbox里的唯一位置。使用这个`messageId`结合`AVStatusQuery`可以做分页查询，AVStatusQuery可以设置`sinceId`和`maxId`：

* sinceId  设定查询返回的status的messageId必须大于传入的message id，
* maxId  限定查询返回的status的messageId必须小于等于传入的message id。

使用这两个id就可以做分页查询。**`AVStatusQuery`查询不支持skip**。

从 v2.6.7 版本开始，`InboxStatusFindCallback` 提供一个方法叫做isEnd(),用来**在查询后**检查收件箱查询是否已经到了最早的一页数据。

### 获取收件箱的计数

使用`AVStatus.getUnreadStatusesCountInBackground`方法可以查询收件箱的未读status数目和总status数目：

```java
AVStatus.getUnreadStatusesCountInBackground(AVStatus.INBOX_TYPE.TIMELINE.toString(), new CountCallback() {
        public void done(int count, AVException e) {
            if (e == null) {
                //count就是未读status数目
            } else {
                //有错误发生。
            }
        }
    });
```

#### 查询发件箱状态

下面的代码能够查询当前用户发件箱内已经发送的50条状态

```java
AVStatusQuery<AVStatus> query = AVStatus.statusQuery(AVStatus.class,AVUser.getCurrentUser());
query.setLimit(50);    //设置最多返回50条状态
query.setSinceId(0);   //查询返回的status的messageId必须大于sinceId，默认为0
//query.setInboxType(AVStatus.INBOX_TYPE.TIMELINE.toString()); 此处可以通过这个方法来添加查询的状态条件，当然这里你也可以用你自己定义的状态类型，因为这里接受的其实是一个字符串类型。
query.findInBackground(new FindCallback<AVStatus>(){
  @Override
  public void done(final List<AVStatus> parseObjects,final AVException parseException) {

  }
});
```

## REST API

本节介绍应用内社交的 REST API。

### 用户关系 API

使用这里的 API 来建立用户关系，你可以关注、取消关注某个用户。

* 这里的3个查询API都遵循我们的 REST API 规范，支持 `where,order,skip,limit,count,include` 等。如果没有特殊说明，返回的结果都是`{results: [数组结果]}`，跟其他查询 API 保持一致。
* 用户在`_Follower`和`_Followee`表中都存储为 pointer 类型，因此如果要查询出用户信息，应该加上include 指定字段。

#### 关注和取消关注用户 API

通过操作 `/users/:user_id/friendship/:target_id` 资源可以关注或者取消关注某个用户，其中：

* `:user_id` 表示发起关注动作的用户的 objectId，(如果设置了`X-AVOSCloud-Session-Token`头, 可以为`self`表示当前登录用户)
* `:target_id` 表示想要关注的目标用户的 objectId

例如，让当前用户 `51fa6886e4b0cc0b5a3792e9` 关注目标用户 `51e3a334e4b0b3eb44adbe1a`：

```sh
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  https://leancloud.cn/1.1/users/51fa6886e4b0cc0b5a3792e9/friendship/51e3a334e4b0b3eb44adbe1a
```

关注后，`_Follower`和`_Followee`都会多出一条记录，如果选择了自动互相关注选项，会各多出两条记录。

取消关注通过:

```sh
curl -X DELETE \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  https://leancloud.cn/1.1/users/51fa6886e4b0cc0b5a3792e9/friendship/51e3a334e4b0b3eb44adbe1a
```

关注还可以增加一些属性：

```sh
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"score": 100}' \
  https://leancloud.cn/1.1/users/51fa6886e4b0cc0b5a3792e9/friendship/51e3a334e4b0b3eb44adbe1a
```

那么`score`字段将同时出现在`_Follower`和`_Followee`表，可以作为查询或者排序条件。

#### 查询粉丝或者关注者列表 API

查询粉丝列表（也就是关注我的人），可以通过：

```sh
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  https://leancloud.cn/1.1/users/51fa6886e4b0cc0b5a3792e9/followers
```

返回的用户列表是 Pointer 类型，如果想要将用户信息也返回，需要 include:

```sh
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -G \
  --data-urlencode 'include=follower' \
  https://leancloud.cn/1.1/users/51fa6886e4b0cc0b5a3792e9/followers
```

查询关注的用户列表：

```sh
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -G \
  --data-urlencode 'include=followee' \
  https://leancloud.cn/1.1/users/51fa6886e4b0cc0b5a3792e9/followees
```

同时查询粉丝和关注的人：

```sh
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -G \
  --data-urlencode 'include=followee' \
  https://leancloud.cn/1.1/users/51fa6886e4b0cc0b5a3792e9/followersAndFollowees
```

结果返回：

```json
{followers: [粉丝列表], followees: [关注用户列表]}
```

如果指定count=1，则返回结果里加上followers_count和followees_count 表示粉丝数目和关注者数目：

```sh
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -G \
  --data-urlencode 'include=followee' \
  --data-urlencode 'count=1' \
  https://leancloud.cn/1.1/users/51fa6886e4b0cc0b5a3792e9/followersAndFollowees
```

### 状态 API

再次解释下术语定义：

* status  一条状态，包含两个预定义属性`messageId`和`inboxType`，其他属性都可自定义。
* target  Status 的目标接收者，也就是 inbox 的 owner。
* inbox   target 的收件箱，拥有 owner,inboxType 两个属性表示所有者和收件箱类型

#### 发布 Status API

调用 API 如下：

```
POST /statuses
```

接受的JSON对象参数：

* query 查询target的条件，包括下列属性：
 * where 查询条件，可为空{}，表示查询全表
 * className target的className
 * key 查询指定的列作为inbox的owner属性存储，如果不指定，默认将为整个对象作为pointer存储到owner。
* data  status的数据json对象，用户自定义。如果包含source（指向发送者的pointer），并且inboxType设定为default，这status会同时往发送者的inbox投递。
* inboxType  字符串，指定接收这条status的inbox类型，可为空，默认为`default`。

示例1，往 dennis 的粉丝群体发送一条status：

```sh
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{
         "data": {
            "image": "paas-files.qiniudn.comwQUf3WohbJpyuXutPjKHPmkSj4gbiYMeNJmTulNo.jpg",
            "message": "AVOS Cloud is great!"
         },
         "inboxType": "default",
         "query": {
             "className": "_Follower",
             "keys": "follower",
             "where": {
                 "user": {
                     "__type": "Pointer",
                     "className": "_User",
                     "objectId": "dennis'id"
                 }
              }
         }
      }' \
   https://leancloud.cn/1.1/statuses
```

这条状态的内容是 data 指定的，并且设定 inboxType 是 `default`：

```json
{
     "image": "paas-files.qiniudn.comwQUf3WohbJpyuXutPjKHPmkSj4gbiYMeNJmTulNo.jpg",
      "message": "AVOS Cloud is great!"
}
```

这条状态的目标用户群体是 query 指定的查询条件，查询的是`_Follower`表中 `dennis id`的粉丝用户。

示例2，dennis向catty发送私信的请求类似：

```json
{
    "data": {
        "source" : {
            "__type" : "Pointer",
            "className" : "_User",
            "objectId" : "dennis's id"
        },
        "message": "hello catty!"
    },
    "inboxType": "private",
    "query": {
        "className": "_User",
        "where": {
            "objectId": "catty's id"
        }
    }
}
```

#### 查询发出的status API

```
GET /statuses
```

跟查询其他对象一样。

知道objectId，查询单条status:

```
GET /statuses/:status_id
```

#### 删除status API

```
DELETE /statuses/:status_id
```

#### 查询已经关注的用户的status聚合列表 API

查询我关注的用户发出来的状态聚合而成的列表，也就是查询自己的收件箱，通过

```
GET /subscribe/statuses
```

接收参数（参数都必须经过url encode）：

* owner JSON序列化后的owner字符串，表示inbox所有者。
* inboxType  inbox的类型，默认为`default`，可为空。
* where  用于过滤status的where条件，也是JSON序列化后的字符串。
* order 排序status的字段列表，必须是status本身的字段，默认`-createdAt`。
* sinceId  查询返回的status的messageId必须大于sinceId，默认为0。
* maxId  查询返回的status的messageId必须小于等于maxId，默认为0。
* limit 最多返回多少条status，默认100，最大100。
* count 默认为空，设置为"1"表示在结果中带上status的count计数。

我们来看一些例子

* 示例1，查询我的主页 timeline:

```sh
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -G \
  --data-urlencode 'owner={"__type":"Pointer","className":"_User","objectId":"dennis id"}' \
  https://leancloud.cn/1.1/subscribe/statuses
```

* 示例2，查询我的最新私信列表

```sh
    --data-urlencode 'owner={"__type":"Pointer","className":"_User","objectId":"dennis"}' \
    --data-urlencode 'inboxType=private'
```

* 示例3，假设上次返回的最大messageId是99，查询从mesageId为99开始最新的status:

```sh
    --data-urlencode 'owner={"__type":"Pointer", "className":"_User","objectId":"dennis"}' \
    --data-urlencode 'sinceId=99'
```

* 示例4，查询messageId在99到199之间的status：

```sh
    --data-urlencode 'owner={"__type":"Pointer","className":"_User","objectId":"dennis"}' \
    --data-urlencode 'sinceId=99' \
    --data-urlencode 'maxId=199'
```

* 示例5，查询最新的status，并且status的image属性存在，也就是查询包含图片的最新status:

```sh
    --data-urlencode 'owner={"__type":"Pointer","className":"_User","objectId":"dennis"}' \
    --data-urlencode 'where={"image":{"$exists":true }}'
```

#### 查询status计数 API

* 查询inbox总消息数目和未读消息数目：

```
GET "/subscribe/statuses/count
```

可指定的条件：

 * owner JSON序列化后的owner字符串，表示inbox所有者。
 * inboxType inbox类型，默认为`default`，可为空

示例1,查询我的未读消息数目

```sh
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -G \
  --data-urlencode 'owner={"__type":"Pointer","className":"_User","objectId":"dennis"}' \
  https://leancloud.cn/1.1/subscribe/statuses/count
```

返回：

```json
{ "total": 100, "unread":20}
```

示例2，查询私信消息数目：

```sh
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -G \
   --data-urlencode 'owner={"__type":"Pointer","className":"_User","objectId":"dennis"}' \
   --data-urlencode 'inboxType=private' \
   https://leancloud.cn/1.1/subscribe/statuses/count
```
