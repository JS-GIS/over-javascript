## 一 RabbitMQ简介
RabbitMQ可以用于实现大并发时的消息队列，同时处理不同的web服务器之间通信需求。  
所谓队列，就是抢占资源时的排队，消息就是不同服务器见传送的数据单位，可以是json、文本等。  
使用消息队列的优势是：
- 1-应用解耦：消息对垒在处理项目需求时，在中间插入了隐含的基于数据的接口层，两边的处理过程都要实现这一接口。这允许你独立的扩展活修改两边的处理过程，只要遵守同样的接口约定即可。
- 2-可扩展性：可以方便的实现大消息入队和处理的频率
- 3-灵活性与峰值处理能力：消息队列可以应对临时的突发高峰值访问
- 4-缓冲提升性能
- 5-异步通信：很多消息不需要立即处理，消息队列可以提供异步处理机制。
RabbitMQ的安装：
```
docker安装：
sudo docker pull rabbitmq
docker run -d -e RABBITMQ_NODENAME=my-rabbit --name some-rabbit -p 5672:5672 rabbitmq:3

mac安装：
    brew install rabbitmq
    vim ~/.bash_profile    //添加： export PATH=$PATH:/usr/local/sbin
    source ~/.bash_profile
    rabbitmq-server       // 重新打开终端输入
    浏览器输入：http://localhost:15672/     //账号密码全输入guest即可登录

    如果显示找不到主机，请在hosts文件中添加:
    vim /private/etc/hosts
    127.0.0.1  localhost

```
## 二 HelloWorld
server.js
```js
const amqp = require("amqplib");

//连接本地127.0.0.1的rabbitmq队列，返回一个promise对象
const connect = amqp.connect("amqp://127.0.0.1");

connect.then((conn)=>{

    //接收到CTRL+C时关闭连接
    process.once("SIGINT", ()=>{
        conn.close;
    });

    //创建一个通道
    let channel =  conn.createChannel();

    channel.then((ch)=>{

        //监听 hello 队列，并设置durable持久化为false，表示队列保存在内存中
        let ok = ch.assertQueue("hello",{durable: false});

        ok = ok.then((_qok)=>{
            
            //让通道消费hello队列，
            return ch.consume("hello", (msg)=>{

                console.log("Received:",msg.content.toString());

            }, {noAck: true});

        });

        //消费成功后，打印一行文本，表示服务器正常工作，等待客户端数据
        return ok.then((_consumeOK)=>{
            console.log("Waiting for msg,press CTRL+C exit!");
        });

    });

}).then(null, console.warn);
```
client.js
```js
const amqp = require("amqplib");

const connect = amqp.connect("amqp://127.0.0.1");

connect.then((conn)=>{

    let channel = conn.createChannel();
        
    channel.then((ch)=>{

        let q = "hello";
        let msg = "hello world";

        let ok = ch.assertQueue(q, {durable: false});

        return ok.then((_qok)=>{
            ch.sendToQueue(q, new Buffer(msg));
            return ch.close();
        });
    
    });


}).then(null, console.warn);
```
## 三 RabbitMQ工作队列
一个生产者配合多个消费者的队列：可以让队列不至于堆积过长，同时也能保证队列的响应时间。  
下列代码使用传统的callback模式：
server.js:
```js
const amqp = require("amqplib/callback_api");

function errHandle(err, conn){
    console.error("err=",err);
    if (conn) {
        conn.close(()=>{
            process.exit(1);
        });
    }
}

function work(msg) {
    let body = msg.content.toString();
    console.log("Receive:", body);
    let secs = body.split(".").length - 1;
    setTimeout(()=>{
        console.log("Done...");
    }, secs * 1000);
}

amqp.connect("amqp://127.0.0.1", (err, conn)=>{

    if (err) {
        return errHandle(err);
    }

    process.once("SIGINT", ()=>{
        conn.close();
    });

    let q = "task_queue";

    conn.createChannel((err, ch)=>{

        if (err) {
            return errHandle(err, conn);
        }

        ch.assertQueue(q, {durable: true}, (err, _ok)=>{

            if (err) {
                return errHandle(err, conn);
            }
            
            ch.consume(q, work, {noAck:false});

            console.log("Waiting for ms,press CTRL+C exit!");

        });

    });


});
```
client.js:
```js
const amqp = require("amqplib/callback_api");

function errHandle(err, conn){
    console.error("err=",err);
    if (conn) {
        conn.close(()=>{
            process.exit(1);
        });
    }
}

amqp.connect("amqp://127.0.0.1", (err, conn)=>{

    if (err) {
        return errHandle(err);
    }

    let q = "task_queue";

    conn.createChannel((err, ch)=>{

        if (err) {
            return errHandle(err, conn);
        }

        let msg = process.argv.slice(2).join(' ') || "Hello World!";

        ch.sendToQueue(q, new Buffer(msg), {persistent: true});

        console.log("Send:", msg);

        ch.close(()=>{
            conn.close();
        });

    });

});
```
启动两个服务端同时监听客户端消息，然后不停的启动客户端发送消息：
```
node client.js first message
node client.js second message
node client.js third message
```
此时我们会发现，任务被平均分配给了2个消费者来处理了。任务量如果增大，可以通过增加消费者来增强队列的计算能力。
## 三 RabbitMQ的PUB/SUB队列
在上一节中，一个复杂的任务轮询分配给了多个消费者来平均处理，这样可以减轻系统负载。但是也有这样的需求，一个消息同时又多个消费者都在订阅，此时需要使用PUB/SUB队列。比如实际的案例：接收到一个消息后，消费者C1负责记录日志，消费者C2负责打印日志。RabbitMQ的核心思想就是每个生产者都不直接将消息丢入队列中，甚至生产者根本不知道这个消息将会被宋到哪个或者哪几个消费者手中，这样的解耦思想有利于整个系统。  
在这样的系统中，生产者只是将消息传递给Exchange，Exchange一边连接生产者接收生产者发送过来的数据，一边连接着队列，负责把数据推送到队列中。Exchange必须知道它将对接收到的数据进行怎样的处理，如将数据推送到指定的队列，或者推送到多条队列中等等，我们称这样的行为为Exchange类型，常见的类型是：direct、topic、headers、fanout。  
案例采用fanout类型，即广播消息出去。
server.js:
```js
const amqp = require("amqplib/callback_api");

function errHandle(err, conn){
    console.error("err=",err);
    if (conn) {
        conn.close(()=>{
            process.exit(1);
        });
    }
}

amqp.connect("amqp://127.0.0.1", (err, conn)=>{

    if (err) {
        return errHandle(err);
    }

    process.once("SIGINT", ()=>{
        conn.close();
    });

    let ex = "test_pub";

    conn.createChannel((err, ch)=>{

        if (err) {
            return errHandle(err, conn);
        }

        ch.assertExchange(ex, "fanout", {durable: false}, (err)=>{

            if (err) {
                return errHandle(err, conn);
            }

            //这里不再给队列命名，exclusive表示断开连接时，删除队列
            ch.assertQueue('', {exclusive: true}, (err, ok)=>{

                //获取队列对象
                let q = ok.queue;       

                //绑定队列与Exchange，参数三为路由，暂时设置为空
                ch.bindQueue(q, ex, '');

                //开始消费数据，并设置消费完成后的回调函数
                ch.consume(
                    q, 
                    msg=>{
                        if (msg) {
                            console.log("Receive:", msg.content.toString())
                        }
                    }, 
                    {noAck: true},
                    (err, ok)=>{
                        if (err) {
                            return errHandle(err, conn);
                        }
                        console.log("Wating,press Ctrl+C exit!");
                    }
                );

            });

        });

    });


});
```
client.js:
```js
const amqp = require("amqplib/callback_api");

function errHandle(err, conn){
    console.error("err=",err);
    if (conn) {
        conn.close(()=>{
            process.exit(1);
        });
    }
}

amqp.connect("amqp://127.0.0.1", (err, conn)=>{

    if (err) {
        return errHandle(err);
    }

    let ex = "test_pub";

    conn.createChannel((err, ch)=>{

        if (err) {
            return errHandle(err, conn);
        }

        let msg = process.argv.slice(2).join(" ") || "info: Hello world!";

        //不再把消息直接推送给队列，而是推送给Exchange
        ch.assertExchange(ex, "fanout", {durable: false} );

        ch.publish(ex, '', new Buffer(msg));

        console.log("Send:", msg);

        ch.close(()=>{
            conn.close();
        });

    });

});
```
测试：启动多个server，然后不断启动client，会发现多个server端都接收到了数据。
## 四 RabbitMQ的队列路由
在第三章节的案例中，消息被多个消费者订阅，实际上还有更深入的需求：消费者是多种多样的，有队日志err专门分析的消费者，有队日志做打印的专门的消费者，这些消费者要根据自己的需要去获取队列里的数据，这时需要用到Exchange的路由功能。上述案例中的fanout只能无脑广播，这里需要修改类型为direct。direct类型的Exchange会把消息推送到绑定这个路由key的队列中去，简单的说，就是生产者将消息和路由的key推送给Exchange，Exchange则个悲剧哪个或者哪几个消费队列绑定了这个路由key，把消息再推送到这些队列中，供消费者消费。
server.js
```js
const amqp = require("amqplib/callback_api");
const path = require("path");

let basename = path.basename;
let serverities = process.argv.slice(2);   
if (serverities.length < 1) {
    console.log("Usage %s [info] [warning] [error]", basename(process.argv[1]));
    process.exit(1);
}

function errHandle(err, conn){
    console.error("err=",err);
    if (conn) {
        conn.close(()=>{
            process.exit(1);
        });
    }
}

amqp.connect("amqp://127.0.0.1", (err, conn)=>{

    if (err) return errHandle(err);

    process.once("SIGINT", ()=>{
        conn.close();
    });

    conn.createChannel((err, ch)=>{

        if (err) return errHandle(err, conn);

        let ex = "test_direct";

        ch.assertExchange(ex, "direct", {durable: false});

        ch.assertQueue('', {exclusive: true}, (err, ok)=>{

            if (err) return errHandle(err, conn); 

            let q = ok.queue;  
            let i = 0;  
            
            //定义绑定队列的递归函数
            function sub(err) {
                if (err) return errHandle(err, conn);
                while (i < serverities.length) {
                    ch.bindQueue(q, ex, serverities[i], {}, sub);
                    i++;
                }
            }

            ch.consume(
                q, 
                msg=>{
                    if (msg) {
                        console.log("Receive:", msg.content.toString())
                    }
                }, 
                {noAck: true},
                (err)=>{
                    if (err) {
                        return errHandle(err, conn);
                    }
                    sub(null);
                }
            );

        });
    })
})
```
client.js
```js
const amqp = require("amqplib/callback_api");

function errHandle(err, conn){
    console.error("err=",err);
    if (conn) {
        conn.close(()=>{
            process.exit(1);
        });
    }
}

amqp.connect("amqp://127.0.0.1", (err, conn)=>{

    if (err) {
        return errHandle(err);
    }

    conn.createChannel((err, ch)=>{

        if (err) {
            return errHandle(err, conn);
        }

        //获取参数，这里生产者发送日志的级别，默认info
        let args = process.argv.slice(2);
        let severity = (args.length > 0 ) ? args[0] : "info";
        let msg = args.slice(1).join(" ") || "Hello world!";

        //将Exchange节点命名为direct_logs
        let ex = "test_direct";
    
        //不再把消息直接推送给队列，而是推送给Exchange
        ch.assertExchange(ex, "direct", {durable: false}, (err, ok)=>{

            if (err) return errHandle(err, conn);

            //向节点ex推送数据，并带上key变量severity
            ch.publish(ex, severity, new Buffer(msg));

            console.log("Send:", msg);

            ch.close(()=>{
                conn.close();
            });

        } );

    });

});
```
测试：
```
启动一个监听 info  warning error 的消费者：
node server.js info waring error
再启动一个只监听info的消费者：
node server.js info

客户端测试：
node client.js error "只发送了错误error"
node client.js info "发送了info"
```
## 五 RabbitMQ的RPC远程调用
RabbitMQ还可以提供类似RPC远程调用的功能。当然跨语言通信常用的方式是基于HTTP的RESTful，或者性能更好的Thrift。  
server.js:
```js
const amqp = require('amqplib/callback_api');

//定义斐波那契数列，模拟耗时运算
function fib(n) {

    let a = 0;
    let b = 1;

    for (let i = 0; i < n; i++) {
        let c = a + b;
        a = b;
        b = c;
    }

    return a;

}

function errHandle(err, conn){
    console.error("err=",err);
    if (conn) {
        conn.close(()=>{
            process.exit(1);
        });
    }
}

amqp.connect("amqp://localhost", (err, conn)=>{

    if (err) return errHandle(err);

    process.once("SIGINT", ()=>{
        conn.close();
    });

    let q = "rpc_queue";

    conn.createChannel((err, ch)=>{

        ch.assertQueue(q, {durable: false});
         //设置公平调度：RabbitMQ会将奇数消息推送给A，偶数推送给B，如果奇数消息数目较多，不利于负载，设置为1不回向繁忙消费者推送
        ch.prefetch(1);    

        ch.consume(q, reply, {noAck: false}, (err)=>{

            if (err) return errHandle(err);

            console.log("Waiting rpc req");

        });

        function reply(msg) {

            let n = parseInt(msg.content.toString());
            console.log('[.].fib(%d)', n);
            ch.sendToQueue(
                msg.properties.replyTo, 
                new Buffer(fib(n).toString()),
                {correlationId: msg.properties.correlationId}
            );
            ch.ack(msg);
        }

    });

});
```
client.js:
```js
const amqp = require("amqplib/callback_api");
const uuid = require("node-uuid");

let basename = require("path").basename;

let n;
try {
    if (process.argv.length < 3) {
        throw Error("Too few args");
    }
    n = parseInt(process.argv[2]);
} catch (e) {
    console.error(e);
    process.exit(1);
}

function errHandle(err, conn){
    console.error("err=",err);
    if (conn) {
        conn.close(()=>{
            process.exit(1);
        });
    }
}

amqp.connect("amqp://127.0.0.1", (err, conn)=>{

    if (err) return errHandle(err);

    conn.createChannel((err, ch)=>{

        if (err) return errHandle(err, conn);

        let correlationId = uuid();

        function maybeAnswer(msg) {
            if ( msg.properties.correlationId === correlationId ) {
                console.log('[.] Got %d', msg.content.toString());
            } else {
                return errHandle(new Error("Unexprected msg"));
            }
            ch.close(()=>{
                conn.close();
            });
        }

        ch.assertQueue('', {exclusive: true}, (err, ok)=>{

            if (err) return errHandle(err);

            let q = ok.queue;

            ch.consume(q, maybeAnswer, {noAck:true});

            ch.sendToQueue(
                'rpc_queue', 
                new Buffer(n.toString()),
                {replyTo: q, correlationId: correlationId}
            );

        });

    });

});
```
测试：
```
启动多个服务端，然后客户端多次启动：
node client.js  37
node client.js  30
node client.js  38
```
## 六 基于RBMQ实现Node与Python通信案例
本案例以Node为生产者，通过RabbitMQ消息队列发送一个Hello World给Python消费者，打印该信息。
send.js：
```js
const amqp = require("amqplib/callback_api");

function errHandle(err, conn){
    console.error("err=",err);
    if (conn) {
        conn.close(()=>{
            process.exit(1);
        });
    }
}

amqp.connect("amqp://127.0.0.1", (err, conn)=>{

    if (err) return errHandle(err);

    let q = "hello";
    let msg = "Hello world";

    conn.createChannel((err, ch)=>{

        if (err) return errHandle(err, conn);

        ch.assertQueue(q, {durable: false}, (err, ok)=>{

            if (err) return errHandle(err);

            ch.sendToQueue(
                q, 
                new Buffer(n.toString())
            );

            ch.close(()=>{
                conn.close();
            });

        });

    });

});
```
receive.py:
```python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters(host='localhost'));

channel = connection.channel()

channel.queue_declare(queue=" hello")

print 'Wating msg ,press CTRL+C exit'

def callback(ch, method, properties, body):
    print " Received %r" % (body,)
channel.basic_consume(
    callback,
    queue=' hello',
    no_ack=True
)
```
## 七 RBMQ与HTTP方案应对峰值对比
HTTP方式不容易控制请求频率，比如一个活动，每小时只允许100个用户获奖需求，使用RBMQ则可以将所有的用户进行排队。但是经过测试RBMQ的性能稍差。

