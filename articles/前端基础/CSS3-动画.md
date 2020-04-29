# 动画

如渐变、移动、旋转、缩放等等

- 属性渐变`transition`: 过渡动画
    - `transition-property`: 属性
    - `transition-duration`: 运行时间
    - `transition-timing-function`: 曲线
    - `transition-delay`: 延迟
    - 常用钩子: `transitionend`
  ```css
  {
    transition: transform 1s ease;
  }
  ```

- `animation / keyframes`
    - `animation-name`: 动画名称，对应`@keyframes`
    - `animation-duration`: 运行时间
    - `animation-timing-function`: 曲线
    - `animation-delay`: 延迟
    - `animation-iteration-count`: 次数
        - `infinite`: 循环动画
    - `animation-direction`: 方向
        - `alternate`: 反向播放
    - `animation-fill-mode`: 静止模式
        - `forwards`: 停止时，保留最后一帧
        - `backwards`: 停止时，回到第一帧
        - `both`: 同时运用 `forwards / backwards`
    - 常用钩子: `animationend`
  
  ps: 动画属性: 尽量使用动画属性进行动画，能拥有较好的性能表现

    `transform`
    - `scale`   缩放
    - `translate`  移动
    - `rotate`  旋转
    - `skew`  倾斜  ~~ 稍后还得再查一下

    独立属性
    - `opacity`
    - `color`