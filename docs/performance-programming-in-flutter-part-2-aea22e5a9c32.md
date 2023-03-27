# 颤振的性能规划第二部分

> 原文：<https://itnext.io/performance-programming-in-flutter-part-2-aea22e5a9c32?source=collection_archive---------8----------------------->

***在 Flutter 中使用低级分离 API***

在我之前的[文章](/performance-programming-in-flutter-using-isolates-part-1-997a29011e88)中，我解释了如何在高层次上使用隔离。但是使用高级 API 有一些缺点。

通过在高级计算功能上使用低级 API，您可以获得对隔离的更多控制。您可以在任何时间点*暂停、恢复和停止*隔离，这是使用“*计算*功能无法实现的。

**让我们看看如何实现这些目标。**

# **观看视频**

颤振分离物

在这里，我创建了一个自定义类来发送数据给**隔离**。我称之为线程参数。

```
class ThreadParams {
  ThreadParams(this.val, this.sendPort);
  int val;
  SendPort sendPort;
}
```

在这里，您可以看到其中一个参数是发送端口。这是隔离与呼叫隔离通信
的端口。

还有另一个名为 ReceivePort 的类，调用的 Isolate 类通过它取回数据。

# **开始隔离线程**

这就是 start 方法的样子。

```
Isolate _isolate;
 bool _running = false;
 ReceivePort _receivePort;
 Capability _capability;

 void _start() async {
    if (_running) {
      return;
    }
    setState(() {
      _running = true;
    });
    _receivePort = ReceivePort();
    ThreadParams threadParams = ThreadParams(2000,     _receivePort.sendPort);
    _isolate = await Isolate.spawn(
      _isolateHandler,
      threadParams,
    );
    _receivePort.listen(_handleMessage, onDone: () {
      setState(() {
        _threadStatus = 'Stopped';
      });
    });
}
```

将会创建一个新的隔离线程。调用方法通过 *_receivePort.listen* 监听来自 *_isolate* 的消息，receivePort.listen 具有接收消息的功能。

*_handleMessage* 可以是这样一个简单的函数。

```
void _handleMessage(dynamic data) {
    print(data.toString());
}
```

*_isolateHandler* 方法是 _isolate 的入口方法，它应该是静态方法或者不应该是类中的方法。

下面是 *_isolateHandler* ，它包含了我们将要在 *_isolate* 线程内部执行的大量操作方法。

```
static void _isolateHandler(ThreadParams threadParams) async {
    heavyOperation(threadParams);
  }

  static void heavyOperation(ThreadParams threadParams) async {
    int count = 10000;
    while (true) {
      int sum = 0;
      for (int i = 0; i < count; i++) {
        sum += await computeSum(1000);
      }
      count += threadParams.val;
      threadParams.sendPort.send(sum.toString());
    }
  }

  static Future<int> computeSum(int num) {
    Random random = Random();
    return Future(() {
      int sum = 0;
      for (int i = 0; i < num; i++) {
        sum += random.nextInt(100);
      }
      return sum;
    });
  }
```

# **暂停、恢复和停止隔离**

```
void _pause() {
  if (null != _isolate) {
    _paused ? _isolate.resume(_capability) : _capability = _isolate.pause();
    setState(() {
      _paused = !_paused;
      _threadStatus = _paused ? 'Paused' : 'Resumed';
    });
  }
}

void _stop() {
  if (null != _isolate) {
    setState(() {
      _running = false;
    });
    _receivePort.close();
    _isolate.kill(priority: Isolate.immediate);
    _isolate = null;
  }
}
```

这里要理解的主要事情是暂停隔离是为了恢复隔离，我们需要一个在暂停隔离时可以得到的能力对象。隔离暂停将返回可用于恢复隔离能力。

# **发送消息**

`threadParams.sendPort.send(sum.toString());`

*SendPort.send* 用于将消息发送回调用隔离。我这里所说的隔离是指运行 Flutter 应用程序的主隔离。

