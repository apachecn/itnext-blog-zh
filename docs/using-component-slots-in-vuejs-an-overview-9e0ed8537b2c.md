# 在 VueJS 中使用组件插槽—概述

> 原文：<https://itnext.io/using-component-slots-in-vuejs-an-overview-9e0ed8537b2c?source=collection_archive---------1----------------------->

![](img/3a12e598ebdaca626b63204596aae227.png)

由[卢卡·布拉沃](https://unsplash.com/@lucabravo?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

插槽是 Vue 中组件向子组件注入内容的另一种方式。这是使用模板代码实现的。

就最终输出而言，插槽执行的功能类似于 Vue 中的 props 从父组件向子组件获取数据。

然而，props 将数据值传递给组件，而 slots 只能直接传递模板代码。我认为根据具体情况，这会带来一些好处:

*   你的子组件更容易重用——你可以传递给它不同的组件，而不用担心格式/数据值的一致性
*   更加灵活的是**——你不必总是填充每个值，而对于 props，你需要担心使用`v-if`来检查值是否存在**
*   **这可能是个人的事情，但是我认为子组件看起来更有**可读性****

**我认为最好的理解老虎机的方法是看一个如何使用它们的例子，以及实际发生了什么。**

# **最简单的用例**

**从 slots 开始是一个典型的用例，在这个用例中，我们简单地在子组件中声明一个`slot`,并使用父组件注入内容。**

**我们去看看。首先，让我们设置一个名为`MyContainer.vue`的父组件**

```
<template>
   <div>
     <my-button>Click Me!</my-button>
   </div>
</template>
```

**接下来，让我们设置一个子组件`MyButton.vue`组件。**

```
<template>
   <div>
     <slot></slot>
   </div>
</template>
```

**当 MyButton.vue 呈现时，`<slot>`将被`Click Me!`替换——来自父内容的**。****

**你可以从父组件传递任何类型的模板，不一定只是文本。它可以是一个字体很棒的图标，图像，甚至是另一个组件。**

# **需要多个插槽？说出他们的名字。**

**组织基于插槽的组件系统的最佳方式是将您的插槽命名为。这样，您可以确保将内容注入到组件的正确部分。**

**如您所料，这是通过向您的子组件中的`slot`添加一个**名称属性**来完成的。然后，要从父元素添加内容，只需创建另一个`<template>`元素，并在名为`v-slot`的属性中传递名称**

**让我们来看看实际情况。**

```
// BoxElement.vue
<template>
   <div>
      <slot name='header'></slot>
      <slot name='content'></slot>
   </div>
</template>
```

**然后是父组件。**

```
<template>
   <div> 
      <box-element>    
         <template v-slot:header>
            This will be injected as the header slot.
         </template>
         <template v-slot:content>
            This will be the content of the element
         </template>
      </box-element>  
   </div>
</template>
```

**注意:如果一个插槽没有命名。它的名字只是`default`**

# **传递数据作用域槽**

**关于插槽的另一个巧妙之处是，您可以让父组件**访问子组件内部的数据**。**

**例如，如果子组件使用一个数据对象来决定它显示什么，我们可以让数据**对父组件**可见，并在传递注入的内容时使用它。**

**我们再来看一个例子。**

**如果我们在子组件`Article.vue`中有这个文章标题槽—在这种情况下，我们的后备数据是文章标题。**

```
<div>
   <slot name='header'>
      {{article.title}}
   </slot>
</div>
```

**现在让我们继续看父组件。如果我们想改变内容来显示文章的描述，会发生什么？我们不能这样做，因为我们的父组件**不能访问其子组件`Article.vue`中的文章对象****

**谢天谢地，Vue 可以很容易地处理这种情况。我们可以用一个简单的 `v-bind`将数据从子插槽绑定到父模板**

**让我们来看看我们修改后的 div。**

```
<div>
   <slot name='header' v-bind:article='article'>
      {{article.title}}
   </slot>
</div>
```

**那里。我们的父组件可以访问该属性。现在，让我们看看如何访问它。**

**类似于使用 props 向组件传递数据时，我们的子组件传递一个 **props 对象**，所有有界属性都作为子对象。**

**我们所要做的就是在模板中命名这个对象，然后我们就可以访问它们了。我们暂时将它命名为`articleInfo`，但这只是一个变量，所以你可以使用任何你想要的东西。**

```
<div>
   <article v-slot:header='articleInfo'>
      // we have access to the article object now!
      {{ articleInfo.article header }}
   </article>
</div>
```

**简单对吗？**

# **使用插槽使组件更加灵活**

**Props 是重用组件的一种很好的方式，但是根据您的用例，它们有其局限性。Props 倾向于在具有相同格式和内容的组件中工作得最好，但是只是值不同。**

**有时你需要让你的组件更加灵活和适应:也许你想让一些组件有某些部分，而根据它在哪个页面，你想删除其他部分。**

**通过使用插槽注入内容，可以更容易地切换组件的内容，而不必担心使用模板逻辑如`v-if`和`v-else`来处理呈现。**

# **这是所有的乡亲**

**我希望这篇小文章能帮助你了解一些使用 slots 来组织你的 Vue 项目的可能性。**

**和往常一样，要了解更多信息和技术细节，请查看 [Vue 文档](https://vuejs.org/v2/guide/components-slots.html)。**

**编码快乐！**