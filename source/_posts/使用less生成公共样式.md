---
 title: "使用less生成公共样式"
 date: 2019-12-31
 tags: [Less]
 categories: [前端笔记]
---

项目初始阶段需要根据设计稿抽离出公用的样式，比如width,height,padding,margin，这时候就可以使用less的变量，函数，循环，快速生成原子级样式

```
@widthList: 18, 50, 240;
@heightList: 40, 48;

@spacingList: 5, 10, 12, 15, 16, 20, 24, 40, 60;
@directionShort: l, r, t, b;
@direction: left, right, top, bottom;

.generWidth(@i) when (@i < length(@widthList)+1) {
    .widths(extract(@widthList, @i));
    .generWidth(@i+1);
}
.generHeight(@i) when (@i < length(@heightList)+1) {
    .heights(extract(@heightList, @i));
    .generHeight(@i+1);
}
.widths(@size) {
    .w-@{size} {
        width: unit(@size, px);
    }
}
.heights(@size) {
    .h-@{size} {
        height: unit(@size, px);
    }
}

.generSpacingDirection(@j,@i) when (@j < length(@direction)+1) {
    .spacing(extract(@directionShort, @j), extract(@direction, @j), extract(@spacingList, @i));
    .generSpacingDirection(@j+1, @i);
}
.generSpacing(@i) when (@i < length(@spacingList)+1) {
    .spacingUnit(extract(@spacingList, @i));
    .generSpacingDirection(1, @i);
    .generSpacing(@i+1);
}

.spacingUnit(@size) {
    .p-@{size} {
        padding: unit(@size, px);
    }
    .px-@{size} {
        padding-left: unit(@size, px);
        padding-right: unit(@size, px);
    }
    .py-@{size} {
        padding-top: unit(@size, px);
        padding-bottom: unit(@size, px);
    }
    .m-@{size} {
        margin: unit(@size, px);
    }
    .mx-@{size} {
        margin-left: unit(@size, px);
        margin-right: unit(@size, px);
    }
    .my-@{size} {
        margin-top: unit(@size, px);
        margin-bottom: unit(@size, px);
    }
}
.spacing(@directionShort,@direction,@size) {
    .p@{directionShort}-@{size} {
        padding-@{direction}: unit(@size, px);
    }

    .m@{directionShort}-@{size} {
        margin-@{direction}: unit(@size, px);
    }
}
:global {
    html,
    body,
    #app {
        height: 100%;
    }
    .generWidth(1);
    .generHeight(1);
    .generSpacing(1); 
}
```

得到相应的padding，margin，width，height原子级样式，如下demo  
![生成结果](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5b5c2604bec72~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png "生成结果")