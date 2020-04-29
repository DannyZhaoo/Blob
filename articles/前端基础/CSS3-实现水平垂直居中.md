# 实现水平垂直居中
## 实现水平居中

### 行内元素 
```css
{ text-align: center; }
```

### 块级元素
```css
{
  /* 第一种方法，只要左右是auto就行 */
  margin: 0 auto; 

  /* 第二种方法，绝对定位，相对更好一些，因为translate会 */
  position: absolute;
  left: 50%;
  transform: translateX(-50%);

  /* 第三种方法 */
  display: flex;
  justify-content: center;
}
```


## 实现垂直居中

### 行内元素 
```css
{ line-height: height; }
```

### 块级元素
```css
{
  /* 第一种方法，只要左右是auto就行 */
  margin: 0 auto; 
  
  /* 第二种方法，绝对定位 */
  position: absolute;
  top: 50%;
  transform: translateY(-50%);

  /* 第三种方法 */
  display: flex;
  align-items: center;
}
```



## 实现水平垂直居中

### 行内元素
```css
{
  text-align: center;
  line-height: height;
}
```

### 块级元素
```css
{
  /* 第一种方法 */
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);

  /* 第二种方法 */
  display: flex;
  justify-content: center;
  align-items: center;
}

```