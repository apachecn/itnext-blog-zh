# JavaScript 中的异步性

> 原文：<https://itnext.io/asynchronicity-in-javascript-93abb3a3f81b?source=collection_archive---------3----------------------->

![](img/234e52a54cab14df45f16394a68e6d2f.png)

图像可能受版权保护

# 前言

为了对 JS 异步性有一个基本的了解，你可以在[深入了解 JS 异步性](https://github.com/n0ruSh/the-art-of-reading/issues/1)。如果没有对异步性(例如事件循环、事件队列等)的深刻理解，setTimeout/setInterval、ajax 在浏览器、节点 IO 中的应用将不会走得很远。).

# 空谈是廉价的，给我看看代码

假设我们有一个包含文件名列表的数组。我们希望依次读取这些文件，直到成功检索到一个文件。比如，如果数组是['a.txt '，' b.txt']，我们先读取 *a.txt* ，如果读取成功，我们返回 *a.txt* 的文件内容。否则我们继续读 *b.txt* 。为了读取文件，Nodes 提供了两个 API，一个是 sync [readFileSync](https://nodejs.org/api/fs.html#fs_fs_readfilesync_path_options) ，另一个是 async [readFile](https://nodejs.org/api/fs.html#fs_fs_readfile_path_options_callback) 。

现在假设我们有两个文件: *a.txt* (其内容也是 *a.txt* )和 *b.txt* (其内容也是 *b.txt* )。

同步解决方案非常简单:

```
let fs = require('fs'),
    path = require('path');function readOneSync(files) {
    for(let i = 0, len = files.length; i < len; i++) {
        try {
            return fs.readFileSync(path.join(__dirname, files[i]), 'utf8');
        } catch(e) {
            //ignore
        }
    }
    // all fail, throw an exception
    throw new Error('all fail');
}console.log(readOneSync(['a.txt', 'b.txt'])); //a.txt
console.log(readOneSync(['filenotexist', 'b.txt'])); //b.txt
```

同步读取的主要问题是它会阻塞主线程和事件队列的循环。如果读取需要很长时间才能完成，特别是当文件很大时，程序会变得不活跃。异步读取可以有效地避免这个问题。我们需要注意的只是处理文件读取的顺序(即在前面的 *readFile* 调用的回调中读取下一个文件)。

```
let fs = require('fs'),
    path = require('path');function readOne(files, cb) {
    function next(index) {
        let fileName = files[index];
        fs.readFile(path.join(__dirname, fileName), 'utf8', (err, data) => {
            if(err) {
                return next(index + 1); // if fail, read next file
            } else {
                return cb(data); // use cb to output the result
            }
        });
    }
    next(0);
}readOne(['a.txt', 'b.txt'], console.log); //a.txt
readOne(['filenotexist', 'b.txt'], console.log); //b.txt
```

异步解决方案需要接受另一个参数(即 *cb* )来处理结果。它还定义了一个 *next* 方法来递归读取下一个文件。

# 同时激发多个异步请求。

假设我们有一个包含文件名列表的数组，我们的目标是同时读取文件，如果所有读取都成功，就返回所有文件内容。如果其中任何一个失败，则调用失败的回调。

```
let fs = require('fs'),
    path = require('path');function readAllV1(files, onsuccess, onfail) {
    let result = [];
    files.forEach(file => {
        fs.readFile(path.join(__dirname, file), 'utf8', (err, data) => {
            if(err) {
                onfail(err);
            } else {
                result.push(data);
                if(result.length === files.length) {
                    onsuccess(result);
                }
            }
        });
    });
}readAllV1(['a.txt', 'b.txt'], console.log, console.log);
```

上面的实现有一个明显的问题:结果中*文件内容的顺序与*文件*中的顺序不匹配。所有读取操作都是异步的，因此当读取完成时，回调被插入到事件队列中。假设*文件*为['a.txt '，【b.txt']，则 *a.txt* 和 *b.txt* 的文件大小分别为 100M 和 10kb。当我们同时异步读取这两个文件时， *b.txt* 的读取将在 *a.txt* 之前完成，因此 *b.txt* 的回调将在事件队列中 *a.txt* 的回调之前。最终的*结果*将会是[${content of *b.txt* }，${content of *a.txt* }]。如果我们希望*结果*中文件内容的顺序遵循*文件*中文件名的顺序，我们可以对我们的实现做一个小的修改:*

```
let fs = require('fs'),
    path = require('path');function readAllV2(files, onsuccess, onfail) {
    let result = [];
    files.forEach((file, index) => {
        fs.readFile(path.join(__dirname, file), 'utf8', (err, data) => {
            if(err) {
                onfail(err);
            } else {
                result[index] = data;
                if(result.length === files.length) {
                    onsuccess(result);
                }
            }
        });
    });
}readAllV2(['a.txt', 'b.txt'], console.log, console.log); //结果不确定性
```

乍一看好像很管用，但是！

```
let arr = [];
arr[1] = 'a';
console.log(arr.length); //2
```

基于 *readAllV2* 的实现，如果读取 *b.txt* 在 *a.txt* 之前完成，那么我们设置 result[1]= $ { content of*b . txt*}，导致*result . length = = = files . length*为真。在这种情况下，我们调用成功回调来终止函数，而没有得到 *a.txt* 的结果。因此，我们不能简单地依靠 *result.length* 作为补充指标。

```
let fs = require('fs'),
    path = require('path');function readAllV3(files, onsuccess, onfail) {
    let result = [], counter = 0;
    files.forEach((file, index) => {
        fs.readFile(path.join(__dirname, file), 'utf8', (err, data) => {
            if(err) {
                onfail(err);
            } else {
                result[index] = data;
                counter++;
                if(counter === files.length) {
                    onsuccess(result);
                }
            }
        });
    });
}readAllV3(['a.txt', 'b.txt'], console.log, console.log); //[ 'a.txt', 'b.txt' ]
```

如果你对承诺有点熟悉，你可能知道有一个[的承诺方法，它做完全一样的事情。](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)

# 让你的界面一致

让我们实现具有缓存功能的自定义读取文件方法。如果缓存可用于文件，我们只需返回缓存。否则，我们读取文件并设置缓存。

```
let fs = require('fs'),
    path = require('path'),
    cache = {};function readWithCacheV1(file, onsuccess, onfail) {
    if(cache[file]) {
        onsuccess(cache[file]);
    } else {
       fs.readFile(path.join(__dirname, file), 'utf8', (err, data) => {
           if(err) {
               onfail(err);
           } else {
               cache[file] = data;
               onsuccess(data);
           }
       });
    }
}
```

让我们深入了解一下:

*   当缓存可用时，我们调用*on success***SYNCHRONOUS**

```
cache['a.txt'] = 'hello'; //mock cache data
readWithCacheV1('a.txt', console.log);//synchronous, completes before going into next call.
console.log('after you');//console output:
hello
after you
```

*   当缓存不可用时，由于 *readFile* 的异步性，它是**异步**

```
readWithCacheV1('a.txt', console.log);
console.log('after you');//console output:
after you
hello
```

这种不一致性通常会导致难以跟踪和调试的隐藏错误。我们可以改进解决方案，使其行为一致。

```
let fs = require('fs'),
    path = require('path'),
    cache = {};function readWithCacheV2(file, onsuccess, onfail) {
    if(cache[file]) {
        setTimeout(onsuccess.bind(null, cache[file]),0);
    } else {
       fs.readFile(path.join(__dirname, file), 'utf8', (err, data) => {
           if(err) {
               onfail(err);
           } else {
               cache[file] = data;
               onsuccess(data);
           }
       });
    }
}
```

让我们重新审视两个用例:

*   有缓存可用`` JavaScript cache[' a . txt ']= ' hello '；readWithCacheV2('a.txt '，console . log)；console . log(' after you ')；

//控制台输出:你好

```
* without cache```javascript
readWithCacheV2('a.txt', console.log);
console.log('after you');//console output:
after you
hello
```

# 参考

*   [有效的 JavaScript](https://www.amazon.com/Effective-JavaScript-Specific-Software-Development/dp/0321812182/ref=sr_1_3?s=books&ie=UTF8&qid=1521248523&sr=1-3&keywords=Effective+JavaScript)

# 代码示例

*   [有效的 JavaSript 示例代码](https://github.com/n0ruSh/the-art-of-reading/tree/master/javascript/Effective%20Javascript)

# 通知；注意

*   如果您想了解最新的新闻/文章，请点击[【观看】](https://github.com/n0ruSh/the-art-of-reading)订阅。