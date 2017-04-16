## CSS画波浪线的方法

注：参考文章 [CSS3绘制波浪线](http://oddjohn.github.io/blog/css3-wavy-line/)、[CSS3 | 制作文字波浪线效果](http://www.jianshu.com/p/8570433e3669)

#### linear-gradient()

第一步：先画出45deg方向的梯度斜线

```html
<div>  
</div>

div {
    background: linear-gradient(45deg, transparent 45%, red 55%, transparent 60%);
    height: 10px;
    background-size: 10px 10px
}
```

第二步：在上一步的基础上再画出135deg方向的梯度斜线

```html
<div>
</div>

div {
    background: linear-gradient(45deg, transparent 45%, red 55%, transparent 60%),linear-gradient(135deg, transparent 45%, red 55%, transparent 60%);
    height: 10px;
    background-size: 10px 10px  
}
```

第三步：最后把高度设为原来的一半

```html
<div>
</div>

div {
    background: linear-gradient(45deg, transparent 45%, red 55%, transparent 60%),linear-gradient(135deg, transparent 45%, red 55%, transparent 60%);
    height: 5px;
    background-size: 10px 10px  
}
```

#### radial-gradient()

第一步：先画出一个半径为10px的圆

```html
<div>
</div>

div {
  width: 100%;
  height: 20px;
  background-image: radial-gradient(circle, transparent, transparent 9px, orange 10px, orange 10px, transparent 10px, transparent);
}
```

第二步：通过background-size设置背景图案长宽刚好等于圆的直径来铺面当前div

```html
<div>
</div>

div {
  width: 100%;
  height: 20px;
  background-image: radial-gradient(circle, transparent, transparent 9px, orange 10px, orange 10px, transparent 10px, transparent);
  background-size: 20px 20px;
}
```

第三步：将高度设为原来的一半

```html
<div>
</div>

div {
  width: 100%;
  height: 10px;
  background-image: radial-gradient(circle, transparent, transparent 9px, orange 10px, orange 10px, transparent 10px, transparent);
  background-size: 20px 20px;
}
```

#### text-decoration-style: wavy

可以通过	`text-decoration-style: solid|double|dotted|dashed|wavy|initial|inherit;`来设置下划线的样式，其中wavy为波浪线样式，但是遗憾的是目前只有火狐浏览器支持text-decoration-style

```html
//Firefox下运行
<div>
</div>

div {
  color: #fff;
  text-decoration: underline; 
  text-decoration-color: #000;
  -moz-text-decoration-color: #000;
  text-decoration-style: wavy; 
  -moz-text-decoration-style: wavy; /* 针对 Firefox 的代码 */
}
```

