# Jackson-js:强大的 JavaScript 装饰器，可以将对象序列化/反序列化到 JSON 中，反之亦然(第 1 部分)

> 原文：<https://itnext.io/jackson-js-powerful-javascript-decorators-to-serialize-deserialize-objects-into-json-and-vice-df952454cf?source=collection_archive---------2----------------------->

![](img/eb0745b9c8f02428bb8374ecd38d7bc2.png)

JSON 徽标

经过许多小时的开发，我终于发布了第一个版本的`[jackson-js](https://github.com/pichillilorenzo/jackson-js)`库。顾名思义，`[jackson-js](https://github.com/pichillilorenzo/jackson-js)`decorator 受到了著名的 Java [FasterXML/jackson 库](https://github.com/FasterXML/jackson)的 Java 注释的极大启发。

你可以使用`npm install —-save jackson-js`安装它，它可以在**客户端**(浏览器)和**服务器** (Node.js)端使用。

## 为什么是这个图书馆？用这个库代替`[JSON.parse](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse)`和`[JSON.stringify](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)`有什么区别？

对于简单的情况，不需要这个库，当然可以只使用`JSON.parse`和`JSON.stringify`来序列化/反序列化 JSON。

使用`[jackson-js](https://github.com/pichillilorenzo/jackson-js)`，您可以使用`@JsonProperty()`、`@JsonFormat()`、`@JsonIgnore()`等装饰器轻松操作您的 JavaScript 对象/值序列化/反序列化。但是，这个库在引擎盖下使用了`[JSON.parse](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse)`和`[JSON.stringify](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)`。

此外:

*   它不仅将 JSON 文本反序列化为 JavaScript 对象，还将其转换为在`context`选项中指定的类的**实例(类似的包有: [class-transformer](https://github.com/typestack/class-transformer) 和[TypedJSON](https://github.com/JohnWeisz/TypedJSON))；相反，用`JSON.parse`你将得到一个简单的普通(文字)JavaScript 对象(只是`[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)`类型)；**
*   它支持更高级的对象概念，如**多态**和**对象标识**；
*   支持**循环对象**序列化/反序列化；
*   支持其他原生 JavaScript 类型的序列化/反序列化:`Map`、`Set`、`BigInt`、类型化数组(如`[Int8Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Int8Array)`)；

该库在更复杂的情况下会很有用，例如当您想要:

*   深度操纵 JSON
*   恢复一个 JavaScript 类型(类似的包是[class-transformer](https://github.com/typestack/class-transformer))；
*   保留类型信息(使用多态类型处理修饰符:`@JsonTypeInfo`、`@JsonSubTypes`和`@JsonTypeName`)。类似的包是[typed JSON](https://github.com/JohnWeisz/TypedJSON))；
*   隐藏某些 HTTP 端点或某些其他外部服务的某些属性；
*   对某些外部应用程序有不同的 JSON 响应，或者管理来自其他应用程序的不同 JSON 数据(例如，您需要与对同一模型使用不同 JSON 模式的 Spring Boot 应用程序进行通信，或者与使用 Python、PHP 等开发的其他应用程序进行通信……)；
*   管理循环引用；
*   管理其他 JavaScript 原生类型，如地图和集合；
*   等等。

Java [FasterXML/jackson](https://github.com/FasterXML/jackson) 注释的大多数用例是相似或相同的。

在本文中，我将为每个装饰者提供一个基本的例子。

# ObjectMapper、JsonParser 和 JsonStringifier 类

`[jackson-js](https://github.com/pichillilorenzo/jackson-js)`提供序列化和反序列化 JavaScript 对象的主要类有:`ObjectMapper`、`JsonStringifier`和`JsonParser`。

## 对象映射器

`ObjectMapper`提供读写 JSON 的功能，并应用`jackson-js` **装饰器**。它将使用`JsonParser`和`JsonStringifier`的实例来实现 JSON 的实际读/写。它有两种方法:

*   `stringify(obj: T, context?: JsonStringifierContext): string`:应用**装饰器**将 JavaScript 对象或值序列化为 JSON 字符串的方法；
*   `parse(text: string, context?: JsonParserContext): T`:将 JSON 字符串反序列化为 JavaScript 对象/值(类型为`T`，基于给定的上下文)的方法，应用了**decorator**。

## JsonParser

`JsonParser`提供编写 JSON 的功能，并应用`jackson-js`decorator。主要方法有:

*   `parse(text: string, context?: JsonParserContext): T`:将 JSON 字符串反序列化为应用了**装饰符**的 JavaScript 对象/值(类型`T`，基于给定的上下文)的方法；
*   `transform(value: any, context?: JsonParserContext): any`:将`jackson-js` **装饰器**应用于解析的 JavaScript 对象/值的方法。它返回一个应用了 decorators 的 JavaScript 对象/值。

## JsonStringifier

`JsonStringifier`提供读取 JSON 的功能，并应用`jackson-js` **装饰器**。主要方法有:

*   `stringify(obj: T, context?: JsonStringifierContext): string`:应用**装饰器**将 JavaScript 对象或值序列化为 JSON 字符串的方法；
*   `transform(value: any, context?: JsonStringifierContext): any`:将`jackson-js` **装饰器**应用于 JavaScript 对象/值的方法。它返回一个应用了 decorators 的 JavaScript 对象/值，并准备好进行 JSON 序列化。

# 装修工

在我们继续之前，我需要说最重要的装饰者是:

*   `@JsonProperty()`:每个类属性(或者它的 getter/setter)都必须用这个 decorator 来修饰，否则，反序列化和序列化将无法正常工作！这是因为，例如，给定一个 JavaScript 类，没有任何方法或 API(如 Java 的[反射 API)来确定所有的类属性；还因为，有时候，编译器如](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/package-summary.html) [TypeScript](https://www.typescriptlang.org/) 和 [Babel](https://babeljs.io/) ，可以在编译后从类属性声明中剥离类属性；
*   `@JsonClassType()`:这个装饰器用于定义类属性或方法参数的类型。该信息在序列化过程中使用，更重要的是，在反序列化过程中使用，以了解属性 / **参数**的**类型。这是必要的，因为 JavaScript 不是强类型编程语言，所以，例如，在反序列化期间，如果不使用这个装饰器，就没有任何方法知道类属性的具体类型，比如一个`Date`或一个定制的类类型。**

稍后，将对它们进行更详细的解释。

@JsonProperty()和@JsonClassType()装饰器的快速示例。

# @JsonAlias

在反序列化期间， *@JsonAlias* decorator 为属性定义了一个或多个可选名称。

**API**:[JsonAlias](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsonalias)—装饰选项 [JsonAliasOptions](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsonaliasoptions.html) 。

# @JsonAnyGetter

*@JsonAnyGetter* 装饰器允许灵活地使用`Map`或对象文字字段作为标准属性。

**API**:[JsonAnyGetter](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsonanygetter)—装饰器选项 [JsonAnyGetterOptions](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsonanygetteroptions.html) 。

# @JsonAnySetter

*@JsonAnySetter* 允许我们定义一个逻辑“AnySetter”赋值函数，使用一个非静态的双参数方法作为从 JSON 内容中找到的所有未识别属性的“后备”处理程序。

**API**:[JsonAnySetter](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsonanysetter)—装饰器选项 [JsonAnySetterOptions](https://pichillilorenzo.github.io/jackson-js/latest/modules/types.html#jsonanysetteroptions) 。

# @JsonAppend

*@JsonAppend* 可以用来添加要写在常规属性之后的“虚拟”属性。

**API**:[JsonAppend](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsonappend)—装饰器选项: [JsonAppendOptions](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsonappendoptions.html) 。

# @JsonManagedReference 和@JsonBackReference

*@JsonManagedReference* 和 *@JsonBackReference* 装饰器可以处理父/子关系并解决循环问题。

**API**:[JsonManagedReference](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsonmanagedreference)—装饰器选项[JsonManagedReferenceOptions](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsonmanagedreferenceoptions.html)， [JsonBackReference](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsonbackreference) —装饰器选项[JsonBackReferenceOptions](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsonbackreferenceoptions.html)。

# @JsonClassType

如前所述， *@JsonClassType* 用于定义类属性或方法参数的类型。一个类型被定义为 JavaScript 类的数组，比如对于类型为`number`的属性为`[Number]`，对于类型为`Array<number>`的属性为`[Array, [Number]]`，对于类型为`Map<string, any>`的属性为`[Map, [String, Object]]`。

为什么是 JavaScript 类的数组？因为这样你就可以映射复杂的类型，比如使用`[Map, [String, Object]]`映射`Map<string, any>`或者使用`[Array, [Set, [Object]]]`映射`Array<Set<any>>`。

**API**:[JsonClassType](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsonclasstype)—装饰选项 [JsonClassTypeOptions](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsonclasstypeoptions.html) 。

# @JsonCreator

我们可以使用 *@JsonCreator* decorator 将构造函数和工厂方法定义为一体，用于实例化相关类的新实例。

当我们需要反序列化一些与我们需要获得的目标实体不完全匹配的 JSON 时，这非常有帮助，也有了*@ JSON property*decorator 的帮助。

**API**:[JSON creator](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsoncreator)—装饰选项 [JsonCreatorOptions](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsoncreatoroptions.html) 。

# @JsonSerialize 和@JsonDeserialize

@JsonSerialize 和@ *JsonDeserialize* 用于表示使用定制的串行化器/去串行化器。

**API**:[JSON serialize](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsonserialize)—装饰器选项 [JsonSerializeOptions](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsonserializeoptions.html) ， [JsonDeserialize](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsondeserialize) —装饰器选项 [JsonDeserializeOptions](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsondeserializeoptions.html) 。

# @JsonFilter

*@JsonFilter* 可用于指示哪个逻辑过滤器用于过滤掉修饰类型(类)的属性。

**API** : [JsonFilter](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsonfilter) —装饰器选项 [JsonFilterOptions](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsonfilteroptions.html) 。

# @JsonFormat

*@JsonFormat* 是一个通用的装饰器，用于配置属性值如何序列化的细节。

**API**:[JSON format](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsonformat)—装饰器选项 [JsonFormatOptions](https://pichillilorenzo.github.io/jackson-js/latest/modules/types.html#jsonformatoptions) 。

# @JsonGetter 和@JsonSetter

*@JsonGetter* 和 *@JsonSetter* 是更一般的*@ JSON property*decorator 的替代方法，用于将一个方法标记为逻辑属性的 getter/setter 方法。

**API**:[JsonGetter](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsongetter)—装饰器选项: [JsonGetterOptions](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsongetteroptions.html) ， [JsonSetter](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsonsetter) —装饰器选项 [JsonSetterOptions](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsonsetteroptions.html) 。

# @JsonIdentityInfo

*@JsonIdentityInfo* 表示在序列化/反序列化值时应该使用对象标识——例如，处理无限递归类型的问题。

**API**:[jsonientityinfo](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsonidentityinfo)—装饰选项[jsonientityinfooptions](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsonidentityinfooptions.html)。

# @ JsonIdentityReference

*@ jsonientityreference*可用于定制启用了“对象标识”的对象的引用详情(参见*@ jsonientityinfo*)。主要用例是强制使用对象 Id，即使是第一次引用对象，而不是第一个实例被序列化为完整的类。

**API**:[jsonientityreference](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsonidentityreference)—装饰选项[jsonientityreferenceoptions](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsonidentityreferenceoptions.html)。

# @JsonIgnore、@JsonIgnoreProperties 和@ JsonIgnoreType

*@JsonIgnore* 用于标记在序列化和反序列化过程中要在字段级忽略的属性。

**API**:[JsonIgnore](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsonignore)—装饰选项 [JsonIgnoreOptions](https://pichillilorenzo.github.io/jackson-js/latest/modules/types.html#jsonignoreoptions) 。

*@JsonIgnoreProperties* 可以用作类级装饰器，标记在序列化和反序列化期间将被忽略的属性或属性列表。

**API**:[jsonignorepropertypes](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsonignoreproperties)—装饰选项[jsonignorepropertypesoptions](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsonignorepropertiesoptions.html)。

@ jsonignotype 表示在序列化和反序列化过程中将忽略修饰类型的所有属性。

**API**:[jsonignotype](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsonignoretype)—装饰器选项 [JsonIgnoreTypeOptions](https://pichillilorenzo.github.io/jackson-js/latest/modules/types.html#jsonignoretypeoptions) 。

# @JsonInclude

*@JsonInclude* 可以用来排除空/null/默认值的属性。

**API**:[JSON include](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsoninclude)—装饰选项 [JsonIncludeOptions](https://pichillilorenzo.github.io/jackson-js/latest/modules/types.html#jsonincludeoptions) 。

# @ JsonInject

*@JsonInject* decorator 用于表示反序列化过程中将注入修饰属性的值。

**API**:[JSON inject](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsoninject)—装饰器选项[JSON inject 选项](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsoninjectoptions.html)。

# @JsonNaming

*@JsonNaming* decorator 用于为序列化中的属性选择命名策略(SNAKE_CASE、UPPER_CAMEL_CASE、LOWER_CAMEL_CASE、LOWER_CASE、KEBAB_CASE 和 LOWER_DOT_CASE)，覆盖默认值。

**API**:[JsonNaming](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsonnaming)—装饰选项 [JsonNamingOptions](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsonnamingoptions.html) 。

# @JsonProperty

*@JsonProperty* 可用于将非静态方法定义为逻辑属性的“setter”或“getter”，或将非静态对象字段用作(序列化、反序列化)逻辑属性。

**API**:[JSON property](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsonproperty)—装饰器选项 [JsonPropertyOptions](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsonpropertyoptions.html) 。

# @ jsonpropertyeorder

*@JsonPropertyOrder* 可以用来指定属性序列化的顺序。

**API**:[JsonPropertyOrder](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsonpropertyorder)—装饰器选项[JsonPropertyOrderOptions](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsonpropertyorderoptions.html)。

# @JsonRawValue

*@JsonRawValue* decorator 表示被修饰的方法或字段应该通过原样包括属性的文字字符串值来序列化，而不引用字符。这对于注入已经在 JSON 中序列化的值或者将 javascript 函数定义从服务器传递到 javascript 客户机非常有用。

**API**:[JsonRawValue](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsonrawvalue)—装饰选项 [JsonRawValueOptions](https://pichillilorenzo.github.io/jackson-js/latest/modules/types.html#jsonrawvalueoptions) 。

# @JsonRootName

使用@ JSON root namedecorator——如果启用了包装——来指定要使用的根包装器的名称。

**API**:[jsonroot name](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsonrootname)—装饰器选项 [JsonRootNameOptions](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsonrootnameoptions.html) 。

# 多态类型处理装饰器:@JsonTypeInfo、@JsonSubTypes 和@JsonTypeName

*   *@JsonTypeInfo* :指明在序列化中包含什么类型信息的细节；**API**:[JsonTypeInfo](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsontypeinfo)—装饰选项[JsonTypeInfoOptions](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsontypeinfooptions.html)；
*   *@JsonSubTypes* :表示被注释类型的子类型；**API**:[JsonSubTypes](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsonsubtypes)—装饰器选项[JsonSubTypes 选项](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsonsubtypesoptions.html)；
*   *@JsonTypeName* :定义用于注释类的逻辑类型名；**API**:[JSON typename](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsontypename)—装饰器选项 [JsonTypeNameOptions](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsontypenameoptions.html) 。

# @JsonTypeId

*@JsonTypeId* decorator 用于指示当包含多态类型信息时，被注释的属性应该被序列化为类型 Id，而不是作为常规属性。在反序列化过程中使用多态元数据来重新创建与序列化前相同子类型的对象，而不是已声明超类型的对象。

**API** : [JsonTypeId](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsontypeid) —装饰选项 [JsonTypeIdOptions](https://pichillilorenzo.github.io/jackson-js/latest/modules/types.html#jsontypeidoptions) 。

# @JsonTypeIdResolver

*@ JsonTypeIdResolver*decorator 可以用来插入一个自定义类型标识符处理程序，用于在 JSON 内容中包含的 JavaScript 类型和类型 id 之间进行转换。

**API**:[JsonTypeIdResolver](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsontypeidresolver)—装饰器选项[JsonTypeIdResolverOptions](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsontypeidresolveroptions.html)。

# @JsonUnwrapped

*@JsonUnwrapped* 定义了序列化/反序列化时应该展开/展平的值。

**API**:[JSON unwrapped](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsonunwrapped)—装饰选项 [JsonUnwrappedOptions](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsonunwrappedoptions.html) 。

# @JsonValue

*@JsonValue* decorator 表示修饰的访问器(field 或“getter”方法)的值将用作实例序列化的单个值，而不是收集 Value 属性的常用方法。

**API**:[JsonValue](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsonvalue)—装饰选项 [JsonValueOptions](https://pichillilorenzo.github.io/jackson-js/latest/modules/types.html#jsonvalueoptions) 。

# @JsonView

*@JsonView* decorator 用于指示被修饰的方法或字段定义的属性所属的视图。如果包含多个视图类标识符，该属性将是所有这些标识符的一部分。也可以在类上使用这个装饰器来指示该类型属性的默认视图，除非被每个属性的装饰器覆盖。

**API** : [JsonView](https://pichillilorenzo.github.io/jackson-js/latest/modules/decorators.html#jsonview) —装饰器选项 [JsonViewOptions](https://pichillilorenzo.github.io/jackson-js/latest/interfaces/types.jsonviewoptions.html) 。

# 结论

在下一部分(" [*Jackson-js:客户端(Angular)和服务器(Node.js)端的示例(第 2 部分)*](https://medium.com/@pichillilorenzo/jackson-js-examples-for-client-and-server-side-part-2-7e66df74c851) ")中，我将给出一个使用`[jackson-js](https://github.com/pichillilorenzo/jackson-js)`和 Angular 9 的简单示例，以及两个用于服务器端的示例:一个使用 node . js+[Express](http://expressjs.com/)+SQLite3(with[sequel ze 5](https://sequelize.org/)，另一个使用 Node.js + [LoopBack 4](https://github.com/strongloop/loopback-next) 。