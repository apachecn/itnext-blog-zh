# 颤动中的条形图

> 原文：<https://itnext.io/bar-charts-in-flutter-8a10c3592bbf?source=collection_archive---------0----------------------->

在本文中，我们将看到如何在 flutter 中绘制条形图。

# 观看视频教程

颤动中的条形图

要在 flutter 中绘制图表，我们需要添加一个依赖项。因此，打开您的 *pubspec.yaml* 文件，并在 dependencies 下面添加下面的依赖项。

# 属国

```
flutter: 
    sdk: flutter

  # The following adds the Cupertino Icons font to your application.
  # Use with the CupertinoIcons class for iOS style icons.
  cupertino_icons: ^0.1.2
  ... 
  charts_flutter: ^0.9.0
```

在这之后，运行'*扑包得到*添加包。

一旦安装了插件，将它导入到您想要使用的文件中。

```
**import** 'package:charts_flutter/flutter.dart' as charts;
```

让我们首先为图表创建一些样本数据

首先，我们将创建一个保存图表数据的类。这里的课程叫做“销售”

```
class Sales {
  final String year;
  final int sales;

  Sales(this.year, this.sales);
}
```

现在我们将创建一个列表来保存一系列图表数据。
**_createRandomData** 创建一些随机的销售数据。

```
List<charts.Series> seriesList;

static List<charts.Series<Sales, String>> _createRandomData() {
   final random = Random();

   final desktopSalesData = [
     Sales('2015', random.nextInt(100)),
     Sales('2016', random.nextInt(100)),
     Sales('2017', random.nextInt(100)),
     Sales('2018', random.nextInt(100)),
     Sales('2019', random.nextInt(100)),
   ];

   return [
     charts.Series<Sales, String>(
       id: 'Sales',
       domainFn: (Sales sales, _) => sales.year,
       measureFn: (Sales sales, _) => sales.sales,
       data: desktopSalesData,
       fillColorFn: (Sales sales, _) {
         return charts.MaterialPalette.blue.shadeDefault;
       },
     )
   ];

 }
```

# 添加图表

首先，我们将初始化序列列表，然后将它传递给条形图构造函数。

```
[@override](http://twitter.com/override)
void initState() {
  super.initState();
  seriesList = _createRandomData();
}

barChart() {
  return charts.BarChart(
    seriesList,
    animate: true,
    vertical: false,
  );
}
```

好的。结束了。就这么简单。

如果要添加堆积或分组条形图，只需像这样添加更多系列。

```
static List<charts.Series<Sales, String>> _createRandomData() {
    final random = Random();

    final desktopSalesData = [
      Sales('2015', random.nextInt(100)),
      Sales('2016', random.nextInt(100)),
      Sales('2017', random.nextInt(100)),
      Sales('2018', random.nextInt(100)),
      Sales('2019', random.nextInt(100)),
    ];

    final tabletSalesData = [
      Sales('2015', random.nextInt(100)),
      Sales('2016', random.nextInt(100)),
      Sales('2017', random.nextInt(100)),
      Sales('2018', random.nextInt(100)),
      Sales('2019', random.nextInt(100)),
    ];

    final mobileSalesData = [
      Sales('2015', random.nextInt(100)),
      Sales('2016', random.nextInt(100)),
      Sales('2017', random.nextInt(100)),
      Sales('2018', random.nextInt(100)),
      Sales('2019', random.nextInt(100)),
    ];

    return [
      charts.Series<Sales, String>(
        id: 'Sales',
        domainFn: (Sales sales, _) => sales.year,
        measureFn: (Sales sales, _) => sales.sales,
        data: desktopSalesData,
        fillColorFn: (Sales sales, _) {
          return charts.MaterialPalette.blue.shadeDefault;
        },
      ),
      charts.Series<Sales, String>(
        id: 'Sales',
        domainFn: (Sales sales, _) => sales.year,
        measureFn: (Sales sales, _) => sales.sales,
        data: tabletSalesData,
        fillColorFn: (Sales sales, _) {
          return charts.MaterialPalette.green.shadeDefault;
        },
      ),
      charts.Series<Sales, String>(
        id: 'Sales',
        domainFn: (Sales sales, _) => sales.year,
        measureFn: (Sales sales, _) => sales.sales,
        data: mobileSalesData,
        fillColorFn: (Sales sales, _) {
          return charts.MaterialPalette.teal.shadeDefault;
        },
      )
    ];
  }

  barChart() {
    return charts.BarChart(
      seriesList,
      animate: true,
      vertical: false,
      barGroupingType: charts.BarGroupingType.grouped,
      defaultRenderer: charts.BarRendererConfig(
        groupingType: charts.BarGroupingType.grouped,
        strokeWidthPx: 1.0,
      ),
      // domainAxis: charts.OrdinalAxisSpec(
      //   renderSpec: charts.NoneRenderSpec(),
       ),
    );
  }
```

