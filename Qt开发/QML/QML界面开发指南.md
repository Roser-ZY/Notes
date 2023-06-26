[TOC]

## 一、动画设计

### 1.1 线性动画

#### 1.1.1 从中心向两边展开

##### 方法：利用 *status* 和 *transition*

1. 创建 *status* 和 *transition*。
2. 在源 *status* 中设置 *anchors* 的左右 *margin* 为父组件宽度的一半。
3. 在目的 *status* 中设置 *anchors* 的左右 *margin* 为 `0`。
4. 在 *transition* 中新增属性动画，分别设置 `anchors.rightMargin` 、`anchors.leftMargin` 和 `width`。

使用这种方法效率相比第二种方法稍低。

##### 方法：利用 *animation*



## 二、组件应用

### 2.1 滚动条视图（*ScrollView*）

#### 2.1.1 重点属性

使用时需要注意重点属性的设置：

* `contentWidth` 和 `contentHeight` 如果希望固定为与外部容器相同，则记得绑定到对应的容器，否则为`0`

#### 2.1.2 使用方法

* 内部设置布局组件，布局不需要考虑大小，只是用于承载目标内容，实际会自动变化
