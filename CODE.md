# 业务代码参考

**地图的业务代码参考**

```html
project  // 文件目录
│   index.html
│   package.json
└───src
    │   App.vue
    │   main.js
    │───tools
    │    │   config.js  共享变量
    │    │   viewer.js  图层入口
    │    │   events.js  图层交互
    │    │   rodtower.js  杆塔加载
    │    │   ...   
    └───views
         │───components 组件目录
         │   index.vue 视图页面
         │   ...   
```

## 业务代码解耦

1. tools 业务代码目录
2. views 项目视图目录

> 当业务复杂到一定程度时，就需要将业务代码拆分。
* 简单的拆分就是将视图与业务解耦
```js
// 共享变量 
const Config = {
    // 场景对象
    viewer: null,
}
```
```js
// 场景 
const Viewer = {
    // 初始化SDK
    init: () => ZTTGIS.ready(Viewer.ready),
    ready: () => {
        Config.viewer = new ZTTGIS.Viewer("viewer-container");
        ...
        // 业务对象初始化
        Rodtowers.init()
    },
    // 清除
    clear: () => {
        ...
        Rodtowers.clear()
    },
    // 销毁
    destroy: () => {
        if(Config.viewer){
            Rodtowers.destroy();
            Config.viewer.destroy();
        }
        Config.viewer = null;
    }
}

```
```js
// index.vue
// 视图中只需要调用场景对象
// 视图初始化
onMounted(() => {
  Viewer.init();
});
// 视图销毁
onUnmounted(() => {
  Viewer.destroy();
});
...

```
> 业务代码需要遵循 index.vue -> Viewer.js -> Rodtowers.js , 也就是视图引用场景，场景引用业务对象

```js
// 场景
const Viewer = {
    ready: () => {
        ...
        Config.listener = new Listener();
        // 清除选中状态  例如：杆塔点击时切换选中状态，其他对象需要清除选中对象
        Config.listener.on("ClearGroup", Viewer.clearGroup, Viewer);
    },
    clearGroup: () => {
        // 这里会处理所有业务对象
        Rodtowers.clearGroupTower();
        ...
    },
}
```
```js
// 杆塔
const Rodtowers = {
    loadTower: () => {
        const model = new ZTTGIS.RodtowerPrimitive(item, Config.viewer);
        // 旋转
        model.startMove = (e) => {
            // 触发场景中 清除选中状态
            Config.listener.fire("ClearGroup", null);
            Config.tooltip = "拖动杆塔移动位置";
        };
    },
    ...
}
```
> 如果某个业务对象需要操作其他业务对象，或者说某个业务对象会影响到其他业务对象，请使用事件监听；

## 业务代码的分类
* 业务对象分为图层对象和辅助对象
 ```js
// 当视图层与业务层 产生数据交互
// 交互  
const Events = {
    // 历史记录
    historys: {},
    selectRodtower: (e) => {
        //接口调用
    },
    updateRodtower: (e) => {
        //接口调用
    },
    romveRodtower: (e) => {
        //接口调用
    },
    ...
}
```  

> 包含视图的初始化，例如属性面板；以及属性面板中属性值的修改

```js
// index.vue
// 视图中属性面板监听事件

// 属性更改
const onDrawChnage = (e) => {
  Events.changeDraw(e); //严谨一点 这里应该使用 Viewer， 在Viewer中 调用Events (偷懒了)
};
...
```
* 图层对象的分类
```js
// 杆塔
const Rodtowers = {
    // 杆塔图层
    primitive: null,
    // 线段图层
    vector: null,
    // 初始化  主要用来初始化图层
    init: () => {},
    // 加载   加载已有数据
    load: () => {},
    // 清除  清除图层相关对象
    clear: () => {},
    // 销毁  销毁图层
    destroy: () => {},
    ...
}
```
> 每个图层对象都包含固有的方法和属性，图层，初始化，加载，清除，销毁；当业务继续发展达到一定复杂程度时，可以进一步将对象抽象化进行封装

## 完美的业务代码
将业务代码解耦到一定程度，如果更换技术栈只需要做简单的修改，甚至不需要修改任何代码就可以直接迁移。
> 以上代码示例（业务代码），使用的都是静态类，每个类都可以当作是一个工具类，通过细分不同工具类达到对业务的拆分