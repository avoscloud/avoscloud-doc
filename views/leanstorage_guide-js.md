{# 指定继承模板 #}
{% extends "./leanstorage_guide.tmpl" %}

{# --Start--变量定义，主模板使用的单词和短语在所有子模板都必须赋值 #}
{% set cloudName ="LeanCloud" %}
{% set productName ="LeanStorage" %}
{% set platform_title ="JavaScript" %}
{% set segment_code ="js" %}
{% set sdk_name ="JavaScript SDK" %}
{% set baseObjectName ="AV.Object" %}
{% set objectIdName ="id" %}
{% set updatedAtName ="updatedAt" %}
{% set createdAtName ="createdAt" %}
{% set backgroundFunctionTemplate ="xxxxInBackground" %}
{% set saveEventuallyName ="saveEventually" %}
{% set deleteEventuallyName ="deleteEventually" %}
{% set relationObjectName ="AV.Relation" %}
{% set pointerObjectName ="AV.Pointer" %}
{% set baseQueryClassName ="AV.Query" %}
{% set geoPointObjectName ="AV.GeoPoint" %}
{% set userObjectName ="AV.User" %}
{% set fileObjectName ="AV.File" %}
{% set dateType= "Date" %}
{% set byteType= "Buffer" %}
{% set link_to_acl_doc ="[JavaScript 权限管理使用指南](acl_guide-js.html)" %}
{% set funtionName_whereKeyHasPrefix = "startsWith" %}
{% set saveOptions_query= "query" %}
{% set saveOptions_fetchWhenSave= "fetchWhenSave" %}

{% block text_web_security %}
## Web 安全

如果在前端使用 JavaScript SDK，当你打算正式发布的时候，请务必配置 **Web 安全域名**。配置方式为：进入 [控制台 / 设置 / 安全中心 / **Web 安全域名**](/app.html?appid={{appid}}#/security)。这样就可以防止其他人，通过外网其他地址盗用你的服务器资源。

具体安全相关内容可以仔细阅读文档 [数据和安全](data_security.html) 。
{% endblock %}

{# --End--变量定义，主模板使用的单词和短语在所有子模板都必须赋值 #}

{# --Start--主模板留空的代码段落，子模板根据自身实际功能给予实现 #}

{% block code_quick_save_a_todo %}

```js
  // 声明一个 Todo 类型
  var Todo = AV.Object.extend('Todo');
  // 新建一个 Todo 对象
  var todo = new Todo();
  todo.set('title', '工程师周会');
  todo.set('content', '每周工程师会议，周一下午2点');
  todo.save().then(function (todo) {
    // 成功保存之后，执行其他逻辑.
    console.log('New object created with objectId: ' + todo.id);
  }, function (error) {
    // 失败之后执行其他逻辑
    console.log('Failed to create new object, with error message: ' + error.message);
  });
```
{% endblock %}

{% block code_quick_save_a_todo_with_location %}

```js
  var Todo = AV.Object.extend('Todo');
  var todo = new Todo();
  todo.set('title', '工程师周会');
  todo.set('content', '每周工程师会议，周一下午2点');
  // 只要添加这一行代码，服务端就会自动添加这个字段
  todo.set('location','会议室');
  todo.save().then(function (todo) {
    // 成功保存之后，执行其他逻辑.
  }, function (error) {
    // 失败之后执行其他逻辑
  });
```
{% endblock %}

{% block code_create_todo_object %}

```js
  // AV.Object.extend('className') 所需的参数 className 则表示对应的表名
  // 声明一个类型
  var Todo = AV.Object.extend('Todo');
```

**注意**：如果你的应用时不时出现 `Maximum call stack size exceeded` 异常，可能是因为在循环或回调中调用了 `AV.Object.extend`。有两种方法可以避免这种异常：

- 升级 SDK 到 v1.4.0 或以上版本
- 在循环或回调外声明 Class，确保不会对一个 Class 执行多次 `AV.Object.extend`

从 v1.4.0 开始，SDK 支持使用 ES6 中的 extends 语法来声明一个继承自 `AV.Object` 的类，上述的 Todo 声明也可以写作：

```js
class Todo extends AV.Object {}
// 需要向 SDK 注册这个 Class
AV.Object.register(Todo);
```
{% endblock %}

{% block code_save_object_by_cql %}

```js
  // 执行 CQL 语句实现新增一个 TodoFolder 对象
  AV.Query.doCloudQuery('insert into TodoFolder(name, priority) values("工作", 1)').then(function (data) {
    // data 中的 results 是本次查询返回的结果，AV.Object 实例列表
    var results = data.results;
  }, function (error) {
    //查询失败，查看 error
    console.log(error);
  });
```
{% endblock %}

{% block code_data_type %}

```js
  // 该语句应该只声明一次
  var TestObject = AV.Object.extend('DataTypeTest');

  var number = 2014;
  var string = 'famous film name is ' + number;
  var date = new Date();
  var array = [string, number];
  var object = { number: number, string: string };

  var testObject = new TestObject();
  testObject.set('testNumber', number);
  testObject.set('testString', string);
  testObject.set('testDate', date);
  testObject.set('testArray', array);
  testObject.set('testObject', object);
  testObject.set('testNull', null);
  testObject.save().then(function(testObject) {
    // 成功
  }, function(error) {
    // 失败
  });
```

我们**不推荐**在 `AV.Object` 中储存大块的二进制数据，比如图片或整个文件。**每个 `AV.Object` 的大小都不应超过 128 KB**。如果需要储存更多的数据，建议使用 [`AV.File`](#文件)。
{% endblock %}

{% block section_dataType_largeData %}{% endblock %}

{% block code_save_todo_folder %}

```js
  // 声明类型
  var TodoFolder = AV.Object.extend('TodoFolder');
  // 新建对象
  var todoFolder = new TodoFolder();
  // 设置名称
  todoFolder.set('name','工作');
  // 设置优先级
  todoFolder.set('priority',1);
  todoFolder.save().then(function (todo) {
    console.log('objectId is ' + todo.id);
  }, function (error) {
    console.log(error);
  });
```
{% endblock %}

{% block code_saveoption_query_example %}

```js
  new AV.Query('Wiki').first().then(function (data) {
    var wiki = data;
    var currentVersion = wiki.get('version');
    wiki.set('version', currentVersion + 1);
    wiki.save(null, {
      query: new AV.Query('Wiki').equalTo('version', currentVersion)
    }).then(function (data) {
    }, function (error) {
      if (error) {
        throw error;
      }
    });
  }, function (error) {
    if (error) {
      throw error;
    }
  });
```
{% endblock %}

{% block code_get_todo_by_objectId %}

```js
  var query = new AV.Query('Todo');
  query.get('57328ca079bc44005c2472d0').then(function (data) {
    // 成功获得实例
    // data 就是 id 为 57328ca079bc44005c2472d0 的 Todo 对象实例
  }, function (error) {
    // 失败了
  });
```
{% endblock %}

{% block code_fetch_todo_by_objectId %}

```js
  // 第一个参数是 className，第二个参数是 objectId
  var todo = AV.Object.createWithoutData('Todo', '5745557f71cfe40068c6abe0');
  todo.fetch().then(function () {
    var title = todo.get('title');// 读取 title
    var content = todo.get('content');// 读取 content
  }, function (error) {

  });
```
{% endblock %}

{% block code_save_callback_get_objectId %}

```js
  var todo = new Todo();
  todo.set('title', '工程师周会');
  todo.set('content', '每周工程师会议，周一下午2点');
  todo.save().then(function (todo) {
    // 成功保存之后，执行其他逻辑
    // 获取 objectId
    var objectId = todo.id;
  }, function (error) {
    // 失败之后执行其他逻辑
    console.log(error);
  });
```
{% endblock %}

{% block code_access_todo_folder_properties %}

```js
  var query = new AV.Query('Todo');
  query.get('558e20cbe4b060308e3eb36c').then(function (todo) {
    // 成功获得实例
    // todo 就是 id 为 558e20cbe4b060308e3eb36c 的 Todo 对象实例
    var priority = todo.get('priority');
    var location = todo.get('location');
    var title = todo.get('title');
    var content = todo.get('content');

    // 获取三个特殊属性
    var objectId = todo.id;
    var updatedAt = todo.updatedAt;
    var createdAt = todo.createdAt;

    //Wed May 11 2016 09:36:32 GMT+0800 (CST)
    console.log(createdAt);
  }, function (error) {
    // 失败了
    console.log(error);
  });
```
{% endblock %}

{% block code_object_fetch %}

```js
  // 使用已知 objectId 构建一个 AV.Object
  var todo = new Todo();
  todo.id = '5590cdfde4b00f7adb5860c8';
  todo.fetch().then(function (todo) {
    // // todo 是从服务器加载到本地的 Todo 对象
    var priority = todo.get('priority');
  }, function (error) {

  });
```
{% endblock %}

{% block code_object_fetchWhenSave %}

```js
  //设置 fetchWhenSave 为 true
  todo.fetchWhenSave(true);
  todo.save().then(function () {
    // 保存成功
  }, function (error) {
    // 失败了
    console.log(error);
  });
```
{% endblock %}

{% block code_object_fetch_with_keys %}

```js
  // 使用已知 objectId 构建一个 AV.Object
  var todo = new Todo();
  todo.id = '5590cdfde4b00f7adb5860c8';
  todo.fetch({include:'priority,location'},{}).then(function (todo) {
    // 获取到本地
  }, function (error) {
    // 失败了
    console.log(error);
  });
```
{% endblock %}

{% block code_update_todo_location %}

```js
  // 已知 objectId，创建 AVObject
  // 第一个参数是 className，第二个参数是该对象的 objectId
  var todo = AV.Object.createWithoutData('Todo', '558e20cbe4b060308e3eb36c');
  // 更改属性
  todo.set('location', '二楼大会议室');
  // 保存
  todo.save().then(function () {
    // 保存成功
  }, function (error) {
    // 失败了
    console.log(error);
  });
```
{% endblock %}

{% block code_update_todo_content_with_objectId %}

```js
  // 第一个参数是 className，第二个参数是 objectId
  var todo = AV.Object.createWithoutData('Todo', '5745557f71cfe40068c6abe0');
  // 修改属性
  todo.set('content', '每周工程师会议，本周改为周三下午3点半。');
  // 保存到云端
  todo.save();
```

{% endblock %}

{% block code_update_object_by_cql %}

```js
  // 执行 CQL 语句实现更新一个 TodoFolder 对象
  AV.Query.doCloudQuery('update TodoFolder set name="家庭" where objectId="558e20cbe4b060308e3eb36c"')
  .then(function (data) {
    // data 中的 results 是本次查询返回的结果，AV.Object 实例列表
    var results = data.results;
  }, function (error) {
    //查询失败，查看 error
    console.log(error);
  });
```
{% endblock %}

{% block code_atomic_operation_increment %}

```js
  var todo = AV.Object.createWithoutData('Todo', '57328ca079bc44005c2472d0');
  todo.set('views', 0);
  todo.save().then(function (todo) {
    todo.increment('views', 1);
    todo.fetchWhenSave(true);
    // 也可以指定增加一个特定的值
    // 例如一次性加 5
    todo.increment('views', 5);
    todo.save().then(function (data) {
      // 因为使用了 fetchWhenSave 选项，save 调用之后，如果成功的话，对象的计数器字段是当前系统最新值。
    }, function (error) {
      if (error) {
        throw error;
      }
    });
  }, function (error) {
    if (error) {
      throw error;
    }
  });
```
{% endblock %}

{% block code_atomic_operation_array %}

* `AV.Object.add('arrayKey',arrayValue)`<br>
  将指定对象附加到数组末尾。
* `AV.Object.addUnique('arrayKey',arrayValue);`<br>
  如果不确定某个对象是否已包含在数组字段中，可以使用此操作来添加。对象的插入位置是随机的。
* `AV.Object.remove('arrayKey',arrayValue);`<br>
  从数组字段中删除指定对象的所有实例。

{% endblock %}

{% block code_set_array_value %}

```js
  var reminder1 = new Date('2015-11-11 07:10:00');
  var reminder2 = new Date('2015-11-11 07:20:00');
  var reminder3 = new Date('2015-11-11 07:30:00');

  var reminders = [reminder1, reminder2, reminder3];

  var todo = new AV.Object('Todo');
  // 指定 reminders 是做一个 Date 对象数组
  todo.addUnique('reminders', reminders);
  todo.save().then(function (todo) {
   console.log(todo.id);
  }, function (error) {
    // 失败了
    console.log(error);
  });
```
{% endblock %}

{% block code_delete_todo_by_objectId %}

```js
  var todo = AV.Object.createWithoutData('Todo', '57328ca079bc44005c2472d0');
  todo.destroy().then(function (success) {
    // 删除成功
  }, function (error) {
    // 删除失败
  });
```
{% endblock %}

{% block code_delete_todo_by_cql %}

```js
  // 执行 CQL 语句实现删除一个 Todo 对象
  AV.Query.doCloudQuery('delete from Todo where objectId="558e20cbe4b060308e3eb36c"').then(function (data) {
  }, function (error) {
  });
```
{% endblock %}


{% block code_batch_operation %}

```js
  var avObjectArray = [];// 构建一个本地的 AV.Object 对象数组

   // 批量创建、更新
  AV.Object.saveAll(avObjectArray).then(function (avobjs) {
  }, function (error) {
  });
  // 批量删除
  AV.Object.destroyAll(avObjectArray).then(function (avobjs) {
  }, function (error) {
  });
  // 批量获取
  AV.Object.fetchAll(avObjectArray).then(function (avobjs) {
  }, function (error) {
  });
```
{% endblock %}


{% block code_batch_set_todo_completed %}

```js
  var query = new AV.Query('Todo');
  query.find().then(function (todos) {
      for (var i = 0; i < todos.length; i++) {
          var todo = todos[i];
          todo['status'] = 1;
      }
      AV.Object.saveAll(todos).then(function (success) {
      }, function (error) {
      });
  }, function (error) {
  });
```
{% endblock %}


{% block text_work_in_background %}{% endblock %}
{% block save_eventually %}{% endblock %}

{% block code_relation_todoFolder_one_to_many_todo %}

```js
  var todoFolder = new AV.Object('TodoFolder');
  todoFolder.set('name', '工作');
  todoFolder.set('priority', 1);

  var todo1 = new AV.Object('Todo');
  todo1.set('title', '工程师周会');
  todo1.set('content', '每周工程师会议，周一下午2点');
  todo1.set('location', '会议室');

  var todo2 = new AV.Object('Todo');
  todo2.set('title', '维护文档');
  todo2.set('content', '每天 16：00 到 18：00 定期维护文档');
  todo2.set('location', '当前工位');

  var todo3 = new AV.Object('Todo');
  todo3.set('title', '发布 SDK');
  todo3.set('content', '每周一下午 15：00');
  todo3.set('location', 'SA 工位');

  var localTodos = [todo1, todo2, todo3];
  AV.Object.saveAll(localTodos).then(function (cloudTodos) {
      var relation = todoFolder.relation('containedTodos'); // 创建 AV.Relation
      for (var i = 0; i < cloudTodos.length; i++) {
          var todo = cloudTodos[i];
          relation.add(todo);// 建立针对每一个 Todo 的 Relation
      }
      todoFolder.save();// 保存到云端
  }, function (error) {
  });
```
{% endblock %}


{% block code_pointer_comment_one_to_many_todoFolder %}

```js
  var comment = new AV.Object('Comment');// 构建 Comment 对象
  comment.set('like', 1);// 如果点了赞就是 1，而点了不喜欢则为 -1，没有做任何操作就是默认的 0
  comment.set('content', '这个太赞了！楼主，我也要这些游戏，咱们团购么？');
  // 假设已知被分享的该 TodoFolder 的 objectId 是 5735aae7c4c9710060fbe8b0
  var targetTodoFolder = AV.Object.createWithoutData('TodoFolder', '5735aae7c4c9710060fbe8b0');
  comment.set('targetTodoFolder', targetTodoFolder);
  comment.save();//保存到云端
```
{% endblock %}


{% block code_create_geoPoint %}
``` js
  // 第一个参数是： latitude ，纬度
  // 第二个参数是： longitude，经度
  var point1 = new AV.GeoPoint(39.9, 116.4);

  // 以下是创建 AV.GeoPoint 对象不同的方法
  var point2 = new AV.GeoPoint([12.7, 72.2]);
  var point3 = new AV.GeoPoint({ latitude: 30, longitude: 30 });
```
{% endblock %}


{% block code_use_geoPoint %}
``` objc
[todo setObject:point forKey:@"whereCreated"];
```
{% endblock %}

{% block text_deserialize_and_serialize %}
<!-- js 以及 ts 没有序列化和反序列化的需求 -->
{% endblock %}

{% block code_data_protocol_save_date %}
```js
  var testDate = new Date('2016-06-04');
  var testAVObject = new AV.Object('TestClass');
  testAVObject.set('testDate', testDate);
  testAVObject.save();
```
{% endblock %}


{% block code_create_avfile_by_stream_data %}

```js
  var data = { base64: '6K+077yM5L2g5Li65LuA5LmI6KaB56C06Kej5oiR77yf' };
  var file = new AV.File('resume.txt', data);
  file.save().then(function (savedFile) {
  }, function (error) {
  });

  var bytes = [0xBE, 0xEF, 0xCA, 0xFE];
  var byteArrayFile = new AV.File('myfile.txt', bytes);
  byteArrayFile.save();
```
{% endblock %}


{% block code_create_avfile_from_local_path %}
假设在页面上有如下文件选择框：

```html
<input type="file" id="photoFileUpload"/>
```
上传文件对应的代码如下：
```js
    var fileUploadControl = $('#photoFileUpload')[0];
    if (fileUploadControl.files.length > 0) {
      var file = fileUploadControl.files[0];
      var name = 'avatar.jpg';

      var avFile = new AV.File(name, file);
      avFile.save().then(function(obj) {
        // 数据保存成功
        console.log(obj.url());
      }, function(error) {
        // 数据保存失败
        console.log(error);
      });
    }
```
{% endblock %}

{% block code_create_avfile_from_url %}

```js
  var file = AV.File.withURL('Satomi_Ishihara.gif', 'http://ww3.sinaimg.cn/bmiddle/596b0666gw1ed70eavm5tg20bq06m7wi.gif');
  file.save().then(function (savedFile) {
  }, function (error) {
  });
```
{% endblock %}


{% block code_upload_file %}
如果仅是想简单的上传，可以直接在 Web 前端使用 AV.File 上面的相关方法。但真实使用场景中，还有很多开发者需要自行实现一个上传接口，对数据做更多的处理。

以下是一个在 Web 中完整上传一张图片的 Demo，包括前端与 Node.js 服务端代码。服务端推荐使用 LeanCloud 推出的「[云引擎](leanengine_overview.html)」，非常出色的 Node.js 环境。

前端页面（比如:fileUpload.html）：
```html
// 页面元素（限制上传为图片类型，使用时可自行修改 accept 属性）
<form id="upload-file-form" class="upload" enctype="multipart/form-data">
  <input name="attachment" type="file" accept="image/gif, image/jpeg, image/png">
</form>
```
纯前端调用方式：

```javascript
// 前端代码，基于 jQuery
function uploadPhoto() {
  var uploadFormDom = $('#upload-file-form');
  var uploadInputDom = uploadFormDom.find('input[type=file]');
  // 获取浏览器 file 对象
  var files = uploadInputDom[0].files;
  // 创建 formData 对象
  var formData = new window.FormData(uploadFormDom[0]);
  if (files.length) {
    $.ajax({
      // 注意，这个 url 地址是一个例子，真实使用时需替换为自己的上传接口 url
      url: 'https://leancloud.cn/xxx/xxx/upload',
      method: 'post',
      data: formData,
      processData: false,
      contentType: false
    }).then((data) => {
      // 上传成功，服务端设置返回
      console.log(data);
    });
  }
};
```

在服务端可以编写如下代码：

```javascript
// 服务端代码，基于 Node.js、Express
var AV = require('leanengine');
// 服务端需要使用 connect-busboy（通过 npm install 安装）
var busboy = require('connect-busboy');
// 使用这个中间件
app.use(busboy());

// 上传接口方法（使用时自行配置到 router 中）
function uploadFile (req, res) {
  if (req.busboy) {
    var base64data = [];
    var pubFileName = '';
    var pubMimeType = '';
    req.busboy.on('file', (fieldname, file, fileName, encoding, mimeType) => {
      var buffer = '';
      pubFileName = fileName;
      pubMimeType = mimeType;
      file.setEncoding('base64');
      file.on('data', function(data) {
        buffer += data;
      }).on('end', function() {
        base64data.push(buffer);
      });
    }).on('finish', function() {
      var f = new AV.File(pubFileName, {
        // 仅上传第一个文件（多个文件循环创建）
        base64: base64data[0]
      });
      try {
        f.save().then(function(fileObj) {
          // 向客户端返回数据
          res.send({
            fileId: fileObj.id,
            fileName: fileObj.name(),
            mimeType: fileObj.metaData().mime_type,
            fileUrl: fileObj.url()
          });
        });
      } catch (error) {
        console.log('uploadFile - ' + error);
        res.status(502);
      }
    })
    req.pipe(req.busboy);
  } else {
    console.log('uploadFile - busboy undefined.');
    res.status(502);
  }
};
```
{% endblock %}

{% block text_upload_file_with_progress %}{% endblock %}
{% block text_download_file_with_progress %}{% endblock %}

{% block text_file_query %}

### 文件查询
文件的查询依赖于文件在系统中的关系模型，例如，用户的头像，有一些用户习惯直接在 `_User` 表中直接使用一个 `avatar` 列，然后里面存放着一个 url 指向一个文件的地址，但是，我们更推荐用户使用 Pointer 来关联一个 {{userObjectName}} 和 {{fileObjectName}}，代码如下：

{% block code_user_pointer_file %}

```js
    var data = { base64: '文件的 base64 编码' };
    var avatar = new AV.File('avatar.png', data);

    var user = new AV.User();
    var randomUsername = 'Tom';
    user.setUsername(randomUsername)
    user.setPassword('leancloud');
    user.set('avatar',avatar);
    user.signUp().then(function (u){
    });
```
{% endblock %}

{% endblock %}

{% block code_file_image_thumbnail %}

```js
  //获得宽度为100像素，高度200像素的缩略图
  var url = file.thumbnailURL(100, 200);
```
{% endblock %}


{% block code_file_metadata %}

```js
    // 获取文件大小
    var size = file.size();
    // 上传者(AV.User) 的 objectId，如果未登录，默认为空
    var ownerId = file.ownerId();

    // 获取文件的全部元信息
    var metadata = file.metaData();
    // 设置文件的作者
    file.metaData('author', 'LeanCloud');
    // 获取文件的格式
    var format = file.metaData('format');
```
{% endblock %}


{% block code_file_delete %}

```js
  var file = AV.File.createWithoutData('552e0a27e4b0643b709e891e');
  file.destroy().then(function (success) {
  }, function (error) {
  });
```
{% endblock %}


{% block code_cache_operations_file %}{% endblock %}

{% block text_https_access_for_ios9 %}{% endblock %}

{% block code_create_query_by_className %}

```js
  var query = new AV.Query('Todo');
```
{% endblock %}


{% block code_priority_equalTo_zero_query %}

```js
  var query = new AV.Query('Todo');
  // 查询 priority 是 0 的 Todo
  query.equalTo('priority', 0);
  query.find().then(function (results) {
      var priorityEqualsZeroTodos = results;
  }, function (error) {
  });
```
{% endblock %}


{% block code_priority_equalTo_zero_and_one_wrong_example %}

```js
  var query = new AV.Query('Todo');
  query.equalTo('priority', 0);
  query.equalTo('priority', 1);
  query.find().then(function (results) {
  // 如果这样写，第二个条件将覆盖第一个条件，查询只会返回 priority = 1 的结果
  }, function (error) {
  });
```
{% endblock %}


{% block table_logic_comparison_in_query %}
逻辑操作 | AVQuery 方法|
---|---
等于 | `equalTo`
不等于 |  `notEqualTo`
大于 | `greaterThan`
大于等于 | `greaterThanOrEqualTo`
小于 | `lessThan`
小于等于 | `lessThanOrEqualTo`
{% endblock %}

{% block code_query_lessThan %}

```js
  var query = new AV.Query('Todo');
  query.lessThan('priority', 2);
```
{% endblock %}


{% block code_query_greaterThanOrEqualTo %}

```js
  query.greaterThanOrEqualTo('priority',2);
```
{% endblock %}


{% block code_query_with_regular_expression %}

```js
  var query = new AV.Query('Todo');
  var regExp = new RegExp('[\u4e00-\u9fa5]', 'i');
  query.matches('title', regExp);
  query.find().then(function (results) {
  }, function (error) {
  });
```
{% endblock %}


{% block code_query_with_contains_keyword %}

```js
  query.contains('title','李总');
```
{% endblock %}


{% block code_query_with_not_contains_keyword_using_regex %}
```js
  var query = new AV.Query('Todo');
  var regExp = new RegExp('^((?!机票).)*$', 'i');
  query.matches('title', regExp);
```
{% endblock %}


{% block code_query_with_not_contains_keyword %}

```js
  var query = new AV.Query('Todo');
  var filterArray = ['出差', '休假'];
  query.notContainedIn('title', filterArray);
```
{% endblock %}


{% block code_query_array_contains_using_equalsTo %}

```js
  var query = new AV.Query('Todo');
  var reminderFilter = [new Date('2015-11-11 08:30:00')];
  query.containsAll('reminders', reminderFilter);

  // 也可以使用 equals 接口实现这一需求
  var targetDateTime = new Date('2015-11-11 08:30:00');
  query.equalTo('reminders', targetDateTime);
```
{% endblock %}


{% block code_query_array_contains_all %}

```js
  var query = new AV.Query('Todo');
  var reminderFilter = [new Date('2015-11-11 08:30:00'), new Date('2015-11-11 09:30:00')];
  query.containsAll('reminders', reminderFilter);
```
{% endblock %}


{% block code_query_whereHasPrefix %}

```js
  // 找出开头是「早餐」的 Todo
  var query = new AV.Query('Todo');
  query.startsWith('content', '早餐');

  // 找出包含 「bug」 的 Todo
  var query = new AV.Query('Todo');
  query.contains('content', 'bug');
```
{% endblock %}


{% block code_query_comment_by_todoFolder %}

```js
  var query = new AV.Query('Comment');
  var todoFolder = AV.Object.createWithoutData('TodoFolder', '5735aae7c4c9710060fbe8b0');
  query.equalTo('targetTodoFolder', todoFolder);

  // 想在查询的同时获取关联对象的属性则一定要使用 `include` 接口用来指定返回的 `key`
  query.include('targetTodoFolder');
```
{% endblock %}


{% block code_create_tag_object %}

```js
  var tag = new AV.Object('Todo');
  tag.set('name', '今日必做');
  tag.save();
```
{% endblock %}


{% block code_create_family_with_tag %}

```js
  var tag1 = new AV.Object('Todo');
  tag1.set('name', '今日必做');

  var tag2 = new AV.Object('Todo');
  tag2.set('name', '老婆吩咐');

  var tag3 = new AV.Object('Todo');
  tag3.set('name', '十分重要');

  var tags = [tag1, tag2, tag3];
  AV.Object.saveAll(tags).then(function (savedTags) {

      var todoFolder = new AV.Object('TodoFolder');
      todoFolder.set('name', '家庭');
      todoFolder.set('priority', 1);

      var relation = todoFolder.relation('tags');
      relation.add(tag1);
      relation.add(tag2);
      relation.add(tag3);

      todoFolder.save();
  }, function (error) {
  });
```
{% endblock %}


{% block code_query_tag_for_todoFolder %}

```js
  var todoFolder = AV.Object.createWithoutData('Todo', '5735aae7c4c9710060fbe8b0');
  var relation = todoFolder.relation('tags');
  var query = relation.query();
  query.find().then(function (results) {
    // results 是一个 AV.Object 的数组，它包含所有当前 todoFolder 的 tags
  }, function (error) {
  });
```
{% endblock %}


{% block code_query_todoFolder_with_tag %}

```js
  var targetTag = AV.Object.createWithoutData('Tag', '5655729900b0bf3785ca8192');
  var query = new AV.Query('TodoFolder');
  query.equalTo('tags', targetTag);
  query.find().then(function (results) {
  // results 是一个 AV.Object 的数组
  // results 指的就是所有包含当前 tag 的 TodoFolder
  }, function (error) {
  });
```
{% endblock %}


{% block code_query_comment_include_todoFolder %}

```js
  var commentQuery = new AV.Query('Comment');
  commentQuery.descending('createdAt');
  commentQuery.limit(10);
  commentQuery.include('targetTodoFolder');// 关键代码，用 include 告知服务端需要返回的关联属性对应的对象的详细信息，而不仅仅是 objectId
  commentQuery.include('targetTodoFolder.targetAVUser');// 关键代码，同上，会返回 targetAVUser 对应的对象的详细信息，而不仅仅是 objectId
  commentQuery.find().then(function (comments) {
      // comments 是最近的十条评论, 其 targetTodoFolder 字段也有相应数据
      for (var i = 0; i < comments.length; i++) {
          var comment = comments[i];
          // 并不需要网络访问
          var todoFolder = comment.get('targetTodoFolder');
          var avUser = todoFolder.get('targetAVUser');
      }
  }, function (error) {
  });
```
{% endblock %}


{% block code_query_comment_match_query_todoFolder %}

```js
  // 构建内嵌查询
  var innerQuery = new AV.Query('TodoFolder');
  innerQuery.greaterThan('likes', 20);

  // 将内嵌查询赋予目标查询
  var query = new AV.Query('Comment');

  // 执行内嵌操作
  query.matchesQuery('targetTodoFolder', innerQuery);
  query.find().then(function (results) {
     // results 就是符合超过 20 个赞的 TodoFolder 这一条件的 Comment 对象集合
  }, function (error) {
  });

  query.doesNotMatchQuery('targetTodoFolder', innerQuery);
  // 如此做将查询出 likes 小于或者等于 20 的 TodoFolder 的 Comment 对象
```
{% endblock %}


{% block code_query_find_first_object %}

```js
  var query = new AV.Query('Comment');
  query.equalTo('priority', 0);
  query.first().then(function (data) {
    // data 就是符合条件的第一个 AV.Object
  }, function (error) {
  });
```
{% endblock %}


{% block code_set_query_limit %}

```js
  var query = new AV.Query('Todo');
  var now = new Date();
  query.lessThanOrEqualTo('createdAt', now);//查询今天之前创建的 Todo
  query.limit(10);// 最多返回 10 条结果
```
{% endblock %}


{% block code_set_skip_for_pager %}

```js
  var query = new AV.Query('Todo');
  var now = new Date();
  query.lessThanOrEqualTo('createdAt', now);//查询今天之前创建的 Todo
  query.limit(10);// 最多返回 10 条结果
  query.skip(20);// 跳过 20 条结果
```
{% endblock %}

{% block code_query_select_keys %}

```js
  var query = new AV.Query('Todo');
  query.select('title', 'content');
  query.find().then(function (results) {
      for (var i = 0; i < results.length; i++) {
          var todo = results[i];
          var title = todo.get('title');
          var content = todo.get('content');
          var location_1 = todo.get('location');
      }
  }, function (error) {
  });
```
{% endblock %}

{% block code_query_select_pointer_keys %}

```js
    query.select('owner.username');
```

{% endblock %}

{% block code_query_count %}

```js
  var query = new AV.Query('Todo');
  query.equalTo('status', 1);
  query.count().then(function (count) {
      console.log(count);
  }, function (error) {
  });
```
{% endblock %}


{% block code_query_orderby %}

```js
  // 按时间，升序排列
  query.ascending('createdAt');

  // 按时间，降序排列
  query.descending('createdAt');
```
{% endblock %}


{% block code_query_orderby_on_multiple_keys %}

```js
  var query = new AV.Query('Todo');
  query.ascending('priority');
  query.descending('createdAt');
```
{% endblock %}


{% block code_query_with_or %}

```js
  var priorityQuery = new AV.Query('Todo');
  priorityQuery.greaterThanOrEqualTo('priority', 3);

  var statusQuery = new AV.Query('Todo');
  statusQuery.equalTo('status', 1);

  var query = AV.Query.or(priorityQuery, statusQuery);
  // 返回 priority 大于等于 3 或 status 等于 1 的 Todo
```
{% endblock %}


{% block code_query_with_and %}

```js
  var priorityQuery = new AV.Query('Todo');
  priorityQuery.lessThan('priority', 3);

  var statusQuery = new AV.Query('Todo');
  statusQuery.equalTo('status', 0);

  var query = AV.Query.and(priorityQuery, statusQuery);
  // 返回 priority 小于 3 并且 status 等于 0 的 Todo
```
{% endblock %}


{% block code_query_where_keys_exist %}

```js
  var aTodoAttachmentImage = AV.File.withURL('attachment.jpg', 'http://www.zgjm.org/uploads/allimg/150812/1_150812103912_1.jpg');
  var todo = new AV.Object('Todo');
  todo.set('images', aTodoAttachmentImage);
  todo.set('content', '记得买过年回家的火车票！！！');
  todo.save();

  var query = new AV.Query('Todo');
  query.exists('images');
  query.find().then(function (results) {
    // results 返回的就是有图片的 Todo 集合
  }, function (error) {
  });

  // 使用空值查询获取没有图片的 Todo
  query.doesNotExist('images');
```
{% endblock %}


{% block code_query_by_cql %}

```js
  var cql = 'select * from Todo where status = 1';
  AV.Query.doCloudQuery(cql).then(function (data) {
      // results 即为查询结果，它是一个 AV.Object 数组
      var results = data.results;
  }, function (error) {
  });
  cql = 'select * from %@ where status = 1';
  AV.Query.doCloudQuery(cql).then(function (data) {
      // 获取符合查询的数量
      var count = data.count;
  }, function (error) {
  });
```
{% endblock %}


{% block code_query_by_cql_with_placeholder %}

```js
  // 带有占位符的 cql 语句
  var cql = 'select * from Todo where status = ? and priority = ?';
  var pvalues = [0, 1];
  AV.Query.doCloudQuery(cql, pvalues).then(function (data) {
      // results 即为查询结果，它是一个 AV.Object 数组
      var results = data.results;
  }, function (error) {
  });
```
{% endblock %}


{% block text_query_cache_intro %}{% endblock %}

{% block code_set_cache_policy %}{% endblock %}

{% block table_cache_policy %}{% endblock %}

{% block code_query_geoPoint_near %}

```js
  var query = new AV.Query('Todo');
  var point = new AV.GeoPoint('39.9', '116.4');
  query.withinKilometers('whereCreated', point, 2.0);
  query.find().then(function (results) {
      var nearbyTodos = results;
  }, function (error) {
  });
```


在上面的代码中，`nearbyTodos` 返回的是与 `point` 这一点按距离排序（由近到远）的对象数组。注意：**如果在此之后又使用了 `ascending` 或 `descending` 方法，则按距离排序会被新排序覆盖。**
{% endblock %}

{% block text_platform_geoPoint_notice %}{% endblock %}

{% block code_query_geoPoint_within %}

```js
  var query = new AV.Query('Todo');
  var point = new AV.GeoPoint('39.9', '116.4');
  query.withinKilometers('whereCreated', point, 2.0);
```

{% endblock %} code_object_fetch_with_keys


{% block link_to_relation_guide_doc %}[JavaScript 数据模型设计指南](relation_guide-js.html){% endblock %}

{% set link_to_sms_guide_doc = '[JavaScript 短信服务使用指南](sms_guide-js.html#注册验证)' %}

{% block code_send_sms_code_for_loginOrSignup %}

```js
  AV.Cloud.requestSmsCode('13577778888').then(function (success) {
  }, function (error) {
  });
```
{% endblock %}


{% block code_verify_sms_code_for_loginOrSignup %}

```js
  AV.User.signUpOrlogInWithMobilePhone('13577778888', '123456').then(function (success) {
    // 成功
  }, function (error) {
    // 失败
  });
```
{% endblock %}


{% block code_user_signUp_with_username_and_password %}

```js
  // 新建 AVUser 对象实例
  var user = new AV.User();
  // 设置用户名
  user.setUsername('Tom');
  // 设置密码
  user.setPassword('cat!@#123');
  // 设置邮箱
  user.setEmail('tom@leancloud.cn');
  user.signUp().then(function (loginedUser) {
      console.log(loginedUser);
  }, (function (error) {
  }));
```
{% endblock %}

{% block text_logInOrSignUp_with_authData %}
#### 第三方账号登录

为了简化用户注册的繁琐流程，许多应用都在登录界面提供了第三方社交账号登录的按钮选项，例如微信、QQ、微博、豆瓣、Twitter、FaceBook 等，以此来提高用户体验。LeanCloud 封装的 {{userObjectName}} 对象也支持通过第三方账号的 accessToken 信息来创建一个用户。例如，使用微信授权信息创建 {{userObjectName}} 的代码如下：

```js
  AV.User.signUpOrlogInWithAuthData({
      "openid": "oPrJ7uM5Y5oeypd0fyqQcKCaRv3o",
      "access_token": "OezXcEiiBSKSxW0eoylIeNFI3H7HsmxM7dUj1dGRl2dXJOeIIwD4RTW7Iy2IfJePh6jj7OIs1GwzG1zPn7XY_xYdFYvISeusn4zfU06NiA1_yhzhjc408edspwRpuFSqtYk0rrfJAcZgGBWGRp7wmA",
      "expires_at": "2016-01-06T11:43:11.904Z"
  }, 'weixin').then(function (s) {
  }, function (e) {

  });
```

其他的平台可以参考如上代码。

{% endblock %}

{% block code_send_verify_email %}

```js
  AV.User.requestEmailVerfiy('abc@xyz.com').then(function (result) {
      console.log(JSON.stringify(result));
  }, function (error) {
      console.log(JSON.stringify(error));
  });
```
{% endblock %}

{% block code_user_logIn_with_username_and_password %}

```js
  AV.User.logIn('Tom', 'cat!@#123').then(function (loginedUser) {
    console.log(loginedUser);
  }, function (error) {
  });
```
{% endblock %}


{% block code_user_logIn_with_mobilephonenumber_and_password %}

```js
  AV.User.logInWithMobilePhone('13577778888', 'cat!@#123').then(function (loginedUser) {
      console.log(loginedUser);
  }, (function (error) {
  }));
```
{% endblock %}


{% block code_user_logIn_requestLoginSmsCode %}

```js
AV.User.requestLoginSmsCode('13577778888').then(function (success) {
  }, function (error) {
  });
```
{% endblock %}


{% block code_user_logIn_with_smsCode %}

```js
  AV.User.logInWithMobilePhoneSmsCode('13577778888', '238825').then(function (success) {
  }, function (error) {
  });
```
{% endblock %}

{% block code_get_user_properties %}

```js
  AV.User.logIn('Tom', 'cat!@#123').then(function (loginedUser) {
    console.log(loginedUser);
    var username = loginedUser.getUsername();
    var email = loginedUser.getEmail();
    // 请注意，密码不会明文存储在云端，因此密码只能重置，不能查看
  }, function (error) {
  });
```
{% endblock %}

{% block code_set_user_custom_properties %}

```js
  AV.User.logIn('Tom', 'cat!@#123').then(function (loginedUser) {
    loginedUser.set('age', 25);
    loginedUser.save();
  }, function (error) {
    // 失败了
    console.log(error);
  });
```
{% endblock %}

{% block code_update_user_custom_properties %}

```js
  AV.User.logIn('Tom', 'cat!@#123').then(function (loginedUser) {
    // 25
    console.log(loginedUser.get('age'));
    loginedUser.set('age', 18);
    return loginedUser.save();
  }).then(function(loginedUser) {
    // 18
    console.log(loginedUser.get('age'));
  }).catch(function(error) {
    // 失败了
    console.log(error);
  });
```
{% endblock %}

{% block code_reset_password_by_email %}

```js
  AV.User.requestPasswordReset('myemail@example.com').then(function (success) {
  }, function (error) {
  });
```
{% endblock %}


{% block code_reset_password_by_mobilephoneNumber %}

```js
  AV.User.requestPasswordResetBySmsCode('18612340000').then(function (success) {
  }, function (error) {
  });
```
{% endblock %}


{% block code_reset_password_by_mobilephoneNumber_verify %}

```js
  AV.User.resetPasswordBySmsCode('123456', 'thenewpassword').then(function (success) {
  }, function (error) {
  });
```
{% endblock %}


{% block code_current_user %}

```js
  var currentUser = AV.User.current();
  if (currentUser) {
     // 跳转到首页
  }
  else {
     //currentUser 为空时，可打开用户注册界面…
  }
```
{% endblock %}


{% block code_current_user_logout %}

```js
  AV.User.logOut();
  // 现在的 currentUser 是 null 了
  var currentUser = AV.User.current();
```
{% endblock %}


{% block code_query_user %}

```js
  var query = new AV.Query('_User');
```
{% endblock %}


{% block text_subclass %}{% endblock %}
{% block text_sns %}{% endblock %}
{% block text_feedback %}{% endblock %}
{% block link_to_in_app_search_doc %}[JavaScript 应用内搜索指南](app_search_guide.html){% endblock %}
{% block link_to_status_system_doc %}[JavaScript 应用内社交模块](status_system.html#JavaScript_SDK){% endblock %}

{% block text_js_promise %}
## Promise

除了回调函数之外，每一个在 LeanCloud JavaScript SDK 中的异步方法都会返回一个
 `Promise`。使用 `Promise`，你的代码可以比原来的嵌套 callback 的方法看起来优雅得多。

```javascript
// 这是一个比较完整的例子，具体方法可以看下面的文档
// 查询某个 AV.Object 实例，之后进行修改
var query = new AV.Query('TestObject');
query.equalTo('name', 'hjiang');
// find 方法是一个异步方法，会返回一个 Promise，之后可以使用 then 方法
query.find().then(function(results) {
  // 返回一个符合条件的 list
  var obj = results[0];
  obj.set('phone', '182xxxx5548');
  // save 方法也是一个异步方法，会返回一个 Promise，所以在此处，你可以直接 return 出去，后续操作就可以支持链式 Promise 调用
  return obj.save();
}).then(function() {
  // 这里是 save 方法返回的 Promise
  console.log('设置手机号码成功');
}).catch(function(error) {
  // catch 方法写在 Promise 链式的最后，可以捕捉到全部 error
  console.log(error);
});
```

### then 方法

每一个 Promise 都有一个叫 `then` 的方法，这个方法接受一对 callback。第一个 callback 在 promise 被解决（`resolved`，也就是正常运行）的时候调用，第二个会在 promise 被拒绝（`rejected`，也就是遇到错误）的时候调用。

```javascript
obj.save().then(function(obj) {
  //对象保存成功
}, function(error) {
  //对象保存失败，处理 error
});
```

其中第二个参数是可选的。

### try、catch 和 finally 方法

你还可以使用 `try,catch,finally` 三个方法，将逻辑写成：

```javascript
obj.save().try(function(obj) {
  //对象保存成功
}).catch(function(error) {
  //对象保存失败，处理 error
}).finally(function(){
  //无论成功还是失败，都调用到这里
});
```

类似语言里的 `try ... catch ... finally` 的调用方式来简化代码。

为了兼容其他 Promise 库，我们提供了下列别名：

* `AV.Promise#done` 等价于 `try` 方法
* `AV.Promise#fail` 等价于 `catch` 方法
* `AV.Promise#always` 等价于 `finally` 方法

因此上面例子也可以写成：

```javascript
obj.save().done(function(obj) {
  //对象保存成功
}).fail(function(error) {
  //对象保存失败，处理 error
}).always(function(){
  //无论成功还是失败，都调用到这里
});
```

### 将 Promise 组织在一起

Promise 比较神奇，可以代替多层嵌套方式来解决发送异步请求代码的调用顺序问题。如果一个 Promise 的回调会返回一个 Promise，那么第二个 then 里的 callback 在第一个 then
的 callback 没有解决前是不会解决的，也就是所谓 **Promise Chain**。

```javascript
var query = new AV.Query('Student');
query.addDescending('gpa');
query.find().then(function(students) {
  students[0].set('valedictorian', true);
  return students[0].save();

}).then(function(valedictorian) {
  return query.find();

}).then(function(students) {
  students[1].set('salutatorian', true);
  return students[1].save();

}).then(function(salutatorian) {
  // Everything is done!

});
```

### 错误处理

如果任意一个在链中的 Promise 返回一个错误的话，所有的成功的 callback 在接下
来都会被跳过直到遇到一个处理错误的 callback。

处理 error 的 callback 可以转换 error 或者可以通过返回一个新的 Promise 的方式来处理它。你可以想象成拒绝的 promise 有点像抛出异常，而 error callback 函数则像是一个 catch 来处理这个异常或者重新抛出异常。

```javascript
var query = new AV.Query('Student');
query.addDescending('gpa');
query.find().then(function(students) {
  students[0].set('valedictorian', true);
  // 强制失败
  return AV.Promise.error('There was an error.');

}).then(function(valedictorian) {
  // 这里的代码将被忽略
  return query.find();

}).then(function(students) {
  // 这里的代码也将被忽略
  students[1].set('salutatorian', true);
  return students[1].save();
}, function(error) {
  // 这个错误处理函数将被调用，并且错误信息是 'There was an error.'.
  // 让我们处理这个错误，并返回一个“正确”的新 Promise
  return AV.Promise.as('Hello!');

}).then(function(hello) {
  // 最终处理结果
}, function(error) {
  // 这里不会调用，因为前面已经处理了错误
});
```

通常来说，在正常情况的回调函数链的末尾，加一个错误处理的回调函数，是一种很
常见的做法。

利用 `try,catch` 方法可以将上述代码改写为：

```javascript
var query = new AV.Query('Student');
query.addDescending('gpa');
query.find().try(function(students) {
  students[0].set('valedictorian', true);
  // 强制失败
  return AV.Promise.error('There was an error.');

}).try(function(valedictorian) {
  // 这里的代码将被忽略
  return query.find();

}).try(function(students) {
  // 这里的代码也将被忽略
  students[1].set('salutatorian', true);
  return students[1].save();

}).catch(function(error) {
  // 这个错误处理函数将被调用，并且错误信息是 'There was an error.'.
  // 让我们处理这个错误，并返回一个“正确”的新 Promise
  return AV.Promise.as('Hello!');
}).try(function(hello) {
  // 最终处理结果
}).catch(function(error) {
  // 这里不会调用，因为前面已经处理了错误
});
```

### 创建 Promise

在开始阶段,你可以只用系统（譬如 find 和 save 方法等）返回的 promise。但是，在更高级
的场景下，你可能需要创建自己的 promise。在创建了 Promise 之后，你需要调用 `resolve` 或者 `reject` 来触发它的 callback.

```javascript
var successful = new AV.Promise();
successful.resolve('The good result.');

var failed = new AV.Promise();
failed.reject('An error message.');
```

如果你在创建 promise 的时候就知道它的结果，下面有两个很方便的方法可以使用：

```javascript
var successful = AV.Promise.as('The good result.');

var failed = AV.Promise.error('An error message.');
```

除此之外，你还可以为 `AV.Promise` 提供一个函数，这个函数接收 `resolve` 和 `reject` 方法，运行实际的业务逻辑。例如：

```javascript
var promise = new AV.Promise(function(resolve, reject){
  resolve(42);
});

promise.then(functon(ret){
  //print 42.
  console.log(ret);
});
```

尝试下两个一起用：

```javascript
var promise = new AV.Promise(function(resolve, reject) {
  setTimeout(function() {
    if (Date.now() % 2) {
     resolve('奇数时间');
    } else {
     reject('偶数时间');
    }
  }, 2000);
});

promise.then(function(value) {
  // 奇数时间
  console.log(value);
}, function(value) {
  // 偶数时间
  console.log(value);
});
```

### 顺序的 Promise

在你想要某一行数据做一系列的任务的时候，Promise 链是很方便的，每一个任务都等着前
一个任务结束。比如，假设你想要删除你的博客上的所有评论：

<div class="callout callout-info">下文中在代码里出现的 `_.???` 表示引用了 [underscore.js](http://underscorejs.org/) 这个类库的方法。underscore.js 是一个非常方便的 JS 类库，提供了很多工具方法。</div>

```javascript
var query = new AV.Query('Comment');
query.equalTo('post', post); // 假设 post 是一个已经存在的实例

query.find().then(function(results) {
  // Create a trivial resolved promise as a base case.
  var promise = AV.Promise.as();
  _.each(results, function(result) {
    // For each item, extend the promise with a function to delete it.
    promise = promise.then(function() {
      // Return a promise that will be resolved when the delete is finished.
      return result.destroy();
    });
  });
  return promise;

}).then(function() {
  // Every comment was deleted.
});
```

### 并行的 Promise

你也可以用 Promise 来并行的进行多个任务，这时需要使用 when 方法，你可以一次同时开始几个操作。使用 `AV.Promise.when` 来创建一个新的 promise，它会在所有输入的 `Promise` 被 resolve 之后才被 resolve。即便一些输入的 promise 失败了，其他的 Promise 也会被成功执行。你可以在 callback 的参数部分检查每一个 promise 的结果。并行地进行操作会比顺序进行更快，但是也会消耗更多的系统资源和带宽。

简单例子：

```javascript
function timerPromisefy(delay) {
  return new AV.Promise(function (resolve) {
    //延迟 delay 毫秒，然后调用 resolve
    setTimeout(function () {
      resolve(delay);
    }, delay);
   });
}

var startDate = Date.now();

AV.Promise.when(
  timerPromisefy(1),
  timerPromisefy(32),
  timerPromisefy(64),
  timerPromisefy(128)
).then(function (r1, r2, r3, r4) {
  //r1,r2,r3,r4 分别为1,32,64,128
  //大概耗时在 128 毫秒
  console.log(new Date() - startDate);
});

//尝试下其中一个失败的例子
var startDate = Date.now();
AV.Promise.when(
  timerPromisefy(1),
  timerPromisefy(32),
  AV.Promise.error('test error'),
  timerPromisefy(128)
).then(function () {
  //不会执行
}, function(errors){
  //大概耗时在 128 毫秒
  console.log(new Date() - startDate);
  console.dir(errors);  //print [ , , 'test error',  ]
});
```

下面例子执行一次批量删除某个 Post 的评论：

```javascript
var query = new AV.Query('Comment');
query.equalTo('post', post);  // 假设 post 是一个已经存在的实例

query.find().then(function(results) {
  // Collect one promise for each delete into an array.
  var promises = [];
  _.each(results, function(result) {
    // Start this delete immediately and add its promise to the list.
    promises.push(result.destroy());
  });
  // Return a new promise that is resolved when all of the deletes are finished.
  return AV.Promise.when(promises);

}).then(function() {
  // Every comment was deleted.
});
```

`when` 会在错误处理器中返回所有遇到的错误信息，以数组的形式提供。

除了 `when` 之外，还有一个类似的方法是 `AV.Promise.all`，这个方法和 `when` 的区别在于：

它只接受数组形式的 promise 输入，并且如果有任何一个 promise 失败，它就会直接调用错误处理器，而不是等待所有 promise 完成，其次是它的 resolve 结果返回的是数组。例如：

```javascript
AV.Promise.all([
  timerPromisefy(1),
  timerPromisefy(32),
  timerPromisefy(64),
  timerPromisefy(128)
]).then(function (values) {
  //values 数组为 [1, 32, 64, 128]
});
//测试下失败的例子
AV.Promise.all([
  timerPromisefy(1),
  timerPromisefy(32),
  AV.Promise.error('test error'),
  timerPromisefy(128)
]).then(function () {
  //不会执行
}, function(error){
  console.dir(error);  //print 'test error'
});

//http://jsplay.avosapps.com/zuy/embed?js,console
```

### race 方法

`AV.Promise.race` 方法接收一个 promise 数组输入，当这组 promise 中的任何一个 promise 对象如果变为 resolve 或者 reject 的话， 该函数就会返回，并使用这个 promise 对象的值进行 resolve 或者 reject。`race`，顾名思义就是在这些 promise 赛跑，谁先执行完成，谁就先 resolve。

```javascript
var p1 = AV.Promise.as(1);
var p2 = AV.Promise.as(2);
var p3 = AV.Promise.as(3);
Promise.race([p1, p2, p3]).then(function (value) {
  // 打印 1
  console.log(value);
});
```

### 创建异步方法

有了上面这些工具以后，就很容易创建你自己的异步方法来返回 promise 了。譬如，你可以创建一个有 promise 版本的 setTimeout：

```javascript
var delay = function(millis) {
  var promise = new AV.Promise();
  setTimeout(function() {
    promise.resolve();
  }, millis);
  return promise;
};

delay(100).then(function() {
  // This ran after 100ms!
});
```

### 兼容性

在非 node.js 环境（例如浏览器环境）下，`AV.Promise` 并不兼容 [Promises/A+](https://promisesaplus.com/) 规范，特别是错误处理这块。
如果你想兼容，可以手工启用：

```javascript
AV.Promise.setPromisesAPlusCompliant(true);
```

在 node.js 环境下如果启用兼容 Promises/A+， 可能在一些情况下 promise 抛出的错误无法通过 `process.on('uncaughtException')` 捕捉，你可以启用额外的 debug 日志：

```javascript
AV.Promise.setDebugError(true);
```

默认日志是关闭的。

### JavaScript Promise 迷你书

如果你想更深入地了解和学习 Promise，我们推荐[《JavaScript Promise迷你书（中文版）》](http://liubin.github.io/promises-book/)这本书。
{% endblock %}

{% block js_push_guide %}
## Push 通知

通过 JavaScript SDK 也可以向移动设备推送消息，使用也非常简单。

如果想在 Web 端独立使用推送模块，包括通过 Web 端推送消息到各个设备、以及通过 Web 端也可以接收其他端的推送，可以了解下我们的 [JavaScript 推送 SDK 使用指南](./js_push.html) 来获取更详细的信息。

一个简单例子推送给所有订阅了 `public` 频道的设备：

```javascript
AV.Push.send({
  channels: [ 'Public' ],
  data: {
    alert: 'Public message'
  }
});
```

这就向订阅了 `public` 频道的设备发送了一条内容为 `public message` 的消息。

如果希望按照某个 `_Installation` 表的查询条件来推送，例如推送给某个 `installationId` 的 Android 设备，可以传入一个 `AV.Query` 对象作为 `where` 条件：

```javascript
var query = new AV.Query('_Installation');
query.equalTo('installationId', installationId);
AV.Push.send({
  where: query,
  data: {
    alert: 'Public message'
  }
});
```

此外，如果你觉得 AV.Query 太繁琐，也可以写一句 [CQL](./cql_guide.html) 来搞定：

```javascript
AV.Push.send({
  cql: 'select * from _Installation where installationId="设备id"',
  data: {
    alert: 'Public message'
  }
});
```

`AV.Push` 的更多使用信息参考 API 文档 [AV.Push](/api-docs/javascript/symbols/AV.Push.html)。更多推送的查询条件和格式，请查阅 [消息推送指南](./push_guide.html)。

iOS 设备可以通过 `prod` 属性指定使用测试环境还是生产环境证书：

```javascript
AV.Push.send({
  prod: 'dev',
  data: {
    alert: 'Public message'
  }
});
```

`dev` 表示开发证书，`prod` 表示生产证书，默认生产证书。
{% endblock %}

{% block use_js_in_webview %}
## WebView 中使用

JS SDK 支持在各种 WebView 中使用（包括 PhoneGap/Cordova、微信 WebView 等）。

### Android WebView 中使用

如果是 Android WebView，在 Native 代码创建 WebView 的时候你需要打开几个选项，
这些选项生成 WebView 的时候默认并不会被打开，需要配置：

1. 因为我们 JS SDK 目前使用了 window.localStorage，所以你需要开启 WebView 的 localStorage：

  ```java
  yourWebView.getSettings().setDomStorageEnabled(true);
  ```
2. 如果你希望直接调试手机中的 WebView，也同样需要在生成 WebView 的时候设置远程调试，具体使用方式请参考 [Google 官方文档](https://developer.chrome.com/devtools/docs/remote-debugging)。

  ```java
  if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
      yourWebView.setWebContentsDebuggingEnabled(true);
  }
  ```

  注意：这种调试方式仅支持 Android 4.4 已上版本（含 4.4）
3. 如果你是通过 WebView 来开发界面，Native 调用本地特性的 Hybrid 方式开发你的 App。比较推荐的开发方式是：通过 Chrome 的开发者工具开发界面部分，当界面部分完成，与 Native 再来做数据连调，这种时候才需要用 Remote debugger 方式在手机上直接调试 WebView。这样做会大大节省你开发调试的时间，不然如果界面都通过 Remote debugger 方式开发，可能效率较低。
4. 为了防止通过 JavaScript 反射调用 Java 代码访问 Android 文件系统的安全漏洞，在 Android 4.2 以后的系统中间，WebView 中间只能访问通过 [@JavascriptInterface](http://developer.android.com/reference/android/webkit/JavascriptInterface.html) 标记过的方法。如果你的目标用户覆盖 4.2 以上的机型，请注意加上这个标记，以避免出现 **Uncaught TypeError**。
{% endblock %}



{# --End--主模板留空的代码段落，子模板根据自身实际功能给予实现 #}
