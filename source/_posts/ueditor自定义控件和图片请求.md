---
 title: "ueditor自定义控件和图片请求"
 date: 2019-12-31
 tags: [JavaScript]
 categories: 
---

认识ueditor
---------

ueditor时百度出的一款非常强大的富文本编辑器，但是并不能将它理解为一个现成的编辑器。富文本编辑器的原理并不复杂，核心api是document.execCommand，以及Selection和Range操作。对于一些自定义的控件，则可以用原生的dom操作完成。然而正式因为dom操作和状态判断非常复杂，因此ueditor铺设了整个编辑器的底层逻辑。所以，你应该将ueditor理解成一个基础设施。

自定义请求
-----

如果遇到富文本编辑器需求，拿到ueditor官网的demo后，首先要改的就是图片上传的请求。一般项目中，请求都会封装成统一的方法，返回Promise或者PromiseLike，因此我们将serverUrl选项改为一个方法  
`imgUpload: (file:File) =>Promise<{url:string}>;`  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5f5238cd06b~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "image.png")  
然后我们抛弃它自带的图片上传逻辑，自定义一个控件叫image，沿用simpleupload的图标，点击图标会生成一个input\['type="file"'\]，然后模拟点击这个input，进行图片选择。选中后调用serverUrl中配置的方法，得到后台返回的url后执行  
`editor.execCommand('insertimage',{src:res.url});`

具体代码如下

```
editorui.image=function(editor) {

    var ui=neweditorui.Button({

    className:"edui-for-simpleupload",

    title:

    editor.options.labelMap['simpleupload'] ||

    editor.getLang("labelMap.simpleupload") ||

    "",

    theme:editor.options.theme,

    onclick:function() {

    var fileInput=document.createElement('input');

    fileInput.id='ueditor-custom-upload'

    fileInput.type='file'

    fileInput.style.display='none'

    fileInput.click()

    fileInput.addEventListener('change',function(e){

    var uploadFn=editor.getOpt('serverUrl');

    uploadFn(e.target.files[0]).then((res)=>{

    editor.execCommand('insertimage',{src:res.url});

          })

        })

      }

    });

editorui.buttons.image=ui;

return ui;

};
```

然后在ueditor.config.js的toolbars和ueditor.all.js的btnCmds中添加'image'的命令。这样我们就大致完成了图片上传的改造。

自定义其他控件
-------

自定义控件分两个步骤：1.自定义新的控件ui；2.自定义新的控件命令。我们举两个例子：

### 对图片加边框的控件

首先生成相应的ui，代码如下，其中ui.setDisabled和ui.setChecked是控制控件是否可点和是否已经加了边框。editor.execCommand('imgborder')和editor.queryCommandState('imgborder')、editor.queryCommandValue('imgborder')是自定义的控件命令。  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5f523b4dd6a~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "image.png")  
有了ui后还需要添加相应的css，我们找到一张imageborder的图片，然后在ueditor.css中添加样式代码，这样样式的改造就完成了。  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5f523cbbe72~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "image.png")  
然后需要自定义image-border的命令：  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5f523cf1dc1~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "image.png")  
execCommand执行命令，添加border或者取消border，另外两个方法是查询当前是否可以执行命令和是否已经执行了命令。  
当然别忘了在toolbars和btnCmds中别忘了添加相关的命令  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5f54c835ba7~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "image.png")  
至此，这个控件就完成了。

### 添加字间距控件

字间距是一个下拉列表，首先在ueditor.config.js中进行可选项的配置：toolbars中添加控件'letterspace'  
然后改造ueditor.js,btnCmds中添加'letterspace'。  
增加letterspace的ui  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5f54dbceac6~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "image.png")  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5f54ffc1303~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "image.png")  
增加command  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5f5501bac62~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "image.png")

最终demo如下  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5f574730aee~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "image.png")