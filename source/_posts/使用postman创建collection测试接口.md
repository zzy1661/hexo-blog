---
 title: "使用postman创建collection测试接口"
 date: 
 tags: [Postman]
 categories: 
---

postman可以创建一个工作流按顺序测试多个接口，并可以将前面的接口的返回值作为变量传递给后面的接口使用。

比如要顺序测试智联卓聘的接口：  
1.[data.highpin.cn/api/J...](https://data.highpin.cn%2Fapi%2FJobSearch%2FSearch "https://data.highpin.cn/api/JobSearch/Search") 查询职位列表  
2.[www.highpin.cn/api/jo...](https://www.highpin.cn%2Fapi%2Fjob%2FGetPositionDetail "https://www.highpin.cn/api/job/GetPositionDetail") 查询职位详情

那么我们首先编辑两个接口：

1.  *   body部分 ![body部分](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b593f8f41412~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "body部分")
    *   Pre-request Script中可以在请求发送前设置一个环境变量：Q;body中可以使用`{{}}`的形式获取环境或全局变量
        
        ```routeros
        pm.environment.set('Q','前端'); 
        ```
        
    *   Test中可以获取到查询结果，这里我们取第一条记录的信息传递给后面的接口
        
        ```applescript
        var id = pm.response.json().body.JobList[0].JobID;
        var ref = pm.response.json().body.JobList[0].ReferrerType;
        pm.globals.set("id", id);
        pm.globals.set("ref", ref);
        ```
        
2.  第二个接口可以这样获取前面设置的全局变量  
    ![图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b593f8e2d9f5~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "图片描述")
3.  将这两个接口保存为collection，侧边栏可以调整执行的顺序；点击run进入Collection Runner界面，  
    ![图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b593f8e7000b~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "图片描述")
4.  运行一下，可以得到两个接口执行的结果  
    ![图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b593f8f47206~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "图片描述")