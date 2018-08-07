# Play 入门教程 · JavaScript

欢迎使用 LeanCloud Play。本教程将通过模拟一个比较玩家分数大小的场景，来讲解 Play SDK 的核心使用方法。


## 安装

Play 客户端 SDK 是开源的，源码地址请访问 [Play-SDK-JS](https://github.com/leancloud/Play-SDK-JS)。也可以直接下载 [Release 版本]((https://github.com/leancloud/Play-SDK-JS/releases)。

### Cocos Creator

也适用于 Cocos Creator 导出的微信小游戏。下载 `play.js` 并拖拽至 Cocos Creator 项目中即可。**注意不要选择「插件方式」**。

### 微信小程序

下载 `play-weapp.js` 并拖拽至微信小程序的工程目录下即可。

### Node.js 安装

```
npm install @leancloud/play --save。
```


## 初始化

导入需要的类和变量

```javascript
import {
  play,
  Region,
  Event,
} from '../play';
```
其中 `play` 是 SDK 实例化并导出的 Play 的对象，并不是 Play 类。

```javascript
const opts = {
  // 设置 APP ID
  appId: YOUR_APP_ID,
  // 设置 APP Key
  appKey: YOUR_APP_KEY,
  // 设置节点地区
  // EastChina：华东节点
  // NorthChina：华北节点
  // NorthAmerica：美国节点
  region: Region.EastChina,
}
play.init(opts);
```


## 设置玩家 ID

```javascript
// 这里使用随机数作为 userId
const randId = parseInt(Math.random() * 1000000, 10);
play.userId = randId.toString();
```


## 连接至 Play 服务器

```javascript
play.connect();
```

连接完成后，会通过 `CONNECTED`（连接成功）或 `CONNECT_FAILED`（连接失败）事件来通知客户端。


## 创建或加入房间

默认情况下 Play SDK 会在连接成功后自动加入大厅；玩家在大厅中，创建 / 加入指定房间。

```javascript
// 注册连接成功事件
play.on(Event.CONNECTED, () => {
  console.log('on joined lobby');
  const roomName = 'cocos_creator_room';
  play.joinOrCreateRoom(roomName);
});
```

`joinOrCreateRoom` 通过相同的 roomName 保证两个客户端玩家可以进入到相同的房间。请参考 [开发指南](play-js.html#创建房间) 获取更多关于 `joinOrCreateRoom` 的用法。


## 通过 CustomPlayerProperties 同步玩家属性

当有新玩家加入房间时，Master 为每个玩家分配一个分数，这个分数通过「玩家自定义属性」同步给玩家。
（这里没有做更复杂的算法，只是为 Master 分配了 10 分，其他玩家分配了 5 分）。

```javascript
// 注册新玩家加入房间事件
play.on(Event.PLAYER_ROOM_JOINED, (data) => {
  const { newPlayer } = data;
  console.log(`new player: ${newPlayer.userId}`);
  if (play.player.isMaster()) {
    // 获取房间内玩家列表
    const playerList = play.room.playerList;
    for (let i = 0; i < playerList.length; i++) {
      const player = playerList[i];
      // 判断如果是房主，则设置 10 分，否则设置 5 分
      if (player.isMaster()) {
        player.setCustomProperties({
          point: 10,
        });
      } else {
        player.setCustomProperties({
          point: 5,
        });
      }
    }
    // ...
  }
});
```

玩家得到分数后，显示自己的分数。

```javascript
// 注册「玩家属性变更」事件
play.on(Event.PLAYER_CUSTOM_PROPERTIES_CHANGED, data => {
  const { player } = data;
  // 解构得到玩家的分数
  const { point } = player.getCustomProperties();
  console.log(`${player.userId}: ${point}`);
  if (player.isLocal()) {
    // 判断如果玩家是自己，则做 UI 显示
    this.scoreLabel.string = `score:${point}`;
  }
});
```


## 通过「自定义事件」通信

当分配完分数后，将获胜者（Master）的 ID 作为参数，通过自定义事件发送给所有玩家。

```javascript
if (play.player.isMaster()) {
  play.sendEvent('win', 
    { winnerId: play.room.masterId }, 
    { receiverGroup: ReceiverGroup.All });
}
```

根据判断胜利者是不是自己，做不同的 UI 显示。

```javascript
// 注册自定义事件
play.on(Event.CUSTOM_EVENT, event => {
  // 解构事件参数
  const { eventId, eventData } = event;
  if (eventId === 'win') {
    // 解构得到胜利者 Id
    const { winnerId } = eventData;
    console.log(`winnerId: ${winnerId}`);
    // 如果胜利者是自己，则显示胜利 UI；否则显示失败 UI
    if (play.player.actorId === winnerId) {
      this.resultLabel.string = 'win';
    } else {
      this.resultLabel.string = 'lose';
    }
    play.disconnect();
  }
});
```


## Demo

我们通过 Cocos Creator 完成了这个 Demo，供大家运行参考。

[QuickStart 工程](https://github.com/leancloud/Play-Quick-Start-JS)。


## 构建注意事项

您可以通过 Cocos Creator 构建出其支持的工程，目前 SDK 支持的平台包括：Mac、Web、微信小游戏、FaceBook Instant Game、iOS、Android。

其中仅在构建 Android 工程时需要做一点额外的配置，需要在初始化 `play` 之前添加如下代码：

```js
onLoad() {
  const { setAdapters } = Play;
  if (cc.sys.platform === cc.sys.ANDROID) {
    const caPath = cc.url.raw('resources/cacert.pem');
    setAdapters({
      WebSocket: (url) => new WebSocket(url, null, caPath)
    });
  }
}
```

这样做的原因是 Play SDK 使用了基于 WebSocket 的 wss 进行安全通信，需要通过以上代码适配 Android 平台的 CA 证书机制。





