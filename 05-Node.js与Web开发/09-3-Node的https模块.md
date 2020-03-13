## 一 Node中的TCP使用TLS服务

### 1.1 服务端

Node的tls模块可以创建一个安全的TCP服务，该服务只是一个简单echo服务：
```js
var tls = require('tls');
var fs = require('fs');

var options = {
	key: fs.readFileSync('./keys/server.key'),
	cert: fs.readFileSync('./keys/server.crt'),
	requestCert: true,
	ca: [ fs.readFileSync('./keys/ca.crt') ]
};

var server = tls.createServer(options, function (stream) {
	console.log('server connected', stream.authorized ? 'authorized' : 'unauthorized');
	stream.write("welcome!\n");
	stream.setEncoding('utf8');
	stream.pipe(stream);
});

server.listen(8000, function() {
	console.log('server bound');
});
```

启动后测试证书：
```
openssl s_client -connect 127.0.0.1:8000
```

### 1.2 客户端

在构建客户端之前，需要为客户端生成属于自己的私钥和签名：
```
# 创建私钥
openssl genrsa -out client.key 1024

# 生成CSR
openssl req -new -key client.key -out client.csr

# 生成签名证书
openssl x509 -req -CA ca.crt -CAkey ca.key -CAcreateserial -in client.csr -out client.crt
```

构建客户端：
```js
var tls = require('tls');
var fs = require('fs');

var options = {
	key: fs.readFileSync('./keys/client.key'),
	cert: fs.readFileSync('./keys/client.crt'),
	ca: [ fs.readFileSync('./keys/ca.crt') ]
};

var stream = tls.connect(8000, options, function () {
	console.log('client connected', stream.authorized ? 'authorized' : 'unauthorized');
	process.stdin.pipe(stream);
});

stream.setEncoding('utf8');

stream.on('data', function(data) {
	console.log(data);
});

stream.on('end', function() {
	server.close();
});
```

## 二 Node中使用HTTPS服务

### 2.1 服务端

HTTPS 服务器代码:
```js
var https = require('https');
var fs = require('fs');

var options = {
	key: fs.readFileSync('./keys/server.key'),
	cert: fs.readFileSync('./keys/server.crt')
};

https.createServer(options, function (req, res) {
	res.writeHead(200);
	res.end("hello world\n");
}).listen(8000);
```

启动后通过curl进行测试：
```
curl https://localhost:8000/
```

此时会爆出错误警告，因为curl工具无法验证服务端证书是否正确，解决方案：
- 加 -k 选项，忽略证书验证，这样仍然通过公钥加密传输，但是无法保证对方是否可靠，存在中间人攻击风险
- 使用：`curl --cacert keys/ca.crt https://localhost:8000/`

### 2.2 客户端

```js
var https = require('https');
var fs = require('fs');

var options = {
	hostname: 'localhost',
	port: 8000,
	path: '/',
	method: 'GET',
	key: fs.readFileSync('./keys/client.key'),
	cert: fs.readFileSync('./keys/client.crt'),
	ca: [fs.readFileSync('./keys/ca.crt')]
};

options.agent = new https.Agent(options);

var req = https.request(options, function(res) {
	res.setEncoding('utf-8');
	res.on('data', function(d) {
		console.log(d);
	});
});

req.end();

req.on('error', function(e) {
	console.log(e);
});
```

如果不设置ca选项，则会得到异常：`Error: UNABLE_TO_VERIFY_LEAF_SIGNATURE`。解决方案是添加选项属性：rejectUnauthorized为false，其效果与curl工具添加 -k 一致。