---
 title: "React中使用UMEditor"
 date: 2019-12-31
 tags: [React.js]
 categories: [前端笔记]
---

最近项目中需要使用富文本编辑器，参考了运营小姐姐日常使用平台上的编辑器，最后考虑采用百度的UMEditor。因为轻量，功能和配置简单，没有很多定制化的功能，所以没采用UEditor。不过我后续会出一篇文章将UEditor的二次开发。

umeditor的引入
-----------

### 组件设计

首先看一下组件大致的内容：  
1.组件props：  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5dfb1e4235b~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "image.png")  
2.组件关键的成员属性：  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5dfb1fabe1f~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "image.png")  
3.简单的render:  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5dfb2058558~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "image.png")  
4.UMEditor的实例化  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5dfb20a18f3~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "image.png")  
UMEditor源码里需要改动的主要就是图片的请求了，配置中的imgUrl我传的是一个方法，这个方法中请求后台并返回Promise<{url:string}>

### 源码修改

源码修改两个文件  
image.js中两处更改  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5dfd3e142f5~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "image.png")

![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5dfd787ad81~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "image.png")  
autoupload.js中一处修改  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5dfdeb77fa6~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "image.png")

UMEditor的源码存放在dll目录下，打包时会被webpack拷贝道相应的目录下，UMEDITOR\_HOME\_URL和这个目录路径保持一致  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5dfdedd31e6~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "image.png")

umeditor的依赖处理
-------------

### 文件合并

由于依赖文件过多，我们使用gulp合并一下  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5e005ec6e2d~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "image.png")  
core文件夹下的依赖合并为core.min.js,其他plugin，ui，addapter也一样合并为相应的min.js  
原本由editor\_api.js引入依赖的，现在我们自己写个方法引入。

### 依赖加载

组件中定义需要引入的文件，这是一个二维数组，同级的文件按顺序引入，不同级别的可以并发请求，比如：\['/third-party/jquery.min.js', '/third-party/template.min.js'\]中的两个文件同时请求，但是保证它们都load完再请求后面的文件  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5e00b71d6eb~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "image.png")

加载的时候使用SyncRequire方法  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5e00ce73838~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "image.png")

### 使用一步迭代器实现可控加载

![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5e00f5fb948~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "image.png")  
loadDep负责文件加载，具体如下：  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5e031d28570~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "image.png")  
SyncRequire内部维护一个异步迭代器，迭代的对象是每一个文件的加载。最后使用for await进行异步迭代  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5e0346a17e1~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "image.png")  
如果是一个文件路径数组，则说明这个数组中的文件可以同时使用loadDep加载，如果是一个文件路径字符串，则说明这个文件加载完才可以加载后面的文件。loaders具体实现如下  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5e035e1c019~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "image.png")

好了，有了这个SyncRequire方法，就可以既同时又顺序加载我们的外部js或者css文件了，最后可以在控制台中验证效果  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5e039342268~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "image.png")