---
title: 解析 Vue3.0 的 dom-diff 核心算法——最长递增子序列
date: 2021-02-09 13:58:06
tags: 算法
keywords:
  - vue3 dom-diff 核心算法
  - dom-diff
  - 最长递增子序列
---

# 一、基础背景

去年 Vue3.0 正式版本推出，受到很多人的追捧。vue3.0 中也对 dom-diff 算法进行了优化，其中就用到了 **「最长递增子序列」**。

先简要介绍下基础背景。我们在 vue 开发项目的时候，常用模板或者 jsx 语法来编写 DOM。实际上我们编写的代码会被`@vue/compiler-dom`转化为虚拟 DOM 节点，即 Virtual DOM，之后再将虚拟 DOM 节点渲染成实际的 DOM 节点，Virtual DOM 也会被组织成树形结构，即 Virtual DOM 树。类似如下所示 👇🏽：
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/78240500087d4536b74286e30aff5b4c~tplv-k3u1fbpfcp-watermark.image)

<!-- more -->

也许你会有个问题，为什么要引入虚拟 DOM，而不是直接把组件转换成正式 DOM 节点呢？在我看来有两个优点：

1. **更高效：** 当组件发生变化时，利用虚拟 DOM 树可以对新旧 DOM 进行比较，找出区别，只更新发生变化的 DOM，让渲染更加高效。
2. **更灵活：** 将原来直接渲染变成先构建虚拟 DOM，再进行渲染。相当于抽离出中间层，对应的渲染层可以是针对浏览器的渲染，也可以是针对小程序等其他平台的渲染。

根据上面的介绍，我们知道 Vue 会把我们编写的组件转换成虚拟 DOM 树，并且将虚拟 DOM 树进行比较后再根据变化情况更新真实 DOM。比如原有列表为 `[a, b, c, d, e, f]` ，而新列表为 `[a, d, b, c, e, f]`， 这时会这样进行 diff：

1. **去除相同前置和后置元素** ，此优化由 [Neil Fraser](https://neil.fraser.name/writing/diff/) 提出，可以比较容易实现而且带来带来比较明显的提升；

   比如针对上情况，去除相同的前置和后置元素后，真正需要处理的是 `[ b, c, d]` 和 `[d, b, c]` ，复杂性会大大降低。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/64b6fce49042461b9c75213ed74f1dd5~tplv-k3u1fbpfcp-watermark.image)

2. **最长递增子序列**

   接着要将原数组中的`[ b, c, d]` 转化成 `[d, b, c]` 。Vue3 中对移动次数进行了进一步的优化。下面对这个算法进行介绍：

   1. 首先遍历新列表，通过 key 去查找在原有列表中的位置，从而得到新列表在原有列表中位置所构成的数组。比如原数组中的`[ b, c, d]`， 新数组为 `[d, b, c]` ，得到的位置数组为 `[3, 1, 2]` ，现在的算法就是通过位置数组判断最小化移动次数；

   2. 计算最长递增子序列

      最长递增子序列是经典的动态规划算法，不了解的可以前往 [最长递增子序列](https://baike.baidu.com/item/最长递增子序列/22828111#1) 去补充一下前序知识。那么为什么最长递增子序列就可以保证移动次数最少呢？因为在位置数组中递增就能保证在旧数组中的相对位置的有序性，从而不需要移动，因此递增子序列的最长可以保证移动次数的最少

      对于前面的得到的位置数组`[3, 1, 2]`，得到最长递增子序列 `[1, 2]` ，在子序列内的元素不移动，不在此子序列的元素移动即可。对应的实际的节点即 `d` 节点移动至`b, c`前面即可。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ef1b8d1d19d1489fb0c50dbbed2dabfb~tplv-k3u1fbpfcp-watermark.image)

本文就将介绍最长递增子序列的算法。大家可以配合着 Vue3 中的源码来阅读，希望读完这篇文章后能够更加理解 Vue3 中关于最长递增子序列的实现。

Vue3.0 的源码 👉：[vue-next/renderer.ts](https://github.com/vuejs/vue-next/blob/540e26f49c09edf09b6a60ac2a978fdec52686bf/packages/runtime-core/src/renderer.ts) 请阅读其中的`getSequence`方法。

力扣上没有完全一样的题目，有一道最接近的题 [300. 最长递增子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/) ，它的要求是求最长递增子序列的长度，我们可以把题目换成求最长递增子序列的索引。

# 二、题目描述：

给你一个整数数组 `nums` ，找到其中最长严格递增子序列的长度。

子序列是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，[3,6,2,7] 是数组 [0,3,1,6,2,2,7] 的子序列。

**示例 1：**

> 输入：nums = [10,9,2,5,3,7,101,18]
> 输出：[ 2, 4, 5, 7 ]
> 解释：最长递增子序列是 [2,3,7,18]，因此索引是[ 2, 4, 5, 7 ]。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d8fe1a74e0934fc1b8a83d2416ce866d~tplv-k3u1fbpfcp-watermark.image)

