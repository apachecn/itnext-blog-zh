# 利用 Vue 事件减少属性声明

> 原文：<https://itnext.io/leveraging-vue-events-to-reduce-prop-declarations-e38f5dce2aaf?source=collection_archive---------3----------------------->

![](img/4102ebea6c828473d7c306a50c062c73.png)

> [点击这里在 LinkedIn 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fleveraging-vue-events-to-reduce-prop-declarations-e38f5dce2aaf%3Futm_source%3Dmedium_sharelink%26utm_medium%3Dsocial%26utm_campaign%3Dbuffer)

虽然 Javascript 中的事件是许多模式和框架的“面包和黄油”,但 Vue 的抽象简单而优雅。对我来说是一个启示。我经常利用它的一个地方是我的容器和表示组件之间的通信。

让我们看看典型的 todo 示例。这个应用程序有一个项目列表，一个标题将有列表的名称和一个按钮来添加一个新项目。这些信息保存在一个 [Vuex](https://vuex.vuejs.org/en/) 商店中。您可以将这个应用程序的容器想象成这样:

```
<template>
  <section>
    <todo-header
      :title="title"
      :add-item="handleAddItem"
    />
    <todo-list
      :items="items"
      :mark-item-complete="handleItemComplete"
    />
  </section>
</template><script>
import { mapState } from 'vuex';import TodoHeader from './TodoHeader';
import TodoList from './TodoList';
import {
  UPDATE_TITLE,
  ADD_ITEM,
  MARK_ITEM_COMPLETE,
} from './mutation-types';export default {
  name: 'TodoWrapper',
  computed: {
    ...mapState([
      'title',
      'items',
    ]),
  },
  methods: {
    handleUpdateTitle(title) {
      this.$store.commit(UPDATE_TITLE, title);
    },
    handleAddItem(item) {
      this.$store.commit(ADD_ITEM, item);
    },
    handleItemComplete(item) {
      this.$store.commit(MARK_ITEM_COMPLETE, item);
    },
  },
};
</script>
```

我们来看看`TodoHeader`:

```
<template>
  <h1>{{ title }}</h1>
  <input type="text" :v-model="newItem" />
  <button @click="handleAddItem">Add item</button>
</template><script>
  export default {
    name: 'TodoHeader',
    data() {
      return {
        input: '',
      };
    },
    props: {
      title: {
        type: String,
        required: true,
      },
      addItem: {
        type: Function,
        required: true,
      },
    },
    methods: {
      handleAddItem() {
        this.$props.addItem(this.$data.input);
      },
    },
  };
</script>
```

随着更多的功能被添加到`TodoHeader`(或者上面提到的`TodoList`)中，将会有更多类似于`addItem`的回调道具被定义。这不仅会增加代码量，还会在单元测试中需要额外的存根。事件将有助于减少这一点！

```
// ... props: {
      title: {
        type: String,
        required: true,
      },
    },
    methods: {
      handleAddItem() {
        this.$emit('add-item', this.$data.input);
      },
    },
  };// ...
```

现在，`add-item`不再作为道具传递下去，`TodoWrapper`将只是监听事件。我们也可以用`TodoList`做到这一点。

```
// ...<todo-header
  :title="title"
  @add-item="handleAddItem"
/>
<todo-list
  :items="items"
  @mark-item-complete="handleItemComplete"
/>// ...
```

这种模式将进一步把你的表示组件从你的容器中分离出来，这将使你的组件更加可重用，并减少你的单元测试中需要的存根数量。