![](img/31573030d1a65a61fbd5e375cc6c1f55.png)

# 完全码

```
import 'package:flutter/material.dart';
import 'dart:math';
import 'package:charts_flutter/flutter.dart' as charts;

class ChartsDemo extends StatefulWidget {
  //
  ChartsDemo() : super();

  final String title = "Charts Demo";

  [@override](http://twitter.com/override)
  ChartsDemoState createState() => ChartsDemoState();
}

class ChartsDemoState extends State<ChartsDemo> {
  //
  List<charts.Series> seriesList;

  static List<charts.Series<Sales, String>> _createRandomData() {
    final random = Random();

    final desktopSalesData = [
      Sales('2015', random.nextInt(100)),
      Sales('2016', random.nextInt(100)),
      Sales('2017', random.nextInt(100)),
      Sales('2018', random.nextInt(100)),
      Sales('2019', random.nextInt(100)),
    ];

    final tabletSalesData = [
      Sales('2015', random.nextInt(100)),
      Sales('2016', random.nextInt(100)),
      Sales('2017', random.nextInt(100)),
      Sales('2018', random.nextInt(100)),
      Sales('2019', random.nextInt(100)),
    ];

    final mobileSalesData = [
      Sales('2015', random.nextInt(100)),
      Sales('2016', random.nextInt(100)),
      Sales('2017', random.nextInt(100)),
      Sales('2018', random.nextInt(100)),
      Sales('2019', random.nextInt(100)),
    ];

    return [
      charts.Series<Sales, String>(
        id: 'Sales',
        domainFn: (Sales sales, _) => sales.year,
        measureFn: (Sales sales, _) => sales.sales,
        data: desktopSalesData,
        fillColorFn: (Sales sales, _) {
          return charts.MaterialPalette.blue.shadeDefault;
        },
      ),
      charts.Series<Sales, String>(
        id: 'Sales',
        domainFn: (Sales sales, _) => sales.year,
        measureFn: (Sales sales, _) => sales.sales,
        data: tabletSalesData,
        fillColorFn: (Sales sales, _) {
          return charts.MaterialPalette.green.shadeDefault;
        },
      ),
      charts.Series<Sales, String>(
        id: 'Sales',
        domainFn: (Sales sales, _) => sales.year,
        measureFn: (Sales sales, _) => sales.sales,
        data: mobileSalesData,
        fillColorFn: (Sales sales, _) {
          return charts.MaterialPalette.teal.shadeDefault;
        },
      )
    ];
  }

  barChart() {
    return charts.BarChart(
      seriesList,
      animate: true,
      vertical: false,
      barGroupingType: charts.BarGroupingType.grouped,
      defaultRenderer: charts.BarRendererConfig(
        groupingType: charts.BarGroupingType.grouped,
        strokeWidthPx: 1.0,
      ),
      domainAxis: charts.OrdinalAxisSpec(
        renderSpec: charts.NoneRenderSpec(),
      ),
    );
  }

  [@override](http://twitter.com/override)
  void initState() {
    super.initState();
    seriesList = _createRandomData();
  }

  [@override](http://twitter.com/override)
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Container(
        padding: EdgeInsets.all(20.0),
        child: barChart(),
      ),
    );
  }
}

class Sales {
  final String year;
  final int sales;

  Sales(this.year, this.sales);
}
```

如果您觉得这篇文章有帮助，请留下您的宝贵反馈。

在这里找到更多的[颤振教程](https://www.youtube.com/channel/UC5lbdURzjB0irr-FTbjWN1A)。

谢了。