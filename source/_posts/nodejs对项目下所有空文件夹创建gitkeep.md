---
title: "nodejs对项目下所有空文件夹创建gitkeep"
date: 2019-12-31
tags: [Node.js]
categories: 
---

项目/框架初始化时可能需要保留一些空文件，这时候就需要批量新增gitkeep

```javascript
const fs = require('fs')
const baseurl = 'D:/test'
const ignoreDir = ['.git', '.vscode', 'node_modules']
addGitkeep(baseurl)
function addGitkeep(url) {
    fs.readdir(url, {withFileTypes: true}, (err, files) => {
        err && console.log(err)
        //该目录下没有文件
        if (!files.length) {
            return fs.writeFile(url + '/.gitkeep', null, err => {
                err && console.log(err)
            })
        }
        files.forEach(dirent => {
            if (!ignoreDir.includes(dirent.name) && dirent.isDirectory()) {
                addGitkeep(url + '/' + dirent.name)
            }
        })
    })
}
```