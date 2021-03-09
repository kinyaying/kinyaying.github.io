---
title: 【译】五大原则写出优雅易维护的vue代码
date: 2021-02-09 10:58:06
tags: 算法
keywords:
  - vue3 dom-diff 核心算法
  - dom-diff
  - 最长递增子序列
---

> 本文翻译自：https://betterprogramming.pub/5-principles-for-writing-clean-and-maintainable-vue-js-code-35dfcf5ef08c

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/680f207807a24fb2a3865302e31a4a3a~tplv-k3u1fbpfcp-watermark.image)

在编码过程中，让编写的代码清晰易懂是一件重要的事情。清晰的代码就像一瓶好酒，没法立刻显示出优势，但是随着时间的推移仍能让读它的人觉得很好理解，这便是最大的优势。

而有些代码写的低内聚，逻辑混乱，就像意大利面条一样混乱。你在当时能够理清混乱的逻辑，但是等过 3 个月，5 个月或者 20 个月后再去看这段代码，情况就会很糟糕。

作为一个程序员，我在编程（包括 vue 项目）时会坚持一个重要原则，我把它称为 DRY 原则。

DRY 是*don’t repeat yourself*的缩写，也就是说要减少编程中的重复逻辑。

<!-- more -->

坚持 DRY 原则的原因是当编写同一段逻辑两次以上，会产生更加复杂的情况。举个简单的例子：

你在你的代码中复制了一段代码到不同地方，当你或者其他程序员想重构这段逻辑时你们该怎么做？你们不得不重构两个地方。这又会导致多个新问题：

- 你只改了一个地方，忘记改另一个
- 你修改了这段代码，并改动了每个被复制过的地方。但是你发现一共改了 5 处。
- 你已经 5 个月没写这个项目了，然后突然要回来改这段代码。你还记得这段代码被复制到多个地方吗？

如果你编写的代码足够清晰就能够避免很多问题，同时节约大量时间。短期内不一定很有效果，但是长期看来逻辑清晰的代码能够提高编码效率。

# 使用 Mixins

在 Vue 中复用方法，数据属性，计算属性等的主要方式是 Mixins。

如果你在不同组件间有可复用的逻辑，最好把这段逻辑提取出来，然后通过 mixins 来共享。

假设你的项目有多个路由，并且使用 vue-router 将每个路由与对应的组件联系起来。

```
- App.vue
-- Home.vue
-- Analytics.vue
-- Reports.vue
```

假设现在有一个产品导览组件，你希望在所有页面都拥有这个组件。

如果你不使用 mixins，但是希望产品导览出现在所有页面上，则需要将相同的代码复制到每个页面组件的`mounted`，还需要给每个页面组件添加`startProductTour`方法。 我在此处编写了 Home.vue 示例，然后你需要往 Analytics.vue 和 Reports.vue 里复制代码。

**Home.vue**

```vue
export default { mounted() { this.startProductTour(); }. methods: {
startProductTour() { // start product tour actions here } } }
```

但是如果将这段逻辑抽离出来，并使用 mixin...

`**ProductTour**` **mixin**

```js
export default {
  mounted() {
    this.startProductTour();
  }.
  methods: {
    startProductTour() {
      // start product tour actions here
    }
  }
}
```

…只要一行代码，您就可以在所有组件中使用它。 而且，如果你更改代码，也只需更新一次 mixin，而无需编辑多个文件。

**Home.vue**

```vue
import ProductTourMixin from '../mixins/ProductTourMixin'; export default {
mixins: [ProductTourMixin] }
```

在这种情况下，您的新项目结构将如下所示：

```
- App.vue
- mixins/ProductTourMixin
-- Home.vue
-- Analytics.vue
-- Reports.vue
```

通过这个例子可以证明，使用 mixin，能够让代码逻辑更加清晰，并且遵循 DRY 原则。这能够让你节省时间和精力，避免处理头疼的问题

# 尽量拆分 Components

