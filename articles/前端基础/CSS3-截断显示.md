# 截断显示

主要涉及内容：
1. 主要使用属性
2. 使用的限制条件
3. 属性具体作用

## 单行显示
```css
div {
	<!-- 前提是宽度是有限的，不然根本用不到这个，会一直撑着content -->
	overflow: hidden;
	<!-- 不折行 -->
	white-space: nowrap;
	<!-- 真正对于内部的属性，text超出如何展示 -->
	text-overflow: ellipsis;
}

```

## 多行显示s
说明一下，目前只能在webkit浏览器下使用
```css
div {
	overflow: hidden;
	text-overflow: ellipsis;
	<!-- 以下只能在webkit内核中使用 -->
	display: -webkit-box;
	-webkit-line-clamp: 2;
	-webkit-box-orient: vertical
}
```