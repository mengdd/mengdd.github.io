---
title: Say Hello to ConstraintLayout
date: 2017-02-06 15:49:03
tags: [Android, ConstraintLayout]
categories: [Android]
---

## ConstraintLayout介绍
`ConstraintLayout`让你可以在很平的view结构(没有多层布局嵌套)中构建一个复杂的布局结构. 有点像`RelativeLayout`, 所有的view都是根据它和兄弟View和父layout的关系布局的, 但是它比`RelativeLayout`要更加灵活, 在Layout Editor中也更加好用.

<!-- more -->

在Layout Editor中你可以直接靠拖拽来构建`ConstraintLayout`.

为了在`ConstraintLayout`中定义一个view的位置, 你必须给view加上两条或多条约束(constraints). 每一条约束代表了一种和其他View(或parent, 或不可见的guideline)之间的联系或者对齐关系.

每一条约束都规定了这个view沿着水平或竖直轴的一个位置, 所以每个View在每个轴上都必须至少有一条约束(但是通常需要更多).

## Setup
首先确认下载support库, 在`Tools > Android > SDK Manager`的`SDK Tools`tab下:
展开`Support Repository`, check `ConstraintLayout for Android`和`Solver for ConstraintLayout`. 
Check `Show Package Details`, 显示版本信息.

比如当前我最新的版本信息是1.0.0-beta4, 我在module的build.gradle中添加:
```
dependencies {
    compile 'com.android.support.constraint:constraint-layout:1.0.0-beta4'
}
```
然后点击Sync即可.

## 转换已有布局
我们可以新建布局, 然后让它的根节点是`android.support.constraint.ConstraintLayout`.

除此之外, 我们还可以直接转换已有布局. 
打开Layout, 切换到`Design`tab, 然后在`Component Tree`窗口, 右击布局然后选择最底部的`Convert XXXLayout(这里是你布局节点的类型) to ConstraintLayout`.

## 添加约束
在Design模式下, 从Palette窗口中拖一个View到editor中去. 当你把一个View加入到ConstraintLayout中之后, 它会展示出一个bounding box, 四角的四个小方块用来拖拽调节大小, 每一个边上都有一个小圆点用来建立约束.
这些小方块和小圆点都被称为`handles`.

点击View, 然后点击并拖住一个约束handle, 把它拖拽到一个可用的anchor point(另一个View的边缘, layout的边缘, 或者一个guideline).当你松手的时候, 约束就生效了. (有一个默认的[margin](https://developer.android.com/training/constraint-layout/index.html#adjust-the-view-margins))

有几个规则:
- 每个View都至少有两条约束: 一个水平的一个竖直的.
- 你只能在共享平面的handle和anchor point之间建立约束. 比如一个View的竖直平面只能和另一个竖直平面建立约束, baseline也只能和其他baseline建立约束.
- 每一个handle只能被用来建立一个约束, 但是你可以对一个anchor point建立(来自多个View的)多个约束.

要删掉一个约束, 只需要选择这个view, 点击那个对应的handle.

如果你给同一个View加了两个相反的约束, 约束的线条就会变成弹簧状, 来显示两个相反方向的约束.  当View内容的尺寸固定或者是wrap的时候, 在这种情况下View就会在两个约束下居中显示, 如果你想让它展开, 那么就应该修改它的尺寸为[Any Size](https://developer.android.com/training/constraint-layout/index.html#adjust-the-view-size); 如果你想要保持当前的尺寸, 你可以[调节约束的权重](https://developer.android.com/training/constraint-layout/index.html#adjust-the-constraint-bias).

通常情况下可以加的有这几种约束:
- Parent constraint: View的边和Parent的边的关系.
- Position constraint: View之间水平和竖直的位置关系, 拖动可改变相对的margin距离.
- Alignment constraint: View边之间的对齐关系, 对齐后可以调节偏移量.
- Baseline alignment constraint: 对齐View的text baseline, 要创建baseline约束, 首先选中View, 然后把鼠标放在baseline上方两秒钟, 等它变白就可以拖到另一个baseline去建立约束了.
- Constrain to a guideline: 可以创建竖直或水平的guideline, 然后往上绑定约束, guideline对于用户来说是不可见的. 放置guideline的时候可以根据相对于layout边缘dp单位的距离, 也可以根据百分比.
Toolbar上有Guideline的按钮, 点击可选择水平或竖直.
点击Guideline尾部的小圆圈可以切换它到底是根据距离还是百分比放置的, 然后拖动它放到一个想要的位置.

## 使用Autoconnect和Infer Constraints
当打开`Autoconnect`模式之后, 每一个**新加的View**都会自动创建约束. Autoconnect模式默认是关闭的.

点击`Infer Constraints`会给layout中当前所有的View创建约束, 这是一个一次性的action. 它会选择建立最有效的约束, 所以它可能会建立离得很远的两个view之间的约束. 不像Autoconnect模式开启下, 只给新加的View建立约束, 并且只选择最近的元素.

## 调整View大小
可以通过拖拽View四个角的handles来改变View的大小, 但是这样生成的是hard-coded的尺寸, 对于适配来讲这样是不好的.

你可以点击View然后在Properties窗口编辑尺寸.
![ConstraintLayout Properties Window](/images/constraint-layout-properties-window.png)

有三种尺寸模式:
- Wrap Content:  用`>>>`图形表示.
- Any Size: 用弹簧图形表示. 说明View会一直展开到满足所有约束, 实际的值是0dp. 可以把它想象成"match constraints". 如果此时只有单边的约束, 那么它只展开到能放下自己的内容为止. 
- Fixed: 用图形`|-|`表示, 固定尺寸.

可以通过点击图形符号来切换这些模式.

注意: 在`ConstraintLayout`中的View中不应该使用`match_parent`, 而是用"Any Size"(0dp).

## 调整约束偏差
当你给一个View的对立两边都添加了约束, 而View的尺寸是fixed或者wrap_content, 那么默认情况下View就会居中显示在两个anchor point之间(bias=50%).
你可以通过拖拽View或者在Properties窗口中拖拽bias slider来改变它的偏移权重.

## 调节View边距
可以在toolbar上点击默认边距(8)来修改.

注意这个修改只对修改后新添加的View生效.

每一个View的边距都可以通过Properties窗口修改: 点击约束线条上的margin数字.

注意提供的值都是8的倍数, 以确保你遵循了Material Design的建议.

## Resources
- [Build a Responsive UI with ConstraintLayout](https://developer.android.com/training/constraint-layout/index.html)
- Demo: [HelloConstraintLayout](https://github.com/mengdd/HelloConstraintLayout)