这是最基本的。

# 完整示例

这是一个完整的例子。

```
import 'package:flutter/material.dart';
import 'dart:isolate';
import 'dart:math';
import 'dart:async';

class PerformancePage extends StatefulWidget {
  final String title = "Isolates Demo";

  [@override](http://twitter.com/override)
  _PerformancePageState createState() => _PerformancePageState();
}

class _PerformancePageState extends State<PerformancePage> {
  //
  Isolate _isolate;
  bool _running = false;
  bool _paused = false;
  String _message = '';
  String _threadStatus = '';
  ReceivePort _receivePort;
  Capability _capability;

  void _start() async {
    if (_running) {
      return;
    }
    setState(() {
      _running = true;
      _message = 'Starting...';
      _threadStatus = 'Running...';
    });
    _receivePort = ReceivePort();
    ThreadParams threadParams = ThreadParams(2000, _receivePort.sendPort);
    _isolate = await Isolate.spawn(
      _isolateHandler,
      threadParams,
    );
    _receivePort.listen(_handleMessage, onDone: () {
      setState(() {
        _threadStatus = 'Stopped';
      });
    });
  }

  void _pause() {
    if (null != _isolate) {
      _paused ? _isolate.resume(_capability) : _capability = _isolate.pause();
      setState(() {
        _paused = !_paused;
        _threadStatus = _paused ? 'Paused' : 'Resumed';
      });
    }
  }

  void _stop() {
    if (null != _isolate) {
      setState(() {
        _running = false;
      });
      _receivePort.close();
      _isolate.kill(priority: Isolate.immediate);
      _isolate = null;
    }
  }

  void _handleMessage(dynamic data) {
    setState(() {
      _message = data;
    });
  }

  static void _isolateHandler(ThreadParams threadParams) async {
    heavyOperation(threadParams);
  }

  static void heavyOperation(ThreadParams threadParams) async {
    int count = 10000;
    while (true) {
      int sum = 0;
      for (int i = 0; i < count; i++) {
        sum += await computeSum(1000);
      }
      count += threadParams.val;
      threadParams.sendPort.send(sum.toString());
    }
  }

  static Future<int> computeSum(int num) {
    Random random = Random();
    return Future(() {
      int sum = 0;
      for (int i = 0; i < num; i++) {
        sum += random.nextInt(100);
      }
      return sum;
    });
  }

  [@override](http://twitter.com/override)
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text(widget.title),
      ),
      body: new Container(
        padding: EdgeInsets.all(20.0),
        alignment: Alignment.center,
        child: new Column(
          children: <Widget>[
            !_running
                ? OutlineButton(
                    child: Text('Start Isolate'),
                    onPressed: () {
                      _start();
                    },
                  )
                : SizedBox(),
            _running
                ? OutlineButton(
                    child: Text(_paused ? 'Resume Isolate' : 'Pause Isolate'),
                    onPressed: () {
                      _pause();
                    },
                  )
                : SizedBox(),
            _running
                ? OutlineButton(
                    child: Text('Stop Isolate'),
                    onPressed: () {
                      _stop();
                    },
                  )
                : SizedBox(),
            SizedBox(
              height: 20.0,
            ),
            Text(
              _threadStatus,
              style: TextStyle(
                fontSize: 20.0,
              ),
            ),
            SizedBox(
              height: 20.0,
            ),
            Text(
              _message,
              style: TextStyle(
                fontSize: 20.0,
                color: Colors.green,
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class ThreadParams {
  ThreadParams(this.val, this.sendPort);
  int val;
  SendPort sendPort;
}
```

就是这样。

如果你觉得这篇文章有用，请鼓掌。

*感谢阅读，如果你觉得我的帖子有用，请在下面留下你的宝贵意见，订阅我的* [*youtube 频道*](https://www.youtube.com/channel/UC5lbdURzjB0irr-FTbjWN1A) *获取更多视频。*