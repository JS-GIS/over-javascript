## 一 SSL初始

### 1.1 SSL简介

传统的HTTP连接中，所有的内容都是以明文形式传输，安全性很低。  

HTTPS即HTTP+SSL，SSL(Secure Sockets Layer 安全套接层)协议以及其继任者TLS(Transport Layer Security 传输层安全)协议是为网络通信提供安全及数据完整性的一种安全协议。TLS与SSL在传输层对网络连接进行加密，SSL的一大优势在于他独立于上层协议，和HTTP结合为HTTPS，和WebSocket结合为WSS.  

### 1.2 SSL原理

不同的SSL握手过程存在差异，分为以下三种：
- 只验证服务器
- 验证服务器和客户端
- 恢复原有会话

这里只介绍第一种过程。  

第1步：  

客户端发送Client Hello消息，该消息包括SSL版本信息，一个随机数（假设它是random1）、一个session id(用来避免后续请求的握手)和浏览器支持的密码套件（cipher suite）。  

cipher suite是由加密算法名称组成的字符串，包括了4种用途的加密算法：
- 密钥交换算法：常用的有RSA,PSK
- 数据加密算法：常用的有AES256,RC4
- 报文认证信息码（MAC）算法：常用的有MD5,SHA
- 伪随机数（PRF）算法

例如下面的字符串就是一个 cipher suite：
```
TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
```

第2步：  

服务器确定本次通信使用的SSL版本和其他信息，发送Server Hello给客户端，里面的内容包括服务器支持的SSL版本信息，一个伪随机数（假设它是random2）、服务器的密码套件（cipher suite）。 

第3步：  
服务器发送CA整数给客户端  

第4步：  
服务器发送Server Hello done  

第5步：  
客户端验证服务器证书的合法性后(Certificate Verify)后，利用证书中的公钥加密premaster secret(一个在堆成加密密钥生成中的46字节随机数字，以及消息认证代码)作为Client Key Exchange的消息发送给服务器。  

第6步：  
SSL客户端发送Change Cipher Spec消息，该消息属于SSL密码变化协议  

第7步：  
客户端计算历史消息的hash值，然后使用服务器公钥加密后发送给服务器，服务器进行同样的操作，然后两个值结果相同表示密钥交换成功  

第8步：  
服务器发送 Change Cipher Spec消息  

第9步：  
服务器计算历史消息的hash值，通过交换后的密钥加密，将其作为finished消息发送给客户端，客户端利用交换后的密钥解密，如果和本地历史消息相同就黄泽宁服务器身份，握手结束

### 1.3 密钥交换步骤

无论加密数据使用的是非对称加密还是对称加密，客户端和服务器都要交换密钥：
- 对称加密：又称为私钥加密或者会话密钥加。客户端需要将解密的密钥发送给服务器。发送方和接收方使用同一个密钥去加密解密数据，速度较快。
- 非对称加密：又称公钥密钥加密。需要把自己的私钥发给对方。使用不同的密钥来分别完成加密和解密操作，一个公开发布，即公钥，一个用户自己保存，即私钥。信息发送者用公钥去加密，信息接收者使用私钥

常见的密钥交换算法是RSA算法，是一种非对称加密算法，步骤如下：
- 步骤1：Client Verify。客户端接收到服务端传来的整数后，先验证该证书合法性，验证通过后取出证书中的服务端公钥，再生成一个随机数random3，再用服务端公钥加密random3生成PreMasterKey
- 步骤2：Clent Key Exchange：将第一步最后生成的key传给服务端，服务端用自己的私钥解出这个key，得到客户端生成的random3，再加上random1，random2，至此，客户端和服务端都拥有了random1，random2，random3.

两边再根据同样的算法就可以生成一份密钥，握手结束后应用层数据都是用该密钥进行对称加密，使用三个随机数的原因是提升暴力破解难度。

### 1.4 CA证书与中间人攻击

上述的步骤就像两个同学上课传纸条，二人为了不让其他人发现纸条的信息，互相约定了写法和破译，各自的加密内容只有自己的解密办法才能破译，那么在传输信息前需要双方交换自己的加方法（公钥）。  

然而在第一次交换加密方法的纸条传递时，中间负责传递的同学把交换双方的公钥都换成了自己伪造的公钥，那么就可以轻松使用自己的私钥读取她们的通信内容，这就是中间人攻击。  

CA是第三方组织，用来验证证书合法性，即在上述案例中，纸条由互相信任的班主任传递。


## 二 Node中使用HTTPS服务

用Node.js创建自签名的HTTPS服务器:
- 创建自己的CA机构
- 创建服务器端证书
- 创建客户端证书
- 将证书打包

创建自己的CA机构:  
```
# 为CA生成私钥
openssl genrsa -out ca-key.pem -des 1024

# 通过CA私钥生成CSR
openssl req -new -key ca-key.pem -out ca-csr.pem

# 通过CSR文件和私钥生成CA证书
openssl x509 -req -in ca-csr.pem -signkey ca-key.pem -out ca-cert.pem
```

