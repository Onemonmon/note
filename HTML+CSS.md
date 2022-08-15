### 1. BFC

```
块级格式化上下文，是一个独立的渲染区域，规定了内部盒模型如何布局，且其子元素不会影响到外部元素
触发属性：
1. overflow: 不是visible; 
2. float：不是none;
3. display: flex/inline-block; 
4. position: absolute/fixed; 
解决的问题：
1. 计算BFC高度时，浮动元素也会参与计算，可以解决float导致的高度坍塌
2. 同一个BFC下的两个相邻元素上下margin会重叠，可以为其中一个元素创建新的BFC解决
3. 不会与浮动元素重叠，可以阻止普通文档流被浮动元素遮盖
```

### 2. 水平垂直居中

```
1. 行内元素父元素line-height等于heigth，text-align: center，子元素vertical-align: middle
3. flex布局justify-content: center; align-items: cente
3. 子元素绝对定位，top: 0; left: 0; right: 0; bottom: 0; margin: auto
4. 子元素绝对定位，top: 50%; left: 50%; transform: translate(-50%, -50%)
```

### 3. 样式优先级

```
优先级：(A, B, C, D)
A. style内联样式
B. id选择器
C. 类名选择器、属性选择器a[rel="xxx"]、伪类a:hover
D. 标签选择器、伪元素::before
!important优先级最高
```

