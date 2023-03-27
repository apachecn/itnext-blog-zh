# Vuelidate 验证消息

> 原文：<https://itnext.io/vuelidate-validation-messages-7c318322fd87?source=collection_archive---------4----------------------->

![](img/5ab7252b0034244d8e3a0ea521ee9cc3.png)

# 更新

我写了一个名为 [vuelidate-messages](https://gitlab.com/rhythnic/vuelidate-messages) 的库，可以在 npm 上找到。API 和实现与下面描述的类似。

# 介绍

[Vuelidate](https://monterail.github.io/vuelidate/#getting-started) 在存储表单验证状态方面做得很好。它不支持显示错误消息。另一个非常流行的 [Vue](https://vuejs.org/v2/guide/) 表单验证库， [Vee-validate](https://baianat.github.io/vee-validate/) ，对在组件中显示错误消息提供了强大的支持。尽管这是有代价的。Vee-validate 可能会增加你的包的大小，对我来说，这个 API 比 Vuelidate 要复杂一些。Vuelidate 更多地依赖于约定，以减少 API 面。我也喜欢 Vuelidate 用于验证对象和叶节点的共享 API 的对称性。

如果您选择 Vuelidate，那么您必须解决显示错误消息的问题。最流行的打包解决方案是[vuelidate-error-extractor](https://dobromir-hristov.github.io/vuelidate-error-extractor/)。我其实没试过。这可能是一个不错的选择，但是对我来说，包含验证消息组件模板和包装每个输入意味着这个解决方案太复杂了。此外，我们在工作中使用了 [Vuetify](https://vuetifyjs.com/en/getting-started/quick-start) 。Vuetify 已经有了一个传递错误消息的道具，我不想包装 v-text-field 组件。

Vuelidate 已经存储了验证状态。我们的解决方案不应该在状态中存储错误消息，所以我们可以避免试图保持它们同步的问题。这意味着组件数据不在考虑范围内。计算属性也不起作用，因为验证对象的形状可能会改变，很可能是在数组中添加和删除项目时。这就给我们留下了方法。这里给出的解决方案使用 Vue 组件方法来分析 Vuelidate 验证字段($v.name)，如果该字段在某些方面无效，则返回一条验证消息。

# 试映

下面是在 Vue 组件中使用该解决方案的预览。

```
<template>
  <div>
    <input v-model="name" />
    <span>{{validationMsg($v.name)}}</span>
  </div>
</template><script>
import { required, minLength } from 'vuelidate/lib/validators'
import { validationMessage } from '../our-solution'const validations = {
  name: {
    required,
    txtMinLen: minLength(2)
  }
}export default {
  data () {
    return { name: '' } 
  },
  validations,
  methods: {
    validationMsg: validationMessage(validations)
  }
}
</script>
```

# 步骤 1-验证消息创建者

解决方案的第一部分是提供消息。这里我们使用函数作为支持在消息中使用参数的简单方法。该函数还将被 Vue 组件的上下文调用，因此该函数可以访问 [VueI18n](http://kazupon.github.io/vue-i18n/) API。但是要注意，因为箭头函数被绑定到创建它们的上下文中，所以它们不能访问 Vue 组件实例。

```
const plural = (words, count) => {
  words = words.split('|').map(x => x.trim())
  return count > 1 ? words[2].replace('{n}', count) : words[count]
}export const validationMessageCreators = {
  required: () => 'Required',
  email: () => 'Invalid email',
  txtMinLen: ({ $params }) => {
    const min = plural(
      'no characters | one character | {n} characters',
      $params.txtMinLen.min
    )
    return `Must be at least ${min}`
  }
}
```

该解决方案依赖于将 Vuelidate 验证对象中的键与 validationMessageCreators 对象中的键进行匹配。例如，要使用 **txtMinLen** 验证消息，您的验证对象应该是这样的:

```
const validations = {
  name: {
    txtMinLen: minLength(2)
  }
}
```

# 步骤 2 —解析验证器密钥的验证对象

**更新**:我没有在 npm 包中包含步骤 2。

这个函数将递归解析您的验证对象，并构建一个验证器所有键的列表。这样做的主要原因是它允许我们检查(在步骤 3 中)我们有所有验证器的消息。

```
const isFn = x => typeof x === 'function'// Given a Vuelidate validations object, find the validator keys
function extractValidatorKeys (validations) {
  const keys = Object.keys(validations)
  const validatorKeys = keys.filter(x => isFn(validations[x]))
  const nested = keys.filter(x => !isFn(validations[x]))
  return validatorKeys.concat(
    ...nested.map(x => extractValidatorKeys(validations[x]))
  )
```

# 步骤 3 — Vue 组件方法

这是我们导出的函数。您将使用验证消息和验证对象来处理它，并得到一个接受 Vuelidate 字段的函数。我选择只返回一个错误。我不喜欢在每个输入字段中一次显示多个错误，但是您可以很容易地修改它来返回一个数组。

```
export const getValidationMessage = messages => validations => {
  let keys = extractValidatorKeys(validations) // check to make sure all validators have error messages
  const missing = keys.filter(x => !(x in messages))
  if (missing.length) {
    console.warn(
      'Validators missing validation messages: %s',
      missing.join(', ')
    )
    // remove keys that don't have validation messages
    keys = keys.filter(x => missing.indexOf(x) < 0)
  }

  // Vue component method
  // Given a vuelidate field object, maybe return a string
  return function (...args) {
    let field = args[0]
    if (!field.$error) return ''
    for (let i = 0; i < keys.length; i++) {
      let key = keys[i]
      if (field[key] === false) {
        return messages[key].apply(this, args)
      }
    }
    return ''
  }
}
```

# 步骤 4-导出课程实用程序

最后，我们将使用 validationMessageCreators 来处理实用程序，以使导入更容易。

```
export const validationMessage =
    getValidationMessage(validationMessageCreators)
```

# 结论

解决方案本质上是将验证键映射到错误消息。我不确定，但是我假设 Vue 组件方法，在示例中被称为“errors ”,在组件每次更新时为每个输入运行。该函数中的代码量很少，我没有注意到任何缓慢的行为，但是如果您有一个非常大的表单，您可能需要一种优化技术，比如记忆化，使用计算属性，或者将表单分成多个组件。

[vuelidate-消息包](https://gitlab.com/rhythnic/vuelidate-messages)