# react Hooks——设计简单的表单 API——第 6 部分——模式验证

> 原文：<https://itnext.io/react-hooks-designing-a-simple-forms-api-part-6-yup-schema-validation-5a800cb1c330?source=collection_archive---------1----------------------->

在本文中，我们将扩展我们的表单 API 来支持[基于 yup 的模式验证](https://github.com/jquense/yup)。

在本系列的第 1 部分中，我们研究了如何使用 React 钩子来设计一个 React 表单库。第 1 部分介绍了这个库的动机和一些总体设计目标。

在本系列的第 2 部分[中，我们创建了一个`useInput`钩子，将其集成到我们的`useForm`钩子中，回顾了我们的解决方案的状态管理，合并了一些额外的测试，并且更详细地回顾了测试策略。](https://medium.com/@shanplourde/react-hooks-designing-a-simple-forms-api-part-2-1fe5d12f23d9)

在第 3 部分中，我们讨论了验证、异步验证和异步表单提交。它为我们提供了基于 React hooks 的表单 API 的关键元素。

[在第 4 部分](/react-hooks-designing-a-simple-forms-api-part-4-scaling-to-other-input-types-e738db0a3fc3)中，我们扩展了我们的解决方案，以支持标准 HTML 输入类型和自定义输入。

在第 5 部分中，我们扩展了`useForm`以支持动态表单。

# 工作示例

让我们来看看带有模式验证支持的`useForm`钩子的演示:

带有模式验证的 useForm 演示

[完整的解决方案可以在我的 GitHub](https://github.com/shanplourde/react-hooks-form-util) 中找到。演示中有很多样板文件，但我希望在这一点上有非常明确的例子。请注意，由于 CodeSandbox 设计决定不支持从`package.json`加载`devDependencies`，测试此时不会通过 CodeSandbox。

# 是的模式验证概述和好处

Yup 是一个模式验证库。您可以使用它来定义一个 JavaScript 模式，然后根据该模式验证任意对象。Yup 为您提供了一整套标准的模式验证，如 required、min length、max length、email 等。

有很多模式验证库，yup 恰好是其中较大且较活跃的一个。让我们通过查看一个[代码示例](https://runkit.com/jquense/yup#)来看看 yup 验证是什么样子的:

```
const { object, string, number, date } = require('yup')const contactSchema = object({
  name: string()
    .required(),
  age: number()
    .required()
    .positive()
    .integer(),
  email: string()
    .email(),
  website: string()
    .url(),
  createdOn: date()
    .default(() => new Date())
})let contact = {
  name: 'jimmy',
  age: 24,
  email: '[jdog@cool.biz](mailto:jdog@cool.biz)'
}await contactSchema.isValid(contact) // true = valid
```

`contactSchema`是 yup 模式。`contact`是您的数据对象(这可能是来自表单的数据)。最后调用`contactSchema.isValid()`根据`contactSchema`模式验证`contact`对象。Yup 提供了一个流畅的 API 来定义模式。实际上，这可能不是您想要的定义模式的方式。

例如，如果您的字段是从数据库中动态创建的，您将如何存储所有这些流畅的 API 调用？在这些情况下，您可以使用允许您以 JSON 格式存储模式的库，然后可以将模式序列化并存储在任何数据库中。 [schema-to-yup](https://github.com/kristianmandrup/schema-to-yup) 就是这样一个库，它允许您在 JSON 中定义模式，然后将模式转换为 yup 模式对象。

挂钩需要与 yup 集成。当向`useForm`提供 yup 模式时，`useForm`将需要对输入字段应用那些验证。

# 在任意时间应用模式验证

就像我们在之前的文章中创建的[开箱即用验证一样，模式验证应该能够与开箱即用验证同时运行。我们应该允许表单创建者说“嘿，我想在我的名字字段上运行 onBlur 和 onSubmit 的模式验证”，或者“嘿，我想对我的名字字段应用模式验证，同时应用我的](/react-hooks-designing-a-simple-forms-api-part-3-validation-and-a-running-example-18b835a3b817)[奖励早期验证晚期设计模式](https://medium.com/@shanplourde/inline-form-validations-ux-design-considerations-and-react-examples-c2f53f89bebc)”。

# 设计公共 API 以支持模式验证

在我看来，形式是模式的子集。两者之间存在耦合。所以我更喜欢用最初的`useForm`钩子调用来传递模式。

```
export const useForm = ({ id, initialState = {}, validationSchema = {} }) => {
  const inputValues = useRef({
    ...initialState
  });
```

`useForm`现在接收一个`validationSchema`属性，当设置该属性时，它将对表单中最终定义的字段应用模式验证。要使用模式验证，表单创建者需要执行以下操作:

```
const { required, mustBeTrue, schema } = validators;const contactSchema = object({
    firstName: string()
      .required()
      .min(3),
    lastName: string()
      .required()
      .length(10),
    email: string().email()
  });const {
    getFormProps,
    inputValues,
    uiState,
    api,
    formValidity,
    inputUiState
  } = useForm({
    id: "kitchenSinkForm",
    initialState,
    validationSchema: contactSchema
  });const firstNameInput = api.addInput({
    id: "firstName",
    value: inputValues.firstName,
    validators: [
      {
        ...schema,
        when: [
          {
            eventType: onChange,
            evaluateCondition: evaluateConditions.rewardEarlyValidateLate
          },
          onBlur,
          onSubmit
        ]
      }
    ]
  });const lastNameInput = api.addInput({
    id: "lastName",
    value: inputValues.lastName,
    validators: [{ ...schema, when: [onBlur, onSubmit] }]
  });const emailInput = api.addInput({
    id: "email",
    value: inputValues.email,
    validators: [
      { ...schema, when: [onBlur, onSubmit] }
      // {
      //   ...email,
      //   when: [onBlur, onSubmit]
      // }
    ]
  });
```

在上面的示例表单代码中，我们定义了`contactSchema`，这是一个 yup 模式。我们将它传递给`useForm`钩子。然后，当我们定义输入时，我们指出我们希望用`validators: [ { ...schema } ]`验证器对它们应用模式验证。这种方法的好处在于，我们还可以定义何时应用这些模式验证——onBlur、onSubmit 等。

在`api.addInput`中，当我们指定想要使用`schema`验证器时，`useForm`根据输入`id`连接适当的模式字段。

# 实施细节

## 创建模式验证器

在上面的`useForm`代码中，我们使用了`schema`验证器。下面是 validators.js 中的`schema`验证器代码:

```
validators.schema = createValidator({
  validateFn: async ({ value, validationSchema }) => {
    if (!validationSchema || !validationSchema.validate) return true;
    try {
      await validationSchema.validate(value);
      return true;
    } catch (e) {
      return false;
    }
  },
  error: "INVALID_SCHEMA"
});
```

我们使用 yup `validate` API 来执行验证。我们遵循其他验证器的相同模式——基于验证成功返回 true/false。如果一个 yup 验证失败，yup 将返回许多其他属性，甚至是错误描述，但是我的偏好是不在验证错误中包含内容。因此，如果模式验证失败，它们只会返回`INVALID_SCHEMA`错误消息。表单创建者可以决定在这些情况下显示什么样的副本。我相信这可以改进，但我不确定如何改进，因为表单创建者可以访问模式定义，所以如果他们愿意，他们可以查询模式定义来创建错误消息。

`useForm`只会将一个模式字段传递给模式验证。预期是，如果我们验证名字字段，`useForm`将只把`firstName`字段传递给模式验证器，而不是完整的模式

## 更新 useForm 以将架构字段传递给架构验证程序

`useForm`需要最小的改动。现在表单创建者可以将`validationSchema`传递给`useForm`，任何时候需要验证时，`useForm`都可以将正确的模式字段传递给验证。validators.js 只需要将`validationSchema`作为一个参数，并在适当的时候将其传递给模式验证器。

```
**const getSchemaField = (validationSchema, field) =>
  validationSchema.fields ? validationSchema.fields[field] : null;**...const validateAll = async eventType => {
    const promises = [];
    let newUiState = { ...uiState };Object.keys(validators.current).forEach(async field => {
      promises.push(
        runValidators({
          field,
          validators: validators.current[field],
          eventType,
          value: inputs.current[field].value,
          inputValues: inputValues.current,
  **        validationSchema: getSchemaField(validationSchema, field)**
        })
      );
    });...const runInputValidations = async ({ id, value, eventType, timeStamp }) => {
...try {
        const validationResults = await runValidators({
          field: id,
          validators: validators.current[id],
          eventType: eventType,
          value,
          runId: timeStamp,
          inputValues: inputValues.current,
 **validationSchema: getSchemaField(validationSchema, id)**
        });
```

# 摘要

在本文中，我们重构了我们的解决方案，以支持基于 yup 的模式验证。

在后面的部分中，我们将讨论其他主题，比如减少样板文件、取消承诺、去抖、钩子性能考虑和其他设计优化。

此时，该库可能已经“足够好”可以用于生产了😃。在开始自己制作表单库之前，请记住制作一个“足够好”的表单库需要付出的努力和思考！

除非您有一个不需要验证逻辑的极其简单的表单，是的，您可以用 React 拼凑一个表单，但是它会非常有限。一旦你需要越过简单的表单，使用表单库。否则，你可能没有充分利用你的时间。除非你是出于自己的学习目的，当然，我总是建议通过开发你感兴趣的东西来学习。

如果您有任何问题、反馈或建议，或者您希望我介绍本系列中的其他内容，请告诉我。