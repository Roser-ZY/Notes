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

