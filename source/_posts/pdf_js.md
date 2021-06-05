---
title: 使用pdf.js展示pdf及展示电子签章
date: 2021-04-01 19:16:11
tags: [2021,pdf.js]
categories: [代码]
top_img: /img/pdf.png
cover: /img/pdf.png
---
> 项目中之前的小伙伴用的pdfh5.js,用起来有些许bug,于是乎准备使用pdf.js代替之

### js下载
点击[pdf.js官网](http://mozilla.github.io/pdf.js/getting_started/#download)下载，本次使用的为 v2.7.570
下载后保持结构不变移植到项目中
![1617330465291](/images/1617330465291.jpg)


### 使用
>本次使用iframe用法，其他用法略复杂，等以后再研究

在需要展示pdf的地方引用一下代码即可，官方已经封装好用法，即web目录下的viewer.html,在之后跟踪file参数接pdf地址即可，示例如下：

```html
<iframe src="js/plugins/pdfjs/web/viewer.html'?file=http://tax-manage.test/storage/contract/test.pdf')}}" frameborder="0" width="100%" height="800px" id="iframe"></iframe>
```
效果图
![](/images/16173310594409.jpg)

### 签章展示

####  签章报错


然鹅，如上使用后pdf可展示了，but 电子签章不展示，打开控制台看到一个报错
util.js:366 Warning: Unimplemented widget field type "Sig", falling back to base field type.
![](/images/16173314778345.jpg)

#### 无效的解决方案

查了半天，都说注释掉`pdf.worker.js`里三行的代码
```javascript
// if (data.fieldType === "Sig") {
//   data.fieldValue = null;
//   this.setFlags(_util.AnnotationFlag.HIDDEN);
// }
```
然并卵

#### 真正解决

最后在一位叫带甜味的盐的同学的文章（[原文链接](https://blog.csdn.net/s_y_w123/article/details/108869862)）下找到答案
在`pdf.worker.js`中搜索下面一行
![](/images/16173320819737.jpg)

```javascript
(0, _util.warn)('Unimplemented widget field type "' + fieldType + '", ' + "falling back to base field type.");
```

修改为以下
![](/images/16173321213805.jpg)
```javascript
if(fieldType !== "Sig") {
    (0, _util.warn)('Unimplemented widget field type "' + fieldType + '", ' + "falling back to base field type.");
    return new WidgetAnnotation(parameters);
}
```
具体为啥修改，请戳上面原文查看

#### 效果

乙方签章已展示，奶思

![](/images/16173322941298.jpg)


>忍不住吐槽，现在搜个问题解决方案是真滴难，到处都是无脑cv的答案，毛用没有，难受

![](/images/16173330315869.jpg)

