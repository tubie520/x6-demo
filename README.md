# x6

## Project setup
```
yarn install
```

### Compiles and hot-reloads for development
```
yarn serve
```

### Compiles and minifies for production
```
yarn build
```

### Lints and fixes files
```
yarn lint
```

## Bug说明
> 主要问题
- 使用`api`方式创建连接桩后再连线后获取使用`node`调用 `node.visible`隐藏，后再使用显示会使页面卡死。不添加连线的情况下隐藏显示没问题。`Bug`重现启动项目后点击 **显示** 然后在点击 **隐藏** 按钮即会卡死。

> 竟然提了demo 有时间可以看看如下次要问题，可以不处理， 哈哈。
- 鼠标滚动到下面，然后拖动节点`id:9`移动位置，会造成节点在画布位置中跳动，不知道是不是设置有问题。
- 当`Graph` 设置添加`async: true`， 会造成连接线`zIndex`失效问题，当前`demo`创建的连线都是在`node`层级下显示。
- 有没有设置可以使节点不可控制，包括点击移动， 就是相当于只能操作画布移动之类的。