# 我保证这些都很容易(js 承诺)

> 原文：<https://itnext.io/i-promise-these-are-easy-js-promises-c0a6b1f33e7?source=collection_archive---------3----------------------->

![](img/816b6aa25906a4ecaa0bf20c0f42c806.png)

[*点击这里在 LinkedIn*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fi-promise-these-are-easy-js-promises-c0a6b1f33e7) 上分享这篇文章

已经有一段时间了，但我答应过我会写这篇文章，所以就在这里。首先，让我们用一个简单的例子来确定你是否在正确的地方。这里有一个你以前可能用过的承诺:

```
$.get('[https://randomuser.me/api/'](https://randomuser.me/api/'))
  .done(function(data) {
   console.log(data.results[0]);
  })
  .catch(function(err) {
    console.log('Error: ' + err.responseText)
  });
```

很可能你已经使用了上面的片段几十次，却没有真正思考它是如何工作的。首先，它可能有助于理解什么是回调，所以如果你不熟悉这个术语，你可能想读一读我以前关于[回调](https://medium.com/@jgrisafe/callbacks-for-babies-ae5624257a6f)的文章。承诺清理了臭名昭著的回调地狱，你可能听说过。这看起来有点像史蒂夫·巴斯米作为一个代码块的转世。

```
fs.readdir(source, **function** (err, files) {
  getData(function(x){
    getMoreData(x, function(y){
        getMoreData(y, function(z){ 
            ...
        });
    });
});
```

很丑，是吧？别担心，你再也看不到了。承诺旨在让异步代码看起来更像瑞恩·高斯林，但不幸的是，他们并没有完全实现。你必须阅读我在 async/await 上的下一篇文章(抓紧你的裤子，这一篇我会更快)。

那么到底什么是承诺呢？好吧，正如大多数代码一样，这个定义并不公平，特别是如果你只是在学习编码，有些东西看起来仍然像象形文字。承诺本质上是一个表示异步操作完成或失败的对象( [Mozilla](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) )。通俗地说，promise 是一个由异步函数返回的对象，比如 ajax 调用，它*承诺*无论函数成功还是失败都运行代码。promise 对象通常至少有两个主要的方法(函数)，称为`then`和`catch`。不幸的是，有些库，比如 jQuery，喜欢与众不同，用自己的方式实现它们。所以你可能会看到`then`被`done`取代，或者`catch`被`fail`取代。请务必阅读文档！

`then`和`catch`方法各有一个参数，即臭名昭著的回调函数，当函数的目的实现或失败时，将分别运行该函数。让我们用一些代码对此进行分解:

```
// The function below returns a promise object.
// You can think of the the result of this function as an object
// that looks like this { then: function..., catch: function... }
// so that's why you can chain the then/catch functions onto itasynchronousFunction() // immediately returns promise when called // when the asynchronous operation completes,
  // the callback (anonymous function) you pass here is run
  .then(function(result) { // the optional result is passed/injected into your callback
      // now you can use it in something like...
      $('.some-class').text(result.something)
   })

   // if the operation fails the error
   // will be passed/injected into the catch callback
   .catch(function(err) { // do something with the error such as
     // showing it to the user in the html markup or
     console.log(err);
   })
```

开始看起来很像美元。阿贾克斯哈？所以现在你就像“好酷，你刚刚给我看了我已经在用的，非常感谢。”别担心，在我向你展示如何创造你自己的承诺之前，我只是在制定基本规则。

在此之前，让我们提醒你一件关于复试的小事。请记住，回调只是作为参数传递并由另一个函数调用的函数。因此，前面的示例可以这样编写，并且工作方式完全相同:

```
function successCallback(result) {
  $('.some-class').text(result.something)
}function errorCallback(err) {
  console.log(err);
}asynchronousFunction()
  .then(successCallback)
  .catch(errorCallback)
```

请记住，像第一个例子中一样，匿名函数只是编写上述内容的一种更简短的方式，在 javascript 中，函数可以像任何其他数据类型一样作为参数传递！好的，所以，如果事情开始清楚承诺的用法，让我们继续创造它们。如果没有，复制顶部的 jQuery 代码，并对其进行修改，直到您满意为止(它工作了)。

ES2015 或 ES6 具有内置承诺。因此，您可以创建异步函数，而无需导入任何库。所有主流浏览器都支持这一点(显然，除了 Internet Explorer)。所以我们可以这样做:

```
// this function, waitThreeSeconds, returns a promisefunction waitThreeSeconds() { // below is the constructor for the promise. It takes a 
  // callback that accepts two arguments,
  // resolve and reject (functions)
  // you call these functions when the code succeeds or fails return new Promise(function(resolve, reject) { setTimeout(function() { // resolve the promise after 3 seconds
      // the function you pass to 'then' will be called
      resolve(); }, 3000);
  })
}// call the asynchronous function, which returns our promise
waitThreeSeconds() // chain the 'then' method, which accepts a callback to 
  // be called when the promise is rsolved
 .then(function() {
   alert('It\'s been three seconds!');
 }) // we don't need a catch method in this example, because the
 // promise has no potential to fail (be rejected)
```

在 [repl.it](https://repl.it/@jgrisafe/AccomplishedUnlawfulChapter) 上试试这个，自己看吧！正如你在上面看到的，你不需要链接一个`catch`函数，但是如果你的代码有失败的可能，你必须用`reject`在 promise 中处理它，然后在你调用函数时使用`catch`来处理拒绝。

上面的代码在现实生活中没什么用处，所以让我给你举个例子，你可能会用到承诺。假设您想创建一个提示用户上传文件的功能，一旦文件被选中(或未选中)，您想在页面上向用户显示一条消息。检查这个 [jsfiddle](https://jsfiddle.net/jgrisafe/Le7pxr5j/54/) 来查看 html 标记和 css。

```
function promptUserForFile() { return new Promise(function(resolve, reject) { // call the click event on a hidden file input
    $('#file-input').click();

    // set an interval to check if the document regains
    // focus (when the file dialog closes). If no file
    // was chosen, reject the promise
    const interval = setInterval(function() {
        const fileCount = $('#file-input')[0].files.length;
        if (document.hasFocus() && !fileCount) {
          clearInterval(interval)
          reject();
        }
    }, 1000);

    // if the file input changes, resolve the promise
    // and pass it the file name
    $('#file-input').on('change', function() {
      const filePath = $(this).val();
      const fileName = filePath.split('\\').pop();
      clearInterval(interval)
      resolve(fileName);
    })
  })
}$('button').on('click', function() { // call the asynchronous function
  promptUserForFile()

   // when it's resolved (file chosen), set the success text
   .then(function(fileName) {
     $('.file-chosen-message').text('File chosen: ' + fileName);
   })

   // when it's rejected (no file), set the error text
   .catch(function() {
     $('.file-chosen-message')
       .text('no file chosen')
       .addClass('error');
   })
})
```

好吧，我知道这很难接受。但是让我们试着把它分解成一些容易阅读的步骤。

1.  创建一个异步函数`promptUserForFile`，返回一个 promise 对象。
2.  该函数在隐藏的文件输入上调用一个 click 事件，我们选择隐藏它是因为它们很难看，我们需要自己的标记。
3.  promise 在文件输入上设置了一个事件监听器，以监视变化。这仅在用户选择文件时发生。
4.  如果文件输入发生变化，我们解析承诺并传递文件名给它。
5.  它还设置了一个时间间隔，用于观察文档是否重新获得焦点，这将在文件上传对话框关闭时发生。
6.  如果 interval 检测到对话框已经关闭(文档重新获得焦点)，但用户没有选择文件，它将拒绝承诺。
7.  “解决”和“拒绝”都将清除该时间间隔，以便用户下次单击该按钮时可以再次设置。
8.  接下来，我们在自定义按钮上设置了 click listener，它看起来比原生文件输入好得多。
9.  点击事件触发了我们的异步函数`pomptUserForFile`，并将两个回调函数传递给`then`和`catch`。
10.  `then`和`catch`回调将在页面上设置一些文本，无论用户是否选择了文件，也就是承诺被解决或拒绝的时间。
11.  你可以想象我们可以在这些回调中放入任何我们想要的东西，这样我们的承诺处理就非常灵活。例如，如果我们愿意，我们可以在页面上显示图像。

好了，我们已经想出了一个实际的例子，当我们想要使用我们自己的承诺的时候。多好的旅程啊！我希望这篇文章对如何使用和创造承诺有所启发。如果有什么不清楚的，请在评论中说出来，我会做必要的更新！一份爱，乔伊。