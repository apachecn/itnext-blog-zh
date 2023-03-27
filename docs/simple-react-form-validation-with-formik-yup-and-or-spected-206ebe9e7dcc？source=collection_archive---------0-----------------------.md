# 使用 Formik、Yep 和/或 Spected 进行简单的反应表单验证

> 原文：<https://itnext.io/simple-react-form-validation-with-formik-yup-and-or-spected-206ebe9e7dcc?source=collection_archive---------0----------------------->

最近，我在工作中写了一些需要验证的表格。因为我喜欢和图书馆打交道，尤其是那些帮助过我的图书馆，我想把你介绍给他们。

让我们构建一个简单的注册表单进行演示。此类表格通常包含用户的电子邮件、密码、密码确认以及条款和条件协议。

![](img/32b3194c63de283217e64ac8e2d2da70.png)

# 福米克

首先，我们用 [Formik](https://github.com/jaredpalmer/formik) 库创建注册表单。我喜欢 Formik，因为它是轻量级的，几乎没有抽象库来帮助你在 React 中创建表单。如果你不需要 Redux ( [你可能不需要 Redux — Dan Abramov — Medium](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367) ，[你应该把你的表单状态存储在 Redux 中吗？— Gosha Arinich](https://goshakkk.name/should-i-put-form-state-into-redux/) )是非常不错的选择。在我们的例子中，Formik 帮助我们保持状态(值、错误和表单是否被提交)并处理变更。它还在每次提交时为我们调用验证功能。我使用 Formik 作为 React 组件和[渲染道具](https://reactjs.org/docs/render-props.html)，但是你也可以用它作为 HoC。

```
export default function SignUpFormContainer() {
  return (
    <Formik
      initialValues={initialValues}
      validate={validate}
      onSubmit={onSubmit}
      render={SignUpForm}
    />
  )
}function SignUpForm(props) {
  const {
    isSubmitting,
    errors,
    handleChange,
    handleSubmit,
  } = propsreturn (
    <div className="form">
      <label className="form-field" htmlFor="email">
        <span>E-mail:</span>
        <input
          name="email"
          type="email"
          onChange={handleChange}
        />
      </label>
      <div className="form-field-error">{errors.email}</div> <label className="form-field" htmlFor="password">
        <span>Password:</span>
        <input
          name="password"
          type="password"
          onChange={handleChange}
        />
      </label>
      <div className="form-field-error">{errors.password}</div> <label
        className="form-field"
        htmlFor="passwordConfirmation"
      >
        <span>Confirm password:</span>
        <input
          name="passwordConfirmation"
          type="password"
          onChange={handleChange}
        />
      </label>
      <div className="form-field-error">
        {errors.passwordConfirmation}
      </div> <label className="form-field" htmlFor="consent">
        <span>Consent:</span>
        <input
          name="consent"
          type="checkbox"
          onChange={handleChange}
        />
      </label>
      <div className="form-field-error">{errors.consent}</div> <button onClick={handleSubmit}>
        {isSubmitting ? 'Loading' : 'Sign Up'}
      </button>
    </div>
  )
}
```

# 确认

我们如何验证这一点？我们可能会找到更多的方法来验证这种形式。我写下了一些我们将用来验证表单验证方法的需求(是的，这就像电影《盗梦空间》的重演。)

## 要求

*   我不想创建特殊的表单字段组件，并像`<FormField validate={value => someValidation(value)} />`一样单独为表单的每个字段添加验证。我认为验证应该是业务逻辑的一部分，而不是 UI 的一部分，如果没有必要，我不想混淆它。否则，我想有一个整个表单的验证函数，它接受表单值的对象，并返回每个值的错误消息对象。这种验证方法甚至可以用于任何其他 JS 对象，而不仅仅是表单验证。
*   没有`if`语句。
*   有可能用例外来代替`if`的。我也不喜欢那样。异常是针对意外行为的，但是表单域是否被填充并不意外。相反，这是意料之中的。正如我已经提到的，验证是我们业务规则的一部分。
*   每个字段一个或多个规则。我想根据一个或多个规则来验证每个属性。
*   我也希望能够返回给定字段更多的错误。
*   返回错误消息，每个规则都不同。
*   属性之间的交叉验证，这意味着根据其他字段值验证字段。

我提到“不”,因为从这样的事情开始会很有诱惑力:

```
if (!values.email) {
  errors.email = 'E-mail is required!'
}if (!values.password) {
  errors.password = 'Password is required!'
} else if (values.password.length < 6) {
  errors.password = 'Password has to be longer than 6 characters'
}
```

如果有更多的字段或更复杂的规则，可能会很难理解，所以我在想:“我们能做得更好吗？一定有其他更明确的方式，没有`if` s。

## 是的

幸运的是，Formik 本身默认允许使用 [Yup](https://github.com/jquense/yup) 验证库。用最简单的方法，你可以只写`validationSchema`并把它作为道具传递给`<Formik />`组件。

```
const validationSchema = Yup.object().shape({
  email: Yup.string()
    .email('E-mail is not valid!')
    .required('E-mail is required!'),
  password: Yup.string()
    .min(6, 'Password has to be longer than 6 characters!')  
    .required('Password is required!')
})<Formik 
  ...
  validationSchema={validationSchema} 
  ...
/>
```

这看起来比`if`的好得多，但是它只在你需要一些交叉验证时才有效。在我们的例子中，密码和密码确认就是这种情况。

在这种情况下，您仍然可以使用 Yup，但是要和您自己的验证函数一起使用。我们创建`getValidationSchema`函数。它接受`values`对象并返回验证模式，类似于上面的例子。

```
function getValidationSchema(values) {
  return Yup.object().shape({
    email: Yup.string()
      .email('E-mail is not valid!')
      .required('E-mail is required!'),
    password: Yup.string()
      .min(6, 'Password has to be longer than 6 characters!')
      .required('Password is required!'),
    passwordConfirmation: Yup.string()
      .oneOf([values.password], 'Passwords are not the same!')
      .required('Password confirmation is required!'),
    consent: Yup.bool()
      .test(
        'consent',
        'You have to agree with our Terms and Conditions!',
        value => value === true
      )
      .required(
        'You have to agree with our Terms and Conditions!'
      ),
  })
}
```

这很简单。我只想在这里强调两点:

1.  `passwordConfirmation`验证:据我所知，Yup 没有任何类似于`equals`的方法来比较两个字符串。我们需要将`passwordConfirmation`值与数组值`oneOf(array, 'Error message')`进行比较，其中`array`是只有一个来自`values`对象`values.password`的值的数组。
2.  `consent`验证:Yup 也没有办法检查值是`true`还是`false`。我们需要使用带 3 个参数的`test`方法。第一个是验证的名称(我不知道为什么🤔)，第二个是错误消息，第三个是我们的自定义验证函数，它将`value`作为参数，必须返回`true`或`false`。我们我用的只是简单的`value => value === true`。

接下来，我们只是在这个模式上调用方法`validateSync()`:

```
function validate(values) {
  validate = (values) => {
    const validationSchema = getValidationSchema(values)
    try {
      validationSchema.validateSync(values, { abortEarly: false })
      return {}
    } catch (error) {
      return getErrorsFromValidationError(error)
    }
  }
}
```

有一点小问题，因为`validateSync`方法不返回有错误的对象，而是抛出`ValidationError`。如果它直接返回 errors 对象就更好了。总之，我写了另一个简单的函数，它接受这个错误对象，并以我们想要的形式返回错误对象(每个表单字段只有第一个验证错误消息):

```
function getErrorsFromValidationError(validationError) {
  const FIRST_ERROR = 0
  return validationError.inner.reduce((errors, error) => {
    return {
      ...errors,
      [error.path]: error.errors[FIRST_ERROR],
    }
  }, {})
}
```

如您所见，我们的验证现在看起来好多了。是的，我们必须再添加两个函数(是的，我们正在处理异常验证🙄)，但这些都是简单的函数，可以在任何验证模式中重用。如果我可以在`if`和带有另外两个函数的声明式验证模式之间进行选择，我肯定会选择第二个选项。

优点:

*   宣言的
*   预定义的验证方法(`email`、`min`、`max`、`required`)

缺点:

*   你必须学习它的 API
*   由异常处理的验证

## 斯佩克特

Formik 的伟大之处在于，您可以使用任何您想要的验证库。你只需要使用`validate`函数，而不是 Yup 特定的`validationSchema`。我意识到使用 [Spected](https://github.com/25th-floor/spected) 验证库来进行表单验证会很好，因为我们已经在工作项目的后端使用它来验证 JS 对象。

Spected 验证也非常具有声明性。它甚至比“是”更实用。没有命令，没有异常处理，只有简单的声明性函数验证。

让我们从`getValidationSchema`功能开始:

```
function getSpectedValidationSchema(values) {
  return {
    email: [
      [value => !isEmpty(value), 'E-mail is required!'],
      [value => isEmail(value), 'E-mail is not valid!'],
    ],
    password: [
      [value => !isEmpty(value), 'Password is required!'],
      [
        value => value.length >= 6,
        `Password has to be longer than 6 characters!`,
      ],
    ],
    passwordConfirmation: [
      [
        value => !isEmpty(value),
        'Password confirmation is required!',
      ],
      [
        value => value === values.password,
        'Passwords are not the same!',
      ],
    ],
    consent: [
      [
        value => value === true,
        'You have to agree with our Terms and Conditions!',
      ],
    ],
  }
}
```

正如你所看到的，我们需要使用我们自己的`isEmpty`和`isEmail`函数，但是我也可能是有益的，因为我们可以定义和命名函数，以便更方便地用于我们的业务领域。

我们的验证函数看起来有点不同。现在，我们不处理异常，但是我们仍然必须处理验证结果以符合我们的要求(以表单字段作为键，以错误消息作为值的错误对象):

```
function validate(getValidationSchema) {
  return (values) => {
    const spec = getValidationSchema(values)
    const validationResult = spected(spec, values)
    return getErrorsFromValidationResult(validationResult)
  }
}function getErrorsFromValidationResult(validationResult) {
  const FIRST_ERROR = 0
  return Object.keys(validationResult).reduce((errors, field) => {
    return validationResult[field] !== true
      ? { ...errors, [field]: validationResult[field][FIRST_ERROR] }
      : errors
  }, {})
}
```

Spected 有一个问题——必需表单字段的定义。例如，如果`values`对象不包含`consent`，那么就不会调用`consent`的验证函数。这是因为 Spected 不会迭代未定义的属性。

这在我们的注册表单中不是问题，因为我们将所有值都定义为初始状态。在其他情况下，我们可以通过将所有必填字段定义为未定义并用验证对象中的值覆盖它们来解决这个问题:

```
const requiredFields = {
  email: undefined,
  password: undefined,
  passwordConfirmation: undefined,
  consent: undefined,
}const valuesWithRequiredFields = { ...requiredFields, ...values }
```

优点:

*   声明性的，但功能性更强的方法。
*   简单的轻量级 API。
*   验证不是由异常处理的。

缺点:

*   不检查验证对象中不包含的必填字段。
*   没有预定义的验证函数。

你可以在这里找到完整的代码示例:[GitHub—jakubkoci/react-form-validation:用 Formik 和 Yup](https://github.com/jakubkoci/react-form-validation) 进行简单的 React 表单验证。