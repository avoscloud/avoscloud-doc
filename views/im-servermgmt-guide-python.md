# 即时通讯服务端管理开发指南

Python 的数据存储 SDK，基于 REST API 封装了一组对话及消息管理的接口。这部分接口主要用来在服务器或者云引擎中对[即时通讯](realtime_v2.html)的对话或者消息进行管理。

SDK 的安装及初始化请参考[安装指南](sdk_setup-python.html)。

## 对话管理

`leancloud.Conversation` 对应即时通讯中的[对话概念](realtime_v2.html#对话（Conversation）)，同时也是 Python SDK 中 `leancloud.Object` 的子类，因此可以像正常的 `leancloud.Object` 那样来创建、查询对话。

```python
conv = leancloud.Conversation()
conv.save()

same_conv = leancloud.Conversation.query.get(conv.id)
```

`leancloud.Conversation` 的查询与修改等操作，也受限于 LeanCloud 存储服务的 Class 权限设置与 ACL 权限设置。因此创建对话时，请确保当前权限设置正确，以免造成数据泄漏。

### 对话属性

`leancloud.Conversation` 上有一些额外的属性，代表对话上的属性：

| 属性名 | 属性说明 |
| - | - |
| name    |  此对话的名称 |
| creator    |  此对话的创建者，对应表中 'c' 字段 |
| last_message_read_at    |  此对话最后一条已读消息，对应表中 'lm' 字段 |
| members    |  包含此对话中，所有用户的 client id 的 list，对应表中 'm' 字段 |
| muted_members    |  包含此对话中，所有将对话设置为静音的用户的 client id 的 list，对应表中 'mu' 字段 |
| is_system    |  是否为系统对话 |
| is_transient    |  是否为暂态对话 |


### 创建对话

有些时候需要在服务端来进行对话创建，可以把 `leancloud.Conversation` 当作一个 `leancloud.Object` 来直接创建并保存就可以。

```python
conv = leancloud.Conversation('testConversation')
conv.set('chatRoomNumber', 233)
conv.save()
```

创建对话时，指定 `is_system` 或 `is_transient` 参数为真，可以创建系统对话或暂态对话。

```python
sys_conv = leancloud.Conversation(is_system=True)

tr_conv = leancloud.Conversation(is_transient=True)
```

Python SDK 暂不支持创建聊天室。

### 添加用户到对话

调用 `leancloud.Conversation` 上的 `add_member` 方法，可以将一个用户添加到此对话上来。需要注意的是，后面的参数应该是即时通讯的 [client ID](realtime_v2.html#ClientID、用户和登录)，而不是 `leancloud.User` 实例。如果项目使用 `leancloud.User` 作为用户系统，而没有使用自己的用户系统，可以直接使用 `leancloud.User` 的 `id` 作为 client ID。

```python
conv.add_member("Tom")
```

## 消息管理

### 发送消息

#### 普通消息

可以使用任意一个 client ID，在某个对话中发送消息。发送消息时，必须使用 master key 权限进行操作。

```python
conv.send("Tom", "Hello, World!")
```

#### 系统消息

可以在一个系统对话中，给指定的一组用户发送消息，这时只有这部分用户会收到消息。

```python
client_ids = ["Tom", "Jerry", "Spike"]  # 最多同时给 20 个用户发送消息
sys_conv.send("Mammy Two Shoes", "安静！安静！", to_clients=client_ids)
```

#### 发送广播消息

可以在一个系统对话中，向系统中的所有用户广播消息。
目前广播消息只能在系统对话上调用。
系统中的所有用户都会收到此消息，不论是否在此会话中。

```python
sys_conv.broadcast("Mammy Two Shoes", "周六有晚会！")
```

#### 离线推送消息

`send` 和 `broadcast` 方法有一个可选的 `push_data` 参数，用来指定消息的[离线推送通知](realtime-guide-intermediate.html#离线推送通知)。

#### 文本之外的聊天消息

上面的例子中，消息都是一个简单的字符串（文本消息）。如果需要发送文本之外的消息（图像y音频、视频、文件、地理位置等），需要使用符合[指定格式]的 JSON 字符串。也可以传入一个 Python 字典（SDK 会将其自动序列化为 JSON 字符串）。

[指定格式]: https://leancloud.cn/docs/realtime_rest_api.html#hash2100394145

### 历史消息查询

#### 查询对话历史消息

可以查询某个对话中的历史消息。

```python
messages = leancloud.Message.find_by_conversation(conv.id)
```

可选参数：

- `limit`：指定消息返回数量，服务端默认为 100 条，最大 1000 条。
- `reversed`：以默认排序（由新到旧）相反的方向返回结果，服务端默认为 False。

#### 查询用户历史消息

可以根据 Client ID 查询历史消息。

```python
messages = leancloud.Message.find_by_client("Tom", limit="50")
```

其中，可选参数 `limit` 指定消息返回数量，服务端默认为 100 条，最大 1000 条。

#### 查询全部历史消息

可以直接查询应用中所有的历史消息。

```python
import leancloud

messages = find_by_client()
```

#### 更多功能

目前 Python SDK 只封装了即时通讯 REST API 的部分接口，缺少的功能目前需要通过直接调用 [REST API] 实现（例如，通过 Python 标准库的 `http.client` 调用）。

[REST API]: https://leancloud.cn/docs/realtime_rest_api_v2.html