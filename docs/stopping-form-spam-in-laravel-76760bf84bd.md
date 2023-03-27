# 阻止 Laravel 中的垃圾邮件

> 原文：<https://itnext.io/stopping-form-spam-in-laravel-76760bf84bd?source=collection_archive---------0----------------------->

用原生的拉勒维尔能力替换瑞普察

几乎每个 Laravel 应用程序都有某种公开的联系方式。让这些表单易于提交是用户的一项要求。问题是，你一天会被垃圾邮件轰炸多次。

像 ReCAPTCHA 这样的解决方案可以完全避免这种情况，但是它们有自己的权衡，可能会让用户感到厌烦。我想向您展示一种本地的 Laravel 方法来阻止大多数垃圾邮件机器人通过您的表单提交。

# 蜜罐

我们将使用一个“[蜜罐](https://en.wikipedia.org/wiki/Honeypot_(computing))”来使用捕捉技术。这只是意味着我们将在我们的应用程序中加入一些专门用于监视垃圾邮件机器人的代码。

![](img/04eeb8450d949e38d9b40f65a6eafbe4.png)

大多数垃圾邮件机器人会进入一个表单，填写所有可用的字段。如果有一系列复选框，机器人被设计为检查所有字段。我们将通过创建一个隐藏的表单字段来利用这一点，我们将在提交时监视该字段。

创建一个简单的联系人表单。我将对 HTML 类使用 Bootstrap，但是标记并不重要。

```
<form action="/contact" method="POST">[@csrf](http://twitter.com/csrf)
 <div class="form-group">
  <label for="name">Full Name</label>
  <input type="text" class="form-control" id="name" name="name" placeholder="Enter your full name" />
 </div>
 <div class="form-group">
  <label for="email">Email address</label>
  <input type="email" class="form-control" id="email" name="email" placeholder="Enter email" />
 </div>
 <div class="form-group">
  <label for="comments">Comments</label>
  <textarea class="form-control" id="comments" rows="3" name="comments"></textarea>
 </div>
 <button type="submit" class="btn btn-primary">Submit</button>
</form>
```

你可以将它提交给一个带有类似于`store`的某种方法的`ContactController`。

```
Route::post('contact', '[[email protected]](https://yabhq.com/cdn-cgi/l/email-protection)');
```

创建一个`ContactController`，并在其上添加`store`方法。您想要激发一个向您自己发送电子邮件或将其添加到您公司的票证系统的事件。无论您的用例是什么，您的代码可能看起来像这样:

```
/**  
 * Get the contact data and email it to ourselves  
 *  
 * @param  \Illuminate\Http\Request  $request  
 * @return \Illuminate\Http\Response  
 */ 
public function store(Request $request) 
{
    $formData = $request->validate([
        'fullname' => 'required',
        'email'    => 'required|email',
        'comments' => 'required'
    ]);
    event(new ContactFormSubmitted($formData);
    return redirect()->back()
        ->withSuccess('Your form has been submitted'); 
}
```

这段代码对人类和你的 T-1000 系列终结者来说会很好用，但是会让所有的垃圾邮件泛滥成灾。

# 仅接受传真

我们想做的技巧是在我们的网站上添加一个永远不应该被检查的表单域。我喜欢将字段命名为*“仅通过传真联系我”*。没有人会检查这一点。

![](img/042a5a60072b5c98404c41a4d7c9627e.png)

将此代码添加到表单中:

```
<div class="form-group" style="display: none;">
 <label for="faxonly">Fax Only
  <input type="checkbox" name="faxonly" id="faxonly" />
 </label>
</div>
```

确保不要省略`style="display: none;"`属性。我们不希望这个字段向用户显示。

您的下一个直觉可能是将这个字段添加到您的验证中，以确保它是空白的。虽然这样做可行，但是您会通知机器人他们的提交没有通过。这可能会导致他们调整机器人来处理你的表单。

当试图击败垃圾邮件发送者和机器人时，有一个重要的规则。**让他们以为自己赢了**。您希望表单提交能够顺利通过并返回成功的响应。

在控制器中添加一个条件，检查`faxonly`字段是否存在。

```
if ($request->faxonly) {
    return redirect()->back()
        ->withSuccess('Your form has been submitted'); 
}
```

如果输入字段被选中，表单将只返回一个响应而不做任何事情。**好人— 1。垃圾邮件发送者-0。**

最后要清理的是我们添加的重复内容。我们将向视图返回相同的成功响应。我们将通过在您的控制器上添加一个`protected`函数来清空我们的代码，该函数包含您将总是发回的响应。

```
/** 
 * The response to always send back to the frontend 
 * 
 * @return \Illuminate\Http\Response 
 */ 
protected function formResponse() 
{ 
    return redirect()->back()
        ->withSuccess('Your form has been submitted'); 
}
```

现在更新你的存储方法，使用新的`formResponse()`

```
public function store(Request $request) 
{ 
    if ($request->faxonly) { 
        return $this->formResponse(); 
    } 
    event(new ContactFormSubmitted($request->all()));
    return $this->formResponse(); 
}
```

此时，您应该会停止大多数垃圾邮件的提交。您可以通过在 HTML 表单中添加大量 JavaScript 来解决这个问题。我个人最喜欢的是让输入**成为必需的，默认选中**，在页面加载后用 JavsScript 取消选中。

我坚持使用上面的解决方案，因为这是一个很好的免费的 JavaScript 方法来阻止机器人。如果你有任何关于如何改进的想法或主意，请[告诉我](https://twitter.com/chrisblackwell)。

*最初发表于*[*https://yabhq.com*](https://yabhq.com/stopping-form-spam-laravel)*。*