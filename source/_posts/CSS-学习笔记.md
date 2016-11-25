## [CSS学习笔记] 基础知识
<!--more-->
#### align-items
定义flex子项在flex容器的当前行的侧轴（纵轴）方向上的对齐方式
stretch|center|flex-start|flex-end|baseline|initial|inherit

| 值	| 描述 |
| ------------- |
| stretch |	默认值。项目被拉伸以适应容器。|
| center	| 项目位于容器的中心。	|
| flex-start |	项目位于容器的开头。|	
| flex-end |	项目位于容器的结尾。	|
| baseline |	项目位于容器的基线上。	|
| initial |	设置该属性为它的默认值。请参阅 initial。	|
| inherit	 | 从父元素继承该属性。请参阅 inherit。|

#### 元素居中显示
```
<html>
<head>
<style type="text/css">
#container {
  display:flex;
  flex-direction:row;
  justify-content:center;
  align-items:center;
  height:200px;
  width:200px;
  background:#ff0000;
}
#container div{
  background:#0000ff;
  height:20px;
  width:20px;
}
</style>
</head>
<body>

<div id="container">
    <div></div>
 </div>

</body>
</html>
```

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2591396-e29d5ed34f15f213.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
