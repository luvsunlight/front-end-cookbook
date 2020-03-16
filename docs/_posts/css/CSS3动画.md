## transform

transform即对元素本身进行各种尺度的变化，包括

* transform-origin: (left, bottom) 表示变形的基点
* rotate
* translate
* scale
* skew（扭曲，3D变化）
* matrix

## transition

用来描述样式的属性值是如何从一个状态变成另外一种状态的,可以应用于任何触发样式变化的场景比如hover时width变化等等，给我的感觉就是在css属性变化时增加的一层拦截，可以定义其属性变化的特点

* transition-property（变换的属性，即那种形式的变换：大小、位置、扭曲等）
    * none
    * all
    * color
    * length
    * width
    * ...
* transition-duration（变换延续的时间）
    * 100ms
    * 100s
* transition-timing-function（变换的速率）
    * ease(均匀变慢)
    * linear（匀速）
    * ease-in（加速）
    * ease-out（减速）
    * ease-in-out（先加速后减速）
    * cubic-bezier（自定义贝塞尔曲线）
* transition-delay（变换的延时）

## animation

如果说transition和transform的配合只能做到一次变化的，那么animation的出现就给CSS的动画提供了无数的可能，以前我们用transform或者hover这样的伪类只能做到一次状态的变化，但是animation提供了补间的可能

* @keyframes
    * from
    * <percentage>
    * to
* animation-name
* animation-duration
* animation-timing-function
* animation-delay
* animation-iteration-count
* animation-direction
* animation-play-state