可能会遇到的问题:
```
你需要root或者admin的权限
Unable to load config info from /user/local/ssl/openssl.cnf 

解决：从网上下载一份正确的openssl.cnf文件，然后set OPENSSL_CONF=openssl.cnf文件的本地路径 
```

创建服务器端证书:
```
# 为服务器生成私钥
openssl genrsa -out server-key.pem 1024

# 利用服务器私钥文件服务器生成CSR
openssl req -new -key server-key.pem -config openssl.cnf -out server-csr.pem

# 这一步非常关键，你需要指定一份openssl.cnf文件。可以用这个
[req]  
    distinguished_name = req_distinguished_name  
    req_extensions = v3_req  
  
    [req_distinguished_name]  
    countryName = Country Name (2 letter code)  
    countryName_default = CN  
    stateOrProvinceName = State or Province Name (full name)  
    stateOrProvinceName_default = BeiJing  
    localityName = Locality Name (eg, city)  
    localityName_default = YaYunCun  
    organizationalUnitName  = Organizational Unit Name (eg, section)  
    organizationalUnitName_default  = Domain Control Validated  
    commonName = Internet Widgits Ltd  
    commonName_max  = 64  
  
    [ v3_req ]  
    # Extensions to add to a certificate request  
    basicConstraints = CA:FALSE  
    keyUsage = nonRepudiation, digitalSignature, keyEncipherment  
    subjectAltName = @alt_names  
  
    [alt_names]  
	#注意这个IP.1的设置，IP地址需要和你的服务器的监听地址一样
    IP.1 = 127.0.0.1

# 通过服务器私钥文件和CSR文件生成服务器证书
openssl x509 -req -CA ca-cert.pem -CAkey ca-key.pem -CAcreateserial -in server-csr.pem -out server-cert.pem -extensions v3_req -extfile openssl.cnf
```

创建客户端证书：
```
# 生成客户端私钥
openssl genrsa -out client-key.pem

# 利用私钥生成CSR
openssl req -new -key client-key.pem -out client-csr.pem

# 生成客户端证书
openssl x509 -req -CA ca-cert.pem -CAkey ca-key.pem -CAcreateserial -in client-csr.pem -out client-cert.pem
```

HTTPS 服务器代码:
```js
let https = require('https');
let fs = require('fs');

let options = {
	key: fs.readFileSync('./keys/server-key.pem'),
	ca: [fs.readFileSync('./keys/ca-cert.pem')],
	cert: fs.readFileSync('./keys/server-cert.pem')
};

https.createServer(options,function(req,res){
	res.writeHead(200);
	res.end('hello world\n');
}).listen(3000,'127.0.0.1');
```

HTTPS 客户端代码:
```js
let https = require('https');
let fs = require('fs');

let options = {
	hostname:'127.0.0.1',
	port:3000,
	path:'/',
	method:'GET',
	key:fs.readFileSync('./keys/client-key.pem'),
	cert:fs.readFileSync('./keys/client-cert.pem'),
	ca: [fs.readFileSync('./keys/ca-cert.pem')],
	agent:false
};

options.agent = new https.Agent(options);
let req = https.request(options,function(res){
console.log("statusCode: ", res.statusCode);
  console.log("headers: ", res.headers);
	res.setEncoding('utf-8');
	res.on('data',function(d){
		console.log(d);
	})
});

req.end();

req.on('error',function(e){
	console.log(e);
})
```

将证书打包
```
# 打包服务器端证书
openssl pkcs12 -export -in server-cert.pem -inkey server-key.pem -certfile ca-cert.pem -out server.pfx

# 打包客户端证书
openssl pkcs12 -export -in client-cert.pem -inkey client-key.pem -certfile ca-cert.pem -out client.pfx
```

服务器端代码:
```js
let https = require('https');
let fs = require('fs');

let options = {
	pfx:fs.readFileSync('./keys/server.pfx'),
	passphrase:'your password'
};

https.createServer(options,function(req,res){
	res.writeHead(200);
	res.end('hello world\n');
}).listen(3000,'127.0.0.1');
```

客户端代码:
```js
let https = require('https');
let fs = require('fs');

let options = {
	hostname:'127.0.0.1',
	port:3000,
	path:'/',
	method:'GET',
	pfx:fs.readFileSync('./keys/server.pfx'),
	passphrase:'your password',
	agent:false
};

options.agent = new https.Agent(options);
let req = https.request(options,function(res){
console.log("statusCode: ", res.statusCode);
  console.log("headers: ", res.headers);
	res.setEncoding('utf-8');
	res.on('data',function(d){
		console.log(d);
	})
});

req.end();

req.on('error',function(e){
	console.log(e);
})
```
## 三 HTTPS缺点

比http连接要慢，因为额外需要SSL验证。