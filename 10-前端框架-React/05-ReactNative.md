## 一 ReactNative简介
React Native为移动开发者提供了移动UI组件，移动开发者只需要使用react一种技术便可以开发Android和iOS多平台应用。现阶段WebAPP的的体验还是无法达到Native APP的体验，所以RN更加强调的是learn once,write everywhere。  
特别注意：react native在ios上仅支持ios7以上,Android仅支持Android4.1以上。  
推荐学习地址：http://www.lcode.org/react-native/  
## 二 环境搭建
### 2.1 win环境配置Android
第一步：系统环境
> 安装jdk，配置java环境变量即可
> 安装git
> 安装python2.7
> 安装node5以上环境
第二步：安卓环境
直接安装AndroidStudio。

### 2.2 mac环境配置iOS
### 2.3 填坑指南
http://www.lcode.org/react-native%E7%96%91%E9%9A%BE%E7%82%B9%E9%97%AE%E9%A2%98%E6%B7%B1%E5%9D%91%E6%9C%80%E5%BC%BA%E6%80%BB%E7%BB%93%E5%B8%96%E4%B8%8D%E6%96%AD%E6%9B%B4%E6%96%B0%E4%B8%AD/
## 三 HelloWorld
第一步：安装依赖
```
npm install -g react-native-cli
react-native init ReactNativeExample
```
第二步：启动服务
```
react-native start  // 需要一直开着
/*
访问：http://localhost:8081/index.android.bundle?platform=android
如果你遇到了ERROR Watcher took too long to load的报错，请尝试修改node_modules\react-native\packager\react-packager\src\DependencyResolver\FileWatcher\index.js将其中的MAX_WAIT_TIME 从25000改为更大的值（单位是毫秒）
*/
```
第三步：运行项目
```
react-native run-android
摇晃设备或按Menu键（Bluestacks模拟器按键盘上的菜单键，通常在右Ctrl的左边 或者左Windows键旁边）
（1）可以打开调试菜单，点击Dev Settings
（2）选Debug server host for device，输入你的正在运行packager的那台电脑的局域网IP加:8081（同时要保证手机和电脑在同一网段，且没有防火墙阻拦）
（3）再按back键返回，再按Menu键，在调试菜单中选择Reload JS，就应该可以看到运行的结果了。
如果真实设备白屏但没有弹出任何报错，可以在安全中心里看看是不是应用的“悬浮窗”的权限被禁止了。http://jingyan.baidu.com/article/f25ef25466c0fc482d1b824d.html

```
第四步：调试
打开Chrome，访问 http://localhost:8081/debugger-ui，应当能看到一个页面。按F12打开开发者菜单。