这里再强调一遍，我们要求的是最长子序列对应的索引。

**示例 2：**

> 输入：nums = [0,1,0,3,2,3]
> 输出：[ 0, 1, 4, 5 ]
>
> 解释：最长递增子序列是 [0,1,2,3]，因此索引是[ 0,1,4,5 ]。

**示例 3：**

> 输入：nums = [7,7,7,7,7,7,7]
> 输出：[0]
>
> 解释：最长递增子序列是 [7]，因此索引是[0]。

敲黑板，👩‍🏫，我们要求的是最长递增子序列的索引。ok，强调完了，开始解题吧。

# 三、思路分析：

假设我们要实现 getSequence 方法，入参是 nums 数组，返回结果是一个数组。

```js
/**
 * @param {number[]} nums
 * @return {number[]}
 */
var getSequence = function (nums) {
  // your code
}
```

步骤 1. 先创建一个空数组 result 保存索引。遍历 nums，将当前项`current`和 result 的最后一项对应的值`last`进行比较。如果当前项大于最后一项，直接往 result 中新增一项；否则，针对 result 数组进行二分查找，找到并替换比当前项大的那项。下图示意图中为了方便理解 result 存放的是 nums 中的值，实际代码存放的是数组索引。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/daa7150e1a3040ad8359112ae01fa23f~tplv-k3u1fbpfcp-watermark.image)
步骤 2. 这步是难点，因为步骤 1 在替换的过程中`贪心`了，导致最后的结果错乱。

为了解决这个问题，使用的`前驱节点`的概念，需要再创建一个数组 preIndexArr。在步骤 1 往 result 中新增或者替换新值的时候，同时 preIndexArr 新增一项，该项为当前项对应的前一项的索引。这样我们有了两个数组：

- result：[1,3,4,6,7,9]

- preIndexArr：[undefined,0,undefined,1,3,4,4,6,1]

result 的结果是不准确的，但是 result 的最后一项是正确的，因为最后一项是最大的，最大的不会算错。我们可知最大一项是值 9，索引是 7。可查询`preIndexArr[7]`获得 9 的前一项的索引为 6，值为 7...依次类推能够重建新的 result。

注意：下图中为了方便理解，result 存放的是值，实际代码中存放的是索引。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/784259c30746464c98481a9270350fa6~tplv-k3u1fbpfcp-watermark.image)
ok，思路说完了，那么下面就开始码代码了 🧑‍💻

## 步骤 1

```js
var getSequence1 = function (nums) {
  let result = []
  for (let i = 0; i < nums.length; i++) {
    let last = nums[result[result.length - 1]],
      current = nums[i]
    if (current > last || last === undefined) {
      // 当前项大于最后一项
      result.push(i)
    } else {
      // 当前项小于最后一项，二分查找+替换
      let start = 0,
        end = result.length - 1,
        middle
      while (start < end) {
        middle = Math.floor((start + end) / 2)
        if (nums[result[middle]] > current) {
          end = middle
        } else {
          start = middle + 1
        }
      }
      result[start] = i
    }
  }
  return result
}
```

## 步骤 2

```js
var getSequence = function (nums) {
  let result = [],
    preIndex = []
  for (let i = 0; i < nums.length; i++) {
    let last = nums[result[result.length - 1]],
      current = nums[i]
    if (current > last || last === undefined) {
      // 当前项大于最后一项
      preIndex[i] = result[result.length - 1]
      result.push(i)
    } else {
      // 当前项小于最后一项，二分查找+替换
      let start = 0,
        end = result.length - 1,
        middle
      while (start < end) {
        // 重合就说明找到了 对应的值,时间复杂度O(logn)
        middle = Math.floor((start + end) / 2) // 找到中间位置的前一个
        if (nums[result[middle]] > current) {
          end = middle
        } else {
          start = middle + 1
        }
      }

      // 如果相同 或者 比当前的还大就不换了
      if (current < nums[result[start]]) {
        preIndex[i] = result[start - 1] // 要将他替换的前一个记住
        result[start] = i
      }
    }
  }
  // 利用前驱节点重新计算result
  let length = result.length, //总长度
    prev = result[length - 1] // 最后一项
  while (length-- > 0) {
    // 根据前驱节点一个个向前查找
    result[length] = prev
    prev = preIndex[result[length]]
  }
  return result
}
```

**小结：**

- 本题目用到的核心思想有：贪心，二分查找，前驱节点

- 这个题解的时间复杂度为`O(nlogn)`，即 for 循环的`n`乘以二分查找的`logn`

# 四、完整代码：

在 **三、思路分析**的步骤 2 已给出完整代码

# 五、总结：

本文介绍了[300. 最长递增子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/)的其中一种解题思路，核心思想用到了**贪心，二分查找和前驱节点**。

vue3.0 中核心 dom-diff 算法用到了最长递增子序列，学习掌握此题的解法有助于我们更好的了解 vue3.0 的 dom-diff。

感谢阅读~