当我第一次用 vue 写代码时，我的一个 vue 组件有数百行到上万行代码。这个巨大的组件里包含很多逻辑，有巨大的 HTML`<template>`模板，有 js 方法函数。这会造成这几个问题：

- 我写的代码没有可复用性。
- 这段代码随着时间的推移会越来越难阅读和难维护
- 和同事合作很难分工，容易产生冲突。

这让我确信一点——创建一个或几个大型组件而不是许多小型组件绝对是一场噩梦。

那么，创建许多小的组件而不是更少的大型组件有什么好处？

1. 小型组件通常是可复用的，既节省了时间，又避免了重复的代码。
2. 在大多数情况下，您是将父元素和子元素逻辑分离。 如果您更改子组件，则不会影响父元素。 你修改父元素也同样如此，只要你不修改父元素向子元素传递的 props。
3. 较小的组件在大型 v-for 循环中具有性能优势，因为除非 props 更改了（非对象类型的 props），否则 Vue 不会更新它们。

> 对于大型项目而言，有必要把整个项目拆分成多个组件，能让开发过程变得更易管理。

这段话引用自[Vue 官方文档](https://vuejs.org/v2/guide/#Composing-with-Components) ，它说明将项目拆分成组件的重要性。

**提示：**阅读你最喜欢的框架的技术文档可以学到大量的信息，因此我建议你经常阅读。

# 给 Props 增加 Validate 校验

校验 props 是能够让你的 Vue 程序变得更加易维护。

让我们来看一下 props 的错误用法：

```js
export default {
  props: ['myProp'],
}
```

这种定义 props 的方式仅在开发期间可以，但是在上线之前，你应该为 props 编写完整的定义。 至少，你应该为 props 定义一种类型。

```js
// quick way
export default {
  props: {
    myProp: String,
  },
}
// this should be preferred
export default {
  props: {
    myProp: {
      type: String,
    },
  },
}
```

但是最好的方式是对 props 有完整的定义并增加校验逻辑。

```js
props: {
  myProp: {
    type: Number,
    required: true,
    validator: (value) => {
      return value > 0;
    }
  }
}
```

如果编写了 props 的校验逻辑，当传入预期之外的值，此校验逻辑可以帮你处理这种意外情况。

还有一项优势是，通过定义明确的 props，你能够查看 props 的定义和校验逻辑，来了解期望的值。可以将 props 视为我们了解这个组件的小型文档。 在编码时，定义好的入参规则，准确的命名这些语义化的代码比写注释更好。

再强调一遍，逻辑清晰的代码胜过混乱的代码，能够让你随时记住 props 的类型，减少编码过程中产生错误。

# 将视图层和数据层解耦

Vue 不像 Angular，不提供数据层的逻辑，你需要自己编写数据层向后台的接口请求数据并返回给 Vue 进行渲染。

以我的经验来看，应该尽量将视图层和数据层分离。

为了更好地说明我的观点，我将举一个例子。 假设你有一个`Reports`组件，它包含三个子组件：`ReportTable`，`ReportChart`和`ReportStats`。

**Reports.vue**

```js
<!-- This example is intentionally kept simple -->
<template>
  <report-stats :stats="stats" />
  <report-chart :chartData="chartData" />
  <report-table :breakdown="breakdown" />
</template>
<script>
export default {
  data() {
    return {
      stats: null,
      chartData: null,
      breakdown: null
    }
  }
}
</script>
```

这是你的`Reports`组件的基础结构。 现在，需要从 API 中获取数据，因此你将写出以下代码：

```vue
mounted() { this.getStats(); }
```

接着要编写 getStats：

```js
methods: {
  getStats() {
    axios.get('/api/to-get/the-data')
      .then(response => {
        this.stats = response.data.stats
      });
  }
}
```

现在看来还不错，在较小的项目中，这么处理可能还不错。 但是，将数据层和视图层分开有很多好处：

- 接口层有可能在后端环境中被复用。或者当前端不用 Vue 项目，用其他框架搭建时，接口层仍然被保留。

- 能够帮你保持 Vue 代码逻辑清晰，易于理解。 因为视图层不需要知道数据层的复杂逻辑。

- 拥有一个定义明确的 API 层，有点像预控制器，可以帮助你和你的团队了解需要这些接口需要哪些参数以及每个接口是如何被调用的。

那么，我们要如何改写我们的代码？

**ReportsService.js**

```js
const ReportsService = {
  stats: {
    index: async (/* params */) => {
      const response = await axios.get('/api-link')
      return response.data
    },
  },
}
export default ReportsService
```

**Reports.vue**

```vue
import ReportsService from '../services/ReportsService'; export default {
methods: { async getStats() { this.stats = await ReportsService.stats.index(/*
params */); } } }
```

我们使用简单的逻辑创建了一个服务，该服务能够与我们的后端 API 进行交互或实现数据层的逻辑。 然后，我们的 Vue 组件无需了解有关我们的 API 或数据层的更多信息，直接通过我们创建的服务获取所需的数据。

# 按照 Vue 风格指南进行编码

深入讨论 vue 的风格指南并不在这篇文章的范围内。但是，对开发者来说，学习框架的设计原理和框架希望你如何使用它也很重要。

框架的创造者会制定一些标准的编码规范，社区成员在使用框架的过程中要遵循这些准则。

有些重要的编码规范列举如下：

## 使用 data()函数，而不是 data 对象

反例：

```js
export default {
  data: {
    value: 1,
  },
}
```

好例子：

```js
export default {
  data() {
    return {
      value: 1,
    }
  },
}
```

为什么要用 data()函数，而不是 date 对象？当 `data` 的值是一个对象时，它会在这个组件的所有实例之间共享。当组件被 A 和 B 复用时，在 A 中修改了组件的数据，会影响到 B 组件。

## 给 v-for 增加 key

使用 v-for 时需要增加 key，因为这样能帮助 vue 在 dom-diff 时识别出未被修改的组件，并复用该组件，从而提高渲染效率。

```js
<my-item
  v-for="item in items"
  :key="item.id"
  :item="item" />
<!-- or with HTMl elements -->
<div
  v-for="item in items"
  :key="item.id"
>
  {{ item.value }}
</div>
```

## 避免 v-if 和 v-for 用在一起

[vue 文档](https://cn.vuejs.org/v2/style-guide/index.html#%E9%81%BF%E5%85%8D-v-if-%E5%92%8C-v-for-%E7%94%A8%E5%9C%A8%E4%B8%80%E8%B5%B7%E5%BF%85%E8%A6%81)明确的给出了原因。不这么做是因为 Vue 在渲染时只渲染出一小部分数据，也得在每次重渲染的时候遍历整个列表。

举个例子来证明一下：

反例：

```js
<div v-for="item in items" v-if="item.shouldShow()">
  {{ item.value }}
</div>
```

好的例子：

```js
<div v-for="item in visibleItems">
    {{ item.value }}
</div>
```

并且在`compued`中增加下属性：

```js
visibleItems() {
  return this.items.filter(item => {
    return item.shouldShow();
  });
}
```

这么做的好处有这几点：

1. 计算属性`visibleItems` 只有当`items`发生变化时才会重新计算。
2. `v-for`只处理`visibleItems`而不是整个`items`数组，这让渲染更加高效。
3. 模板内的逻辑更加简洁清晰。解耦渲染层的罗杰，让代码的可维护性更强。

如果想了解更多的编码风格，你可以阅读 [official section of the Vue documentation](https://vuejs.org/v2/style-guide/) 这篇文档。哪怕你已经是一个有经验的 Vue 开发者，你仍然能从中学到很多内容。

# 结论

编写清晰易懂的代码会让你的同事和你的合作过程更加愉快。不用再维护大量的像意大利面一样逻辑混乱的代码，对你来说也是一件幸事。

如果你在做 codeview，你可以按照上述的 5 个原则来作为对代码的评价依据，如此能让你的团队贡献高质量，高效率的代码。

感谢阅读本文！
