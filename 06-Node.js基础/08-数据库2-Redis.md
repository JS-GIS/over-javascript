## 二 koa与消息队列

一般来说，消息队列有两种场景：
- 发布订阅模式：发布者向某个频道发布一条消息后，多个订阅者都会收到同一个消息
- 生产消费模式：生产者生产消息放到队列里，消费者同时监听队列，如果队列里有了新的消息就将其取走，对于单条消息，只能由一个消费者消费

发布订阅模式：
```js
// publisher
let redis = require("redis");
let client = redis.createClient(6379, "127.0.0.1");
client.on("error", function(err){
    console.log(err);
});
client.on("ready", function(){
    client.publish("channel1", "hello");
});

//subsciber
let redis = require("redis");
let client = redis.createClient(6379, "127.0.0.1");
client.on("error", function(err){
    console.log(err);
});
client.subscribe("channel1");
client.on("message", function(channel, message){
    console.log("channel: ", channel);
    console.log("message: ", message);
});
```