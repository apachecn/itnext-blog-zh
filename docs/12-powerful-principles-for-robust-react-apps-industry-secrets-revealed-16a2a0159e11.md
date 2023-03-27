# 强大的 React 应用程序的 12 个强大原则(行业机密泄露！)

> 原文：<https://itnext.io/12-powerful-principles-for-robust-react-apps-industry-secrets-revealed-16a2a0159e11?source=collection_archive---------3----------------------->

这是交易。这不是一个关于在哪里放置分号`;`或者如何分配数组`[]`的深度风格指南，这是一个简短而实用的顶级指南，可以快速地在代码库的可读性、可伸缩性和可靠性方面获得巨大的改进。为所有技能水平编写。这是用 React Native 编写的...但是它可以扩展到 React 应用程序。

![](img/baad672a09f6c1d949c97548b690c946.png)

照片由[菲利克斯·罗斯蒂格](https://unsplash.com/@felixrstg?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](/s/photos/friends?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 拍摄

如果你有一个现有的项目，这些原则并不需要在一次大的重构中完成，相反，它们是指路明灯，可以随着时间的推移应用到你自己的实践和惯例中。

# 1.按类型划分的扁平结构

react 本机应用程序的结构应该按类型划分。以下是一个可扩展的示例结构:

```
/components
    /tests
    /styles
       my-component-style.js
    my-component.js/pages
    /tests
	  /styles
       home-page-style.js
    home-page-style.js/utils
   alert-with-exit-app.js
   get-distance-between-points.js
   set-android-icon.js
   use-component-size.js/config
   images.js
   main-tab-strings.js
   debug-config.js/theme
   metrics.js
   global-styles.js
```

如您所见，这种结构是可扩展的。“/theme”文件夹中的文件也可以移动到“/config”文件夹中。对于“/utils”文件夹也是如此，如果您有 200 个包含自定义 react 挂钩的文件，您可以创建一个名为“/hooks”的新文件夹。

建议您不要超出两个文件夹来拆分组件，即使您有数千个组件(**开发人员不通过文件夹搜索，他们使用热键和搜索**)，这是因为这些文件夹有'/styles '和'/tests '的子文件夹，创建更多的文件夹是额外的开销。

上面的示例将组件分为“/pages”和“/components”，以提供应用程序的高级概述。每个根页面都可以通过“/pages”文件夹访问。确保每个根页面都加上`-page`或`-screen`的后缀。对于较小的应用程序，您可以将页面放入“/components”文件夹并删除“/pages”...但是当应用程序变大时，对应用程序的结构和可用页面有一个高层次的概述是很有用的。

> *为什么？当试图对新组件进行分类时，通过域划分特性会产生开销，从而导致摩擦。对于大多数项目来说，按类型划分是最好的方法。*

**关于** **Index.js 导出的说明**

使用平面结构时，不需要使用绝对导入。扁平结构与相对导入配合得很好…绝对导入破坏了 IDE 智能。您应该考虑在每个文件夹中使用一个`index.js`，例如'/components '，'/utils '等。这允许您用一条语句导出和导入文件:

```
import {
  alertWithExitApp,
  getDistanceBetweenPoints,
  useComponentSize
} from "../utils";
```

这确实需要一些额外的开销来维护`index.js`文件中的导出，但是简化导入的好处通常会超过维护开销。

# 2.每个文件一次导出

每个功能和组件都应该被分割成单独的文件。与上面的配置和主题文件夹相同，每个文件应该只包含一个对象配置。例如，`images.js`包含一个需要所有应用程序图像的对象。每个文件应与导出的功能、组件或配置的名称相匹配，例如`function getDistanceBetweenPoints`应在文件名`get-distance-between-points.js`中。

如果你想提供点符号 API。例如，自定义控制台 api。警告，。要做到这一点，您可以从您的文件中导出`customConsole`，它将是一个具有各种功能的对象:

```
export const customConsole = {
   log: () => ...,
   warn: () => ...,
   error: () => ...,
}
```

> *为什么？这使得代码库在查找组件、函数和对象时更容易导航。当文件遵循详细的命名约定时，它还减少了重复代码的机会。*

# 3.使用命名导出

将所有功能、组件和配置导出为命名导出。如`import { MyComponent } from "./components/MyComponent"`。不是默认。

> *为什么？导出命名导出会阻止相同的函数、组件和配置在您的代码库中获取不同的名称，从而降低复杂性和开销。*

# 4.始终使用功能部件(从 React 16.8 开始)

随着 React 16.8 中钩子的引入，你应该总是使用带有钩子的功能组件而不是类组件。

> *为什么？*
> 
> 在一个函数组件中，你可以声明你的变量、进程属性，并声明一次外部状态，然后在你的组件中使用它，而不必将这些值绑定到组件或状态。
> 
> 因为没有考虑将函数绑定到上下文“this”和生命周期方法的开销，所以复杂性更低。

# 5.在同一位置声明组件变量

组件级状态和变量应该总是在功能组件的顶部声明。注释可以用来分隔普通的变量组，以提高可读性，但这不是必须的。下面是一个真实的例子:

```
const EditProfileScreen = ({ componentId }) => {
  /** Redux */
  const isBiometricByDefault = useSelector(state => getIsBiometricByDefault(state));
  const { 
		isLoading: isSubmitting, 
		errors 
	} = useSelector(state => getNetworkRequestByType(state, UPDATE_USER_REQUEST));
  const {
    authToken,
    name,
    email,
    birthday,
    mobile,
    marketingEmails,
    postcode
  } = useSelector(state => getUser(state));
  const dispatch = useDispatch();
  /** Hooks */
  const formikRef = useRef(null);
  const [biometricType, biometricTypeAsString] = useBiometricType();
  const [confirmPasswordError, setConfirmPasswordError] = useState(false);
  /** Variables */
  const firstName = getFirstNameFromName(name);
  const lastName = getLastNameFromName(name);
  const mobilePrefix = getMobilePrefixFromString(mobile);
  const mobileNumber = getMobileNumberFromString(mobile);
  /** Effects */
  useEffect(() => ...
```

> *为什么？这停止了跨大型组件的变量的重新声明，并通过给出所有数据和处理需求以及该组件的顶层概述来帮助减少代码。*

# 6.将回调提取到嵌套函数中

您的`useEffects`和组件(例如`<MyComponent onPress={...}`)中的回调应该提取到组件底部的函数中:

```
useEffect(() => onFirstNameChanged(), [firstName])return (
	 <MyComponent onPress={onPressMyComponent}
   ...function onPressMyComponent() ...
function onFirstNameChanged() ...
```

> *为什么？这保持了组件的组织性和可读性。就像把你的衣服放在你的衣柜里一样，你不能把所有的东西都扔在地板上…不要对你的应用程序做同样的事情。*

# 7.不要将组件嵌套在组件中

当将较大的组件分割成较小的部分时，使用`<Components />`而不是渲染函数，例如`renderComponent`。这个非常简单...不要将 JSX 放在变量或函数中进行条件渲染。使用具有描述性名称的组件。

`renderBottomFooterConditions() { ...` ❌

`my-component-bottom-footer-conditions.js -> <MyComponentBottomFooterConditions` ✅

使用“渲染”功能是避开“2。“每个文件一次导出”原则…

我明白你在做什么👀我对此不以为然。

> *为什么？与组成成分反应。当某个东西足够复杂，以至于内联 JSX 三元组不够用，额外的处理和复杂的条件要求它被提取到自己的函数中时，只需创建一个新的组件。*

# 8.展平组件道具

在大多数情况下，在通过 props 将对象传递给组件之前，先将其展平。在某些情况下，您可能希望将一个对象作为组件属性进行传递，例如*“您有一组公共的数据处理逻辑，需要在应用程序的多个地方发生…* [*参见示例*](https://medium.com/@lukebrandonfarrell/component-composition-for-dummies-cd94515a360e)*”*。在使用带有 JavaScript 的类型系统(如 TypeScript)时，这一原则可以更加灵活……仍然建议将道具扁平化。

> *为什么？*
> 
> 使用对象比使用单值变量更复杂，也更容易出错；布尔值，数字，字符串。
> 
> 将一个对象传递给一个组件会将它耦合到那个对象数据结构，在大多数情况下，您希望您的组件比这更灵活。

# 9.将样式与组件分开

样式应该被分离到一个单独的文件中。按照这个指南，你的样式必须存在于组件文件夹根级的`/styles`文件夹中。如果您使用多个文件夹来分割组件(如组件/容器)，则每个文件夹都有一个单独的样式文件夹。

> *为什么？分离样式增加了组件代码的可读性，并促进了样式在应用程序中的重用。*

# 10.遵循命名约定

遵循项目的命名约定，您可以有自己的命名约定，但这里是默认约定:

*   文件和文件夹名称应该用 kebab-case: `my-new-component.js`
*   在组件之外声明的常量应该是开始用例:`IconSources`
*   导入到组件中的图像应该是开始案例:`import ArrowImage from 'assets/arrow.png';`
*   函数应该用 camel-case 编写:`getDistanceBetweenPoints`
*   组件中的变量和函数应该是骆驼形状:`const activeBrandId ...`
*   组件应使用起始名称命名:`<MyComponent />`
*   来自第三方库的默认和命名导入应该遵循项目命名约定:`import { SetPlatformIcon: setPlatformIcon } from 'rn-native-icon'`

> *为什么？保持一切整洁—* 玛丽·近藤

# 10.使用描述性命名

变量名和函数名应该是描述性的，而不是一般性的:

`let messageForTheWarningBanner = messages?.message_for_banner` ✅

`let message = messages?.message_for_banner` ❌

解构的进一步示例:

```
const { name: companyName } = useSelector(state => getOrganisation(state));
const { email: currentEmail } = useSelector(state => getUser(state));
```

> *为什么？它使代码更容易理解。*

# 评论

“我们是如何”和“为什么”来到这里的！？

请…评论你的代码。如果你不注释你的代码，那么下面解释如何注释代码。注释极大地增加了项目的健壮性，不注释代码就像卖一本没有说明的“自己构建”书架。

# 11.解释“为什么”,而不是“如何”

注释用于解释代码背后的“为什么”。“如何做”可以从代码中推断出来。谈谈你选择编写这部分代码的原因。下面是一个真实应用程序中单个变量的注释:

```
/**
 * Used so the rebrand alert does not appear multiple times,
 * as if the user exits the app, this will trigger new user data
 * to be fetched, which will trigger new organisation data to be
 * fetched, which will lead to multiple FETCH_ORGANISATION_RESPONSE
 * and multiple instances of this generator function waiting at
 * `call(waitUntilStack...`.
 */
let isRebrandAlertShowed = false;
```

> *为什么？“如何”可以从代码中推断出来……而“为什么”却永远消失了。*

# 12.用注释分解程序流

注释可以用来分解程序的控制流，使其更具可读性。注意:在适用的情况下，应使用“为什么”注释，这些注释“有助于如何做”，但不能代替“为什么”:

```
// Used to close the array in our JavaScript file
iconsJavascriptContent = iconsJavascriptContent + "]";// We use object assign, this takes the existing icons, our primary icon, and icons folder and updates them
plistObject["CFBundleIcons"]["CFBundleAlternateIcons"] = 
    Object.assign(plistObject["CFBundleIcons"]["CFBundleAlternateIcons"], PRIMARY_ICON, icons);// Build the plist object back into an XML list
const result = plist.build(plistObject);// Write the modified plist file back to source
try {
    fs.writeFileSync(PLIST_PATH, result);
    terminal.white(`\\n The plist file ${PLIST_PATH} was updated with new app icons. \\n`);
} catch(e) {
    return terminal.red(err);
}// Write to the JavaScript file used in the app to keep track of our active icons
try {
    fs.writeFileSync(JS_FILE_PATH, iconsJavascriptContent);
} catch(e) {
    return terminal.red(err);
}
```

> *为什么？为经验不足、不能立即理解“如何做”的开发人员增加可读性。*