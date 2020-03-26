## 关于本笔记

overnote系列笔记是我在平时一些学习时间阅读书籍博客、观看视频后整理出来的一套体系化的笔记，着重将一个整套知识体系由浅入深的记录下来，并做适当的横向扩展。  

由于笔记刚刚开始整理，许多地方仍然没有体系化、详细化，笔误或者错误依然很多，敬请期待不断校正后的正式版。正式版会以类似gitbook的形式开放，便于阅读。  

为何是笔记形式，而不是独立博客形式？
```
笔者引用并优化、扩展了很多优秀书籍内容。若不按目录主次写好，直接将这样的内容放在独立博客上，笔者认为是互联网的垃圾碎片

独立博客的内容目录极度不清晰，久而久之只能成为一个自己也不会问津的垃圾收集器。如果在平时学习中，学到了某个知识点，比如深入了解了闭包的用法，可以把心得通过git版本控制记录到笔记目录中对应的闭包章节，便于查找，而不是去博客上再次发一篇闭包文章，最终石沉大海。这样坚持下来，笔记就能越来越由浅入深，变得丰富。
```

## 新时代前端的学习方向要点

**需要深入的技术点**：
- 三大框架：Angular/Vue/React的深入理解，精通其原理、周边生态！
- TypeScript：优秀的JS超集，可以体验一下面向接口设计的理念，当然TS也会因为vue3.0的普及而持续上升份额。
- 小程序：小程序已经很多年了，不算新，Taro、Uni-app等第三方多端框架都可以试试。  
- NodeJS：笔者认为：无Node不前端，o(╯□╰)o。
- 计算机基础：所有开发者都逃不掉的三大件：算法、操作系统、计算机网络

**需要实践的技术点**：
- Nest.js：NodeJS后端服务框架，拥有完善IOC机制，可以提升前端的架构视野，当然阿里的midway也不错，这2个框架也是直接使用TS书写，可以加强TS学习
- Flutter：笔者认为是具有全新意义的跨平台框架
- GraphQL：
- Electron：快速构建跨平台应用的利器
- 持续集成：docker，CI/CD，甚至可以了解k8s、Serverless

**可以简单了解的技术点**：
- PWA：简单关注即可，其技术原理也极其简单
- WASM：目前火热的rust、go等语言已经支持，可以尝试
- IOT：
- TensorFlow.js

## 资料

> 没有被录入的一部分书籍也并不代表他们不优秀，只是很多内容与下列推荐书籍重复了，尽量精读1-2本

### 网页与动画

- [《HTML5与CSS3基础教程》第8版](https://book.douban.com/subject/25878992/)：HTML与CSS基础入门
- [《HTML5秘籍》](https://book.douban.com/subject/11610880/)：H5全面了解书籍
- [《CSS揭秘》](https://book.douban.com/subject/26745943/)：重点书籍，CSS书籍瑰宝！！！
- [《HTML5+JavaScript动画基础》](https://book.douban.com/subject/24744218/)：目前最好的动画书籍
- [《HTML5 Canvas核心技术》](https://book.douban.com/subject/24533314/)：目前最好的canvas书籍

### JavaScript

语法基础：
- [《JavaScript高级程序设计》第3版](https://book.douban.com/subject/10546125/)：外号红宝书，笔者认为是最好的JS入门书籍
- [《JavaScript语言精粹》修订版](https://book.douban.com/subject/11874748/)：外号蝴蝶书，是JS核心语法的荟萃，短小精悍
- [《JavaScript权威指南》第6版](https://book.douban.com/subject/10549733/)：外号犀牛书，是一部JS的百科全书，过于庞大，有红宝书足矣

DOM编程：
- [《JavaScript DOM编程艺术》第2版](https://book.douban.com/subject/6038371/)
- [《锋利的jQuery》第2版](https://book.douban.com/subject/10792216/)
- [《jQuery技术内幕》](https://book.douban.com/subject/25823709/)

提升篇：
- [《JavaScript忍者秘籍》第2版](https://book.douban.com/subject/30143702/)：对函数的讲解：闭包、重载、curry化讲解极好
- [《JavaScript核心技术开发解密》](https://book.douban.com/subject/30190189/)：短小精悍的介绍了一些难点
- [《你不知道JavaScript》上卷+中卷](https://book.douban.com/subject/26351021/)：笔者认为目前最好的JS深入书籍之一
- [《JavaScript设计模式与开发实践》](https://book.douban.com/subject/26382780/)
- [《JavaScript面向对象精要》](https://book.douban.com/subject/26352658/)
- [《JavaScript面向对象编程指南（第2版》](https://book.douban.com/subject/26302623/)

ES6篇：
- [《深入理解ECMAScript6》](https://book.douban.com/subject/27072230/)
- [《ECMAScript6入门》第3版](https://book.douban.com/subject/27127030/)

Vue：
- [《深入浅出Vue.js》](https://book.douban.com/subject/32581281/)

React:
- [《深入React技术栈》](https://book.douban.com/subject/26918038/)
- [《深入浅出React和Redux》](https://book.douban.com/subject/27033213/)
- [《React状态管理与同构实战》](https://book.douban.com/subject/30290509/)

综合：
- [《高效前端：Web高效编程与优化实践》](https://book.douban.com/subject/30170670/)
- [《高性能JavaScript》](https://book.douban.com/subject/5362856/)
- [《了不起的JavaScript工程师：从前端到全端高级进阶》](https://book.douban.com/subject/34788884/)：笔者力荐

### Node书籍

- [《Node.js In Action》](https://book.douban.com/subject/25870705/)：Node基础书籍，内容繁杂，都点到为止，适合入门
- [《深入浅出Node.js》](https://book.douban.com/subject/25768396/)：Node书籍的集大成者，虽然一部分技术已经过时，但是仍然值得购买并收藏
- [《Node.js 设计模式》第2版](https://book.douban.com/subject/30159269/)：很不错的书籍
- [《Node.js:来一打C++扩展》](https://book.douban.com/subject/30247892/)：窥探Node底层的好书


## 附录：笔记汇总

**OverNote**地址：https://github.com/overnote    
**笔者的地址**：https://github.com/ruyuejun  

**OverNote分类**：  
- [Golang](https://github.com/overnote/over-golang)：详尽的Go领域笔记：Go语法、Go并发编程、GoWeb编程、Go微服务等
- [服务端架构](https://github.com/overnote/over-server)：分布式与微服务笔记，附Nginx、Mysql、Redis等常用服务端技术
- [大前端](https://github.com/overnote/over-javascript)：包含JavaScript、Node.js、vue/react、微信开发、Flutter等大前端技术
- [Python与机器学习](https://github.com/overnote/over-python)：Python相关笔记，完善中  
- [数据结构与算法](https://github.com/overnote/over-algorithm)：以C/Go实现为主记录数据结构与算法的笔记
- [Linux](https://github.com/overnote/over-linux)：计算机组成原理、操作系统、计算机网络、编译原理基础学科笔记
- [大数据](https://github.com/overnote/over-bigdata)：大数据笔记，完善中
- [flutter](https://github.com/overnote/over-flutter)