# 用 Jest 模拟承诺的 AWS 服务操作调用

> 原文：<https://itnext.io/mock-promisified-aws-service-operation-calls-with-jest-6f11a22a371?source=collection_archive---------1----------------------->

![](img/eb8de494e344506f285f30eb1768b162.png)

那是你读了我的故事后想“这家伙屁都不懂”😆([https://www . pexels . com/photo/photo-of-a-woman-thinking-941555/](https://www.pexels.com/photo/photo-of-a-woman-thinking-941555/))

在 [Mbanq](https://medium.com/u/c174a2050171?source=post_page-----6f11a22a371--------------------------------) 我们在 AWS 上运行大部分服务，并尽可能多地使用 AWS Lambda。

最近我一直在做一个小的 npm 包，它将帮助我们利用 SSM 和 KMS 来管理我们的系统配置。作为 AWS 的大多数服务，SSM 和 KMS 合作得很好。

为了测试新编写的 npm 包，我们不得不模拟承诺的版本`ssm.getParameters(request)`

```
const AWS = require('aws-sdk')
const ssm = AWS.SSM({ region: 'eu-west-1 }) ssm.getParameters(request).promise() // we have to mock the response from this call
```

嘲笑 AWS 有不同的方式。例如，有来自非常酷的 [DWYL](http://DWYL) 的 [aws-mock-sdk](https://github.com/dwyl/aws-sdk-mock) 包。不过，我决定带着纯粹的玩笑去。

![](img/04d76818f9088d2a0e86383d2dccea6f.png)

要使 SSM 的功能可测试，您必须考虑以下因素:

*   在你的函数调用中使用 **ssm** 作为参数，例如`const load = (ssm, keys, expiryMs)`它将帮助你在编写测试时使用被模仿的`ssm`。当然，你也可以`module.exports = { ssm }`连同你想导出的其他函数，但我真的不喜欢这个主意
*   如果你想检查一个`async/await`函数中抛出的错误，你必须使用:`expect(yourFunc()).rejects.toEqual(new Error(`Error Message`))`。常规的`expect(yourFunc()).toThrowError(`Error Message`)` [**行不通的**](https://github.com/facebook/jest/issues/1700#issuecomment-377890222)

好吧，现在你可能会问自己:

> 你怎么能嘲笑承诺的 AWS 服务操作调用呢？

您可能想要模拟来自`ssm.getParameters(request).promise()`的成功响应，或者模拟这个函数调用抛出的`Error`。

## 成功响应

首先，用`promise`键创建一个 js 对象，用将返回一个`Promise`的`jest.fn().mockImplementation()`模拟`promise`的值，当解析后返回一个成功的响应。

然后每当调用`getParameters()`函数时，返回创建的`ssmPromise`。

```
const AWS = require('aws-sdk')
let ssm = new AWS.SSM() 
const ssmPromise = {  
  promise: jest.fn().mockImplementation((request) => {    
    return new Promise((resolve, reject) => {      
      const response = {        
        Parameters: [          
          {            
            Name: 'bar',            
            Type: 'String',            
            Value: 'barfoorista',            
            Version: 1,            
            LastModifiedDate: '2018-08-22T13:49:55.717Z',                   
            ARN: 'arn:aws:ssm:eu-west-1:whatever:parameter/bar'               
          },          
          {            
            Name: 'foo',            
            Type: 'String',            
            Value: 'foobarista',            
            Version: 1,            
            LastModifiedDate: '2018-08-22T13:49:41.486Z',            
            ARN: 'arn:aws:ssm:eu-west-1:whatever:parameter/foo'          
           }        
         ],        
         InvalidParameters: []      
       }      
       resolve(response)    
     })  
  })
}
ssm = { getParameters: () => { return ssmPromise } }
```

## 抛出一个错误

通过使用您在测试顶部创建的`ssm`实例，您还可以一次性模仿`ssm.getParameters()`。

这是一个你如何模仿`ssm.getParameters()`抛出`Error`的例子

```
ssm = {      
  getParameters: () => {        
    return {          
      promise: jest.fn().mockImplementation((request) => {            
        return new Promise((resolve, reject) => {              
          return reject(new Error('foobar'))            
        }).catch(() => console.log('Ok'))          
      })        
    }      
  }    
}
```

在`gist`(双关语)中，你可以看到测试的一部分，我们用它来测试我们的包。

该软件包仍在开发中。一旦它发布，你就可以通过运行`npm i --save sreda`来安装它