# VUE 开发规范

> 不管有多少人共同参与同一项目，一定要确保每一行代码都像是同一个人编写的

## 环境配置

1. [Node.js](https://nodejs.org/) 14.20 或更高版本

2. 使用 [Visual Studio Code (VS Code)](http://www.baidu.com/link?url=Sn0aBa011utoLAP3IC799QxEcwe1-mmChrkEchQsAaOJJwwbN8ZMTyf8XF8uDId9) 进行代码编写

3. 代码提交前使用 VS Code 进行格式化（不要格式引入的外部文件）

4. 规定 Tab 大小为 2 个空格，保证在所有环境下获得一致展现

   ```javascript
   // settings.json
   {
     "editor.tabSize": 2
     // ...
   }
   ```

5. 安装插件 [Vetur](https://marketplace.visualstudio.com/items?itemName=octref.vetur)（ Vue开发扩展及 Vue 文件代码格式化）

6. 安装插件 [Prettier - Code formatter](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)（ CSS / Less / JS 等其他文件代码格式化；Vetur 的格式化基于此插件实现，固可以在所有文件实现统一的格式化）

7. 安装插件 [Chinese (Simplified) Language Pack for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=MS-CEINTL.vscode-language-pack-zh-hans) （ VS Code 简体中文语言包）

8. 使用 Chrome 浏览器进行调试


## 代码规范

参考 RiotJS 编码规范而写。

### 目录

* [基于模块开发](#基于模块开发)
* [vue 组件命名](#vue-组件命名)
* [组件表达式简单化](#组件表达式简单化)
* [组件 props 原子化](#组件-props-原子化)
* [验证组件的 props](#验证组件的-props)
* [将`this` 赋值给 `component` 变量](#将-this-赋值给-component-变量)
* [组件结构化](#组件结构化)
* [组件事件命名](#组件事件命名)
* [使用组件名作为样式作用域空间](#使用组件名作为样式作用域空间)
* [只在需要时创建组件](#只在需要时创建组件)
* [尽可能使用`mixins`](#尽可能使用-mixins)
* [尽量避免使用`this.$parent`](#尽量避免使用-thisparent)
* [尽量避免使用`this.$refs`](#尽量避免使用-thisrefs)
* [尽量避免使用`setTimeout`](#尽量避免使用-settimeout)

### 基于模块开发

始终基于模块的方式来构建你的 app，每一个子模块只做一件事情。

Vue.js 的设计初衷就是帮助开发者更好的开发界面模块。一个模块是应用程序中独立的一个部分。

**怎么做？**

每一个 Vue 组件（等同于模块）[首先]((https://addyosmani.com/first/))必须专注于解决一个[单一的问题](http://en.wikipedia.org/wiki/Single_responsibility_principle)，*独立的*、*可复用的*、*微小的* 和 *可测试的*。

如果你的组件做了太多的事或是变得臃肿，请将其拆分成更小的组件并保持单一的原则。一般来说，尽量保证每一个文件的代码行数不要超过 100 行。也请保证组件可独立的运行。比较好的做法是增加一个单独的 demo 示例。

[↑ 回到目录](#目录)

### Vue 组件命名

组件的命名需遵从以下原则：

* **有意义的**: 不过于具体，也不过于抽象
* **简短**: 2 到 3 个单词
* **具有可读性**: 以便于沟通交流

同时还需要注意：

* 必须符合**自定义元素规范**: [使用连字符](https://www.w3.org/TR/custom-elements/#concepts)分隔单词，切勿使用保留字。
* **`app-` 前缀作为命名空间**: 如果非常通用的话可使用一个单词来命名，这样可以方便于其它项目里复用。

**为什么？**

* 组件是通过组件名来调用的。所以组件名必须简短、富有含义并且具有可读性。

**如何做？**

```html
<!-- 推荐 -->
<app-header></app-header>
<user-list></user-list>
<range-slider></range-slider>

<!-- 避免 -->
<btn-group></btn-group> <!-- 虽然简短但是可读性差. 使用 `button-group` 替代 -->
<ui-slider></ui-slider> <!-- ui 前缀太过于宽泛，在这里意义不明确 -->
<slider></slider> <!-- 与自定义元素规范不兼容 -->
```

[↑ 回到目录](#目录)

### 组件表达式简单化

Vue.js 的表达式是 100% 的 Javascript 表达式。这使得其功能性很强大，但也带来潜在的复杂性。因此，你应该尽量**保持表达式的简单化**。

**为什么？**

* 复杂的行内表达式难以阅读。
* 行内表达式是不能够通用的，这可能会导致重复编码的问题。
* IDE 基本上不能识别行内表达式语法，所以使用行内表达式 IDE 不能提供自动补全和语法校验功能。

**怎么做？**

如果你发现写了太多复杂并难以阅读的行内表达式，那么可以使用 method 或是 computed 属性来替代其功能。

```html
<!-- 推荐 -->
<template>
  <h1>
    {{ `${year}-${month}` }}
  </h1>
</template>
<script type="text/javascript">
  export default {
    computed: {
      month() {
        return this.twoDigits((new Date()).getUTCMonth() + 1);
      },
      year() {
        return (new Date()).getUTCFullYear();
      }
    },
    methods: {
      twoDigits(num) {
        return ('0' + num).slice(-2);
      }
    },
  };
</script>

<!-- 避免 -->
<template>
  <h1>
    {{ `${(new Date()).getUTCFullYear()}-${('0' + ((new Date()).getUTCMonth()+1)).slice(-2)}` }}
  </h1>
</template>
```

[↑ 回到目录](#目录)

#### 组件 props 原子化

虽然 Vue.js 支持传递复杂的 JavaScript 对象通过 props 属性，但是你应该尽可能的使用原始类型的数据。尽量只使用 [JavaScript 原始类型](https://developer.mozilla.org/en-US/docs/Glossary/Primitive)（字符串、数字、布尔值）和函数。尽量避免复杂的对象。

**为什么？**

* 使得组件 API 清晰直观。
* 只使用原始类型和函数作为 props 使得组件的 API 更接近于 HTML(5) 原生元素。
* 其它开发者更好的理解每一个 prop 的含义、作用。
* 传递过于复杂的对象使得我们不能够清楚的知道哪些属性或方法被自定义组件使用，这使得代码难以重构和维护。

**怎么做？**

组件的每一个属性单独使用一个 props，并且使用函数或是原始类型的值。

```html
<!-- 推荐 -->
<range-slider
  :values="[10, 20]"
  :min="0"
  :max="100"
  :step="5"
  @on-slide="updateInputs"
  @on-end="updateResults">
</range-slider>

<!-- 避免 -->
<range-slider :config="complexConfigObject"></range-slider>
```

[↑ 回到目录](#目录)

### 验证组件的 props

在 Vue.js 中，组件的 props 即 API，一个稳定并可预测的 API 会使得你的组件更容易被其他开发者使用。

组件 props 通过自定义标签的属性来传递。属性的值可以是 Vue.js 字符串(`:attr="value"` 或 `v-bind:attr="value"`)或是不传。你需要保证组件的 props 能应对不同的情况。

**为什么？**

验证组件 props 可以保证你的组件永远是可用的（防御性编程）。即使其他开发者并未按照你预想的方法使用时也不会出错。

**怎么做？**

* 提供默认值。
* 使用 `type` 属性[校验类型](http://vuejs.org/v2/guide/components.html#Prop-Validation)。
* 使用 props 之前先检查该 prop 是否存在。

```html
<template>
  <input type="range" v-model="value" :max="max" :min="min">
</template>
<script type="text/javascript">
  export default {
    props: {
      max: {
        type: Number, // 这里添加了数字类型的校验
        default() { return 10; },
      },
      min: {
        type: Number,
        default() { return 0; },
      },
      value: {
        type: Number,
        default() { return 4; },
      },
    },
  };
</script>
```

[↑ 回到目录](#目录)

### 将 `this` 赋值给 `component` 变量

在 Vue.js 组件上下文中，`this`指向了组件实例。因此当你切换到了不同的上下文时，要确保 `this` 指向一个可用的 `component` 变量。

换句话说，如果你正在使用 **ES6** 的话，就不要再编写 `var self = this;` 这样的代码了，您可以安全地使用 Vue 组件。

**为什么？**

* 使用 **ES6**，就不再需要将 `this` 保存到一个变量中了。
* 一般来说，当你使用箭头函数时，会保留 `this` 的作用域。（译者注：箭头函数没有它自己的 this 值，箭头函数内的 this 值继承自外围作用域。）
* 如果你没有使用 **ES6**，当然也就不会使用 `箭头函数` 啦，那你必须将 “this” 保存到到某个变量中。这是唯一的例外。

**怎么做？**

```html
<script type="text/javascript">
export default {
  methods: {
    hello() {
      return 'hello';
    },
    printHello() {
      console.log(this.hello());
    },
  },
};
</script>

<!-- 避免 -->
<script type="text/javascript">
export default {
  methods: {
    hello() {
      return 'hello';
    },
    printHello() {
      const self = this; // 没有必要
      console.log(self.hello());
    },
  },
};
</script>
```

[↑ 回到目录](#目录)

### 组件结构化

按照一定的结构组织，使得组件便于理解。

**为什么？**

* 导出一个清晰、组织有序的组件，使得代码易于阅读和理解。同时也便于标准化。
* 按首字母排序 properties、data、computed、watches 和 methods 使得这些对象内的属性便于查找。
* 合理组织，使得组件易于阅读。（name; extends; props, data 和 computed; components; watch 和 methods; lifecycle methods 等）。
* 使用 `name` 属性。借助于 [vue devtools](https://chrome.google.com/webstore/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd?hl=en) 可以让你更方便的测试。
* 合理的 CSS 结构，如 [BEM](https://medium.com/tldr-tech/bem-blocks-elements-and-modifiers-6b3b0af9e3ea#.bhnomd7gw) 或 [rscss](https://github.com/rstacruz/rscss) - [详情？](#使用组件名作为样式作用域空间)。
* 使用单文件 .vue 文件格式来组件代码。

**怎么做？**

组件结构化

```html
<template lang="html">
  <div class="Ranger__Wrapper">
    <!-- ... -->
  </div>
</template>

<script type="text/javascript">
  export default {
    // 不要忘记了 name 属性
    name: 'RangeSlider',
    // 使用组件 mixins 共享通用功能
    mixins: [],
    // 组成新的组件
    extends: {},
    // 组件属性、变量
    props: {
      bar: {}, // 按字母顺序
      foo: {},
      fooBar: {},
    },
    // 变量
    data() {},
    computed: {},
    // 使用其它组件
    components: {},
    // 方法
    watch: {},
    methods: {},
    // 生命周期函数
    beforeCreate() {},
    mounted() {},
  };
</script>

<style scoped>
  .Ranger__Wrapper { /* ... */ }
</style>
```

[↑ 回到目录](#目录)

### 组件事件命名

Vue.js 提供的处理函数和表达式都是绑定在 ViewModel 上的，组件的每一个事件都应该按照一个好的命名规范来，这样可以避免不少的开发问题，具体可见如下

**为什么？**

* 开发者可以随意给事件命名，即使是原生事件的名字，这样会带来迷惑性。
* 过于宽松的事件命名可能与 [DOM 模板不兼容](https://vuejs.org/v2/guide/components.html#DOM-Template-Parsing-Caveats)。

**怎么做？**

* 事件名也使用连字符命名。
* 一个事件的名字对应组件外的一组意义操作，如：upload-success、upload-error 以及 dropzone-upload-success、dropzone-upload-error （如果需要前缀的话）。
* 事件命名应该以动词（如 client-api-load） 或是 名词（如 drive-upload-success）结尾。（[出处](https://github.com/GoogleWebComponents/style-guide#events)）


[↑ 回到目录](#目录)


### 使用组件名作为样式作用域空间

Vue.js 的组件是自定义元素，这非常适合用来作为样式的根作用域空间。可以将组件名作为 CSS 类的命名空间。

**为什么？**

* 给样式加上作用域空间可以避免组件样式影响外部的样式。
* 保持模块名、目录名、样式根作用域名一样，可以很好的将其关联起来，便于开发者理解。

**怎么做？**

使用组件名作为样式命名的前缀，可基于 BEM 或 OOCSS 范式。同时给 style 标签加上 scoped 属性。加上 scoped 属性编译后会给组件的 class 自动加上唯一的前缀从而避免样式的冲突。

```html
<style scoped>
  /* 推荐 */
  .MyExample { }
  .MyExample li { }
  .MyExample__item { }

  /* 避免 */
  .My-Example { } /* 没有用组件名或模块名限制作用域, 不符合 BEM 规范 */
</style>
```

[↑ 回到目录](#目录)

### 只在需要时创建组件

**为什么？**

Vue.js 是一个基于组件的框架。如果你不知道何时创建组件可能会导致以下问题：

* 如果组件太大, 可能很难重用和维护;
* 如果组件太小，你的项目就会（因为深层次的嵌套而）被淹没，也更难使组件间通信;

**怎么做?**

* 始终记住为你的项目需求构建你的组件，但是你也应该尝试想到它们能够从中脱颖而出（独立于项目之外）。如果它们能够在你项目之外工作，就像一个库那样，就会使得它们更加健壮和一致。
* 尽可能早地构建你的组件总是更好的，因为这样使得你可以在一个已经存在和稳定的组件上构建你的组件间通信（props & events）。

**规则**

* 首先，尽可能早地尝试构建出诸如模态框、提示框、工具条、菜单、头部等这些明显的（通用型）组件。总之，你知道的这些组件以后一定会在当前页面或者是全局范围内需要。
* 第二，在每一个新的开发项目中，对于一整个页面或者其中的一部分，在进行开发前先尝试思考一下。如果你认为它有一部分应该是一个组件，那么就创建它吧。
* 最后，如果你不确定，那就不要。避免那些“以后可能会有用”的组件污染你的项目。它们可能会永远的只是（静静地）待在那里，这一点也不聪明。注意，一旦你意识到应该这么做，最好是就把它打破，以避免与项目的其他部分构成兼容性和复杂性。

[↑ 回到目录](#目录)

---

### 尽可能使用 mixins

**为什么?**

Mixins 封装可重用的代码，避免了重复。如果两个组件共享有相同的功能，则可以使用 mixin。通过 mixin，你可以专注于单个组件的任务和抽象的通用代码。这有助于更好地维护你的应用程序。

**怎么做?**

假设你有一个移动端和桌面端的菜单组件，它们共享一些功能。我们可以抽象出这两个组件的核心功能到一个 mixin 中，例如：

```js
const MenuMixin = {
  data () {
    return {
      language: 'EN'
    }
  },

  methods: {
    changeLanguage () {
      if (this.language === 'DE') this.$set(this, 'language', 'EN')
      if (this.language === 'EN') this.$set(this, 'language', 'DE')
    }
  }
}

export default MenuMixin
```
要使用 mixin，只需将其导入到两个组件中（我只展示移动组件）。

```html
<template>
  <ul class="mobile">
    <li @click="changeLanguage">Change language</li>
  </ul>
</template>

<script>
  import MenuMixin from './MenuMixin'

  export default {
    mixins: [MenuMixin]
  }
</script>
```

[↑ 回到目录](#目录)

### 尽量避免使用 this.$parent

Vue.js 支持组件嵌套，并且子组件可访问父组件的上下文。访问组件之外的上下文违反了[基于模块开发](#基于模块开发)的[第一原则](https://addyosmani.com/first/)。因此你应该尽量避免使用 **`this.$parent`**。

**为什么？**

* 组件必须相互保持独立，Vue 组件也是。如果组件需要访问其父层的上下文就违反了该原则。
* 如果一个组件需要访问其父组件的上下文，那么该组件将不能在其它上下文中复用。

**怎么做？**

* 通过 props 将值传递给子组件。
* 通过 props 传递回调函数给子组件来达到调用父组件方法的目的。
* 通过在子组件触发事件来通知父组件。

[↑ 回到目录](#目录)

### 尽量避免使用 this.$refs

Vue.js 支持通过 `ref` 属性来访问其它组件和 HTML 元素。并通过 `this.$refs` 可以得到组件或 HTML 元素的上下文。在大多数情况下，通过 `this.$refs`来访问其它组件的上下文是可以避免的。在使用的的时候你需要注意避免调用了不恰当的组件 API，所以应该尽量避免使用 `this.$refs`。

**为什么？**

* 组件必须是保持独立的，如果一个组件的 API 不能够提供所需的功能，那么这个组件在设计、实现上是有问题的。
* 组件的属性和事件必须足够的给大多数的组件使用。

**怎么做？**

* 提供良好的组件 API。
* 总是关注于组件本身的目的。
* 拒绝定制代码。如果你在一个通用的组件内部编写特定需求的代码，那么代表这个组件的 API 不够通用，或者你可能需要一个新的组件来应对该需求。
* 检查所有的 props 是否有缺失的，如果有提一个 issue 或是完善这个组件。
* 检查所有的事件。子组件向父组件通信一般是通过事件来实现的，但是大多数的开发者更多的关注于 props 从忽视了这点。
* **Props向下传递，事件向上传递！**。以此为目标升级你的组件，提供良好的 API 和 独立性。
* 当遇到 props 和 events 难以实现的功能时，通过 `this.$refs`来实现。
* 当需要操作 DOM 无法通过指令来做的时候可使用 `this.$ref` 而不是 `JQuery`、`document.getElement*`、`document.queryElement`。


```html
<!-- 推荐，并未使用 this.$refs -->
<range :max="max"
  :min="min"
  @current-value="currentValue"
  :step="1"></range>
```

```html
<!-- 使用 this.$refs 的适用情况-->
<modal ref="basicModal">
  <h4>Basic Modal</h4>
  <button class="primary" @click="$refs.basicModal.hide()">Close</button>
</modal>
<button @click="$refs.basicModal.open()">Open modal</button>

<!-- Modal component -->
<template>
  <div v-show="active">
    <!-- ... -->
  </div>
</template>

<script>
  export default {
    // ...
    data() {
      return {
        active: false,
      };
    },
    methods: {
      open() {
        this.active = true;
      },
      hide() {
        this.active = false;
      },
    },
    // ...
  };
</script>

```

```html
<!-- 如果可通过 emited 来做则避免通过 this.$refs 直接访问 -->
<template>
  <range :max="max"
    :min="min"
    ref="range"
    :step="1"></range>
</template>

<script>
  export default {
    // ...
    methods: {
      getRangeCurrentValue() {
        return this.$refs.range.currentValue;
      },
    },
    // ...
  };
</script>
```

[↑ 回到目录](#目录)

### 尽量避免使用 setTimeout

**setTimeout运行机制**

* setTimeout和setInterval的运行机制是，将指定的代码移出本次执行，等到下一轮Event Loop时，再检查是否到了指定时间。
* 如果到了，就执行对应的代码；如果不到，就等到再下一轮Event Loop时重新判断。
* 这意味着，setTimeout指定的代码，必须等到本次执行的所有代码都执行完，才会执行。
* 每一轮Event Loop时，都会将“任务队列”中需要执行的任务，一次执行完。
* setTimeout和setInterval都是把任务添加到“任务队列”的尾部。
* 因此，它们实际上要等到当前脚本的所有同步任务执行完，然后再等到本次Event Loop的“任务队列”的所有任务执行完，才会开始执行。
* 由于前面的任务到底需要多少时间执行完，是不确定的，所以没有办法保证，setTimeout和setInterval指定的任务，一定会按照预定时间执行。

**使用 nextTick 代替**

* Vue 在内部对异步队列尝试使用原生的 Promise.then、MutationObserver 和 setImmediate
* 如果执行环境不支持，则会采用 setTimeout(fn, 0) 代替

**为什么使用？**

* 在Vue生命周期的created()钩子函数进行的DOM操作一定要放在Vue.nextTick()的回调函数中
* 在created()钩子函数执行的时候DOM 其实并未进行任何渲染，而此时进行DOM操作无异于徒劳，所以此处一定要将DOM操作的js代码放进Vue.nextTick()的回调函数中。
* 与之对应的就是mounted()钩子函数，因为该钩子函数执行时所有的DOM挂载和渲染都已完成，此时在该钩子函数中进行任何DOM操作都不会有问题 。
* 在数据变化后要执行的某个操作，而这个操作需要使用随数据改变而改变的DOM结构的时候，这个操作都应该放进Vue.nextTick()的回调函数中。

查看 [JAVASCRIPT](JAVASCRIPT.md) 执行机制可了解更多底层原理

[↑ 回到目录](#目录)