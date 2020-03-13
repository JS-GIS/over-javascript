## 一 Web存储
web存储即把数据存储在浏览器中。传统方式我们以document.cookie来进行存储的，但是由于其存储大小只有4k左右，并且解析也相当的复杂，HTML5规范则提出解决方案：Storage 存储（WebSQL、IndexDB已经被w3c 放弃了）。分别有两种存储方式：
```
会话存储：
    设置：window.sessionStorage.setItem("name","lisi");
    获取：window.sessionStorage.getItem("name");
    删除：window.sessionStorage.removeItem("name");
    清除：window.sessionStorage.clear();
    说明：生命周期为关闭浏览器窗口,在同一个窗口下数据可以共享，可存储大约5M
    
本地存储：
    获取：window.localStorage
    获取：window.localStorage.getItem("name");
    删除：window.localStorage.removeItem("name");
    清除：window.localStorage.clear();
    说明：永久生效，除非手动删除,可以多窗口共享,可存储大约20M
```
## 二 历史管理
## 三 地理位置
HTML5提供了Geolocation地理位置支持。目前大多数浏览器都可以支持（IE9+）。  
出于安全考虑，部分最新的浏览器只允许通过HTTPS协议使用GeolocationAPI，在http协议下回跑出异常，当然本地开发不会有问题。  
访问地理位置方式：
```javascript
navigator.geolocation           //会提示用户是否允许获取该API的权限
```
示例代码：
```javascript
        //判断浏览器是否兼容代码
        if (navigator.geolocation) {
            navigator.geolocation.getCurrentPosition(
                function success(position) {
                    console.log("获取位置成功，当前位置为：", position);
                },
                function errror(error) {
                    console.log("获取位置错误，错误为：", error);
                }, 
                {
                    timeout: 10000,
                }
            )
        } else {
            //没有获取到地理位置执行的操作
        }
```
获取位置成功后，返回的position参数对象，包含下列信息：
```
timestamp： 获取位置时的时间戳
ciirds：    坐标信息对象，内部包含：latitude（纬度），longitude（经度），accuracy（坐标精度，单位为米）
```
获取位置失败后，返回的错误对象，常见为：
```
code:   错误标识，1为用户拒绝分享位置 2为获取用户位置失败 3为获取超时 0为其他错误
```
传入的位置参数对象：
```
timeout:允许获取位置的超时时间，单位为毫秒
enableHighAccuracy： 布尔值是否获取高精度信息 
maximumAge：用户位置信息缓存的最大时间，默认为0，单位是毫秒  
```
当用户位置发生变化时，可以通过watchPostion方法监听用户的位置信息，该方法和getCurrentPosition使用方式一致：
```javascript
let watchID = navigator.geolocation.watchPosition(success, error, option);
//取消监听
navigator.geolocation.clearWatch(watchID);
```
## 四 检测用户网络状态
我们可以通过window.onLine来检测，用户当前的网络状况，返回一个布尔值
```js
window.online       //用户网络连接时被调用
window.offline      //用户网络断开时被调用
window.addEventListener("online",function(){
    alert("已经建立了网络连接")
});
window.addEventListener("offline",function(){
    alert("已经失去了网络连接")
});
```
## 五 访问设备
### 5.1 访问摄像头
```
新标准：部分浏览器不支持，支持的浏览器有：Fifox36+（需要moz前缀），Chrome47+（需要webkit前缀）
navigator.getUserMedia
旧标准：目前已废除，支持的浏览器有：Edg12+，Firefox17+，Chrome21+，AndroidBrowser56+，Chrome for Android57+，UC11.4+
navigator.mediaDevices.getUserMedia
```
案例代码：
```html
<!DOCTYPE html> 
<html>  
  <head>  
    <meta charset="UTF-8">  
    <title></title>  
    <style>
      #canvas,#video {
        float: left;  
        margin-right: 10px;  
        background: #fff;  
      }      
      .box {  
        overflow: hidden;  
        margin-bottom: 10px;  
      }
    </style>
  </head>  
  <body>  
    <div class="box">
      <video id="video" width="400" height="300"></video>
      <canvas id="canvas"></canvas>
    </div>
    <button id="live">直播</button>
    <button id="snap">截图</button>
    <script>  
      let video = document.getElementById('video');
      let canvas = document.getElementById('canvas');  
      let ctx = canvas.getContext('2d');  
      let width = video.width;  
      let height = video.height;  
      canvas.width = width;  
      canvas.height = height;   
      function liveVideo(){  
        let URL = window.URL || window.webkitURL;   // 获取到window.URL对象
        navigator.getUserMedia({  
          video: true  
        }, function(stream){  
          video.src = URL.createObjectURL(stream);   // 将获取到的视频流对象转换为地址
          video.play();   // 播放
          //点击截图     
          document.getElementById("snap").addEventListener('click', function() {  
            ctx.drawImage(video, 0, 0, width, height);  
            let url = canvas.toDataURL('image/png');  
            document.getElementById('download').href = url;  
          });
        }, function(error){  
          console.log(error.name || error);  
        });  
      }  
      document.getElementById("live").addEventListener('click',function(){  
        liveVideo();  
      });    
    </script>  
  </body> 
</html>
```
### 5.2 传感器与摇一摇实战
现代手机都内置了传感器，通过传感器，可以感知手机的方向和位置变化。  
手机方向变化和位置变化的解释：
```
X轴：左右横贯手机方向，当手机绕X轴旋转时移动方向称为Beta
Y轴：上下纵贯手机方向，时手机绕Y轴旋转时移动方向称为Gamma
X轴：垂直手机平面方向，当手机绕Z轴旋转时移动方向称为Alpha
```
方向事件 deviceorientation：设备方向发生变化时触发  
```javascript
window.addEventListener('deviceorientation', orientationHandler , true);
```
回调函数orientationHandler 在注册后，会被定时调用，并会受到一个DeviceOrientationEvent类型的参数，通过该茶树获取设备的方向信息，如下所示：
```
absolute：如果方向数据跟地球坐标系和设备坐标系有差异，则为true，如果方向设备由设备本身的坐标系提供，则为false
alpha：设备在该方向上的旋转角度，范围是0-360
beta：设备在该方向上的旋转角度，范围是0-360
gamma：设备在该方向上的旋转角度，范围是0-360
```
移动事件devicemotion：设备位置发生变化时触发：
```javascript
window.addEventListener('devicemotion', motionHandler , true);
```
同样的，回调函数motionHandler被注册后，也会被定时调用，并收到一个DevicemotionEvnent的参数，该参数可以访问设备的方向和位置信息，参数如下：
```
acceleration：设备在XYZ三个轴方向上移动的距离，已抵消重力加速
accelerationIncludingGravity：设备在XYZ三个轴的方向上移动的距离，包含重力加速
rotationRate：设备在Alpha，Beta，Gamma三个方向旋转的角度
interval：从设备获取数据的频率（单位毫秒）
```
摇一摇案例：摇一摇可以理解为手机不低于已定的速度在移动。  
实现的方法是监听devicemotion事件后，判断设备在XYZ三个方向上的移动距离与前一次移动的距离差，并除以两次事件触发的时间差，即为移动设备移动的速度
```html
<!DOCTYPE html>
<html lang='en'>

<head>
    <meta charset='UTF-8' />
    <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0,user-scalable=no">
    <title>陀螺仪实现摇一摇</title>
</head>

<body>
    <div>用力摇一摇你的手机</div>
    <script type="text/javascript">
        let SHAKE_SPEED_THRESHOLD = 300; // 摇动速度阈值
        let lastTime = 0; // 上次变化的时间
        let x = y = z = lastX = lastY = lastZ = 0; // 位置变量初始化

        function motionHandler(evt) {
            let acceleration = evt.accelerationIncludingGravity; // 取得包含重力加速的位置信息
            let curTime = Date.now(); // 取得当前时间
            if ((curTime - lastTime) > 120) { // 判断
                let diffTime = curTime - lastTime; // 两次变化时间差
                lastTime = curTime; // 保存此次变化的时间
                x = acceleration.x;
                y = acceleration.y;
                z = acceleration.z;
                let speed = Math.abs(x + y + z - lastX - lastY - lastZ) / diffTime * 1000; // 计算速度
                if (speed > SHAKE_SPEED_THRESHOLD) { // 速度是否大于预设速度
                    alert("你摇动了手机");
                }
                lastX = x; // 保存此次变化的位置x
                lastY = y; // 保存此次变化的位置y
                lastZ = z; // 保存此次变化的位置z
            }
        }
        if (window.DeviceMotionEvent) {
            window.addEventListener('devicemotion', motionHandler, false);
        } else {
            alert('您的设备不支持位置感应');
        }
    </script>
</body>

</html>
```
## 六 离线应用
离线应用可以帮助用户在没有网络时使用web程序，H5的离线功能包含：离线资源缓存、在线状态监测、本地数据存储等。  
离线web应用比普通的web应用多了一个描述文件，该文件用来列出需要缓存和永不缓存的资源，描述文件的扩展名为：.manifest 或者 .appcache(推荐使用)。  
首先需要在项目目录下创建 offline.appcache文件：
```
CACHE MANIFEST      # 说明这是离线应用描述文件
CACHE:              # 会被缓存的资源列表
index.html
index.js
NETWORK:            # 总是从web获取的资源列表
test.js
```
html文件需要添加如下配置：
```html
<html lmanifest="./offline.appcache">
```













