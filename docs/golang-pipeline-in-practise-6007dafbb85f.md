# 实践中的戈朗管道

> 原文：<https://itnext.io/golang-pipeline-in-practise-6007dafbb85f?source=collection_archive---------2----------------------->

![](img/ebfa9731c15b78e43306b00575b55576.png)

我有一个请求，执行一些关键字的谷歌 API 搜索，并将结果(网址)保存到文本文件中。这是使用 Go 的并发模式来组装管道的好机会，因此，并行化 IO 和 CPU。

## 谷歌搜索 API

给定一个关键词，比方说一个类别，下面的函数使用 [go-resty](https://github.com/go-resty/resty) 调用 Google search API，然后使用 [gjson](https://github.com/tidwall/gjson) 库从结果中选取相应的链接。

```
func SearchGoogle(category string, count int) ([]SearchResult, error) {
 results := []SearchResult{}
 // *... skip some initial assignment*
 for i := 0; i < pages; i++ {
  resp, err := client.R().
   SetQueryParams(map[string]string{
    "cx":  cx,
    "key": key,
    "q":   category,
    "start": fmt.Sprintf("%d", i*10+1)
   }).
   SetHeader("Accept", "application/json").
   Get("https://www.googleapis.com/customsearch/v1") //... *skip the error handling part*  content := resp.Body()
  if resp.StatusCode() != 200 {
   logrus.Errorf("staus code is non 200, code:%v", resp.StatusCode())
   return results, err
  }
  gjson.GetBytes(content, "items").ForEach(func(_, elm gjson.Result) bool {
   counter += 1
   title := elm.Get("title").String()
   link := elm.Get("link").String()
   snippet := elm.Get("snippet").String()

   results = append(results, SearchResult{
    Category: category,
    Title:    title,
    Link:     link,
    Snippet:  snippet,
   })
   return true //keep looping
  })
 }
 return results, nil
}
```

我们读取关键字来搜索一段字符串。让我们用通道建立 golang 管道。

## 数据馈送

要启动管道，接收一段字符串并输出一个字符串通道。

```
func Feed(list []string) chan string {
 out := make(chan string)
 go func() {
  for _, v := range list {
   out <- v
  }
  close(out)
 }() return out
}
```

首先，创造“出”禅。然后创建一个 goroutine 来遍历关键字列表，将它发送到输出通道。一旦循环完成，关闭输出通道以通知接收端，它关闭了。

请注意，除非管道已连接并正在移动，否则发送到通道的操作会被阻塞。这就是为什么我们实际上需要运行它作为一个 goroutine。

## 搜索

在数据馈送之后，我们为搜索构造动作。常见的模式是，用一个输入通道拾取输入，构造一个输出通道，处理数据并将结果发送到输出通道，并将输出返回给管道组件。

```
func Search(worker int, in chan string) chan []SearchResult {
 out := make(chan []SearchResult) go func() {
  for text := range in {
   result, err := SearchGoogle(text, 100)
   if err != nil {
    logrus.Errorf("Failed to search: %v", text)
    return
   }
   out <- result
  } close(out)
 }() return out
}
```

我们在`*in*`通道上执行范围循环，一旦`*in*`通道关闭，范围循环将停止，我们相应地关闭输出通道。对于从通道中获取的每个文本，运行 SearchGoogle()，并将结果发送到输出通道。

工人参数只是工人的一个指标。

## 扇出

为了并行搜索，我们使用扇出模式。基本上，在管道上，接收一个输入通道，输出一系列通道。看看这个。

```
func Fanout(workers int, ch chan string) []chan []SearchResult {
 chs := []chan []SearchResult{}
 for i := 0; i < workers; i++ {
  chs = append(chs, Search(i, ch))
 }
 return chs
}
```

在函数中，为 SearchResult 切片构造一个通道切片。注意，Search 函数用于返回 SearchResult 的一个通道。

## 扇入

在搜索分支之后，现在我们需要使用扇入模式来合并结果。

```
func Fanin(chans []chan []SearchResult) chan SearchResult {
 out := make(chan SearchResult) var wg sync.WaitGroup
 wg.Add(len(chans)) for _, ch := range chans {
  go func(c chan []SearchResult) {
   for list := range c {
    for _, v := range list {
     out <- v
    }
   } wg.Done()
  }(ch)
 } go func() {
  wg.Wait()
  close(out)
 }() return out
}
```

与 common 模式相同，该函数接受一部分通道，但返回一个通道。这里引入 Waitgroup 是为了确保所有的扇出通道都完成它们的工作。

循环通过输入通道片，在输出通道上发送结果。注意，为了避免在 for 循环中运行 goroutine 的常见缺陷，我们将通道作为参数传递给 goroutine 函数。

扇出通道上的范围环路完成后，标记工作组完成。

在另一个 goroutine 之外，等待所有扇出通道完成，并关闭输出通道。

## 救援

最后，我们使用 [gocsv](https://github.com/gocarina/gocsv) 模块将搜索结果保存到一个 CSV 文件中。

```
func Save(filename string, in chan SearchResult, done chan struct{}) {
 csvFile, err := os.OpenFile("results.csv", os.O_RDWR|os.O_CREATE, os.ModePerm)
 if err != nil {
  logrus.Fatalf("Failed to open results file:%v", err)
 } go func() {
  for result := range in {
   list := []SearchResult{result}
   bf := bytes.NewBufferString("")
   err := gocsv.MarshalCSVWithoutHeaders(&list, gocsv.NewSafeCSVWriter(csv.NewWriter(bf)))
   if err != nil {
    logrus.Errorf("Failed to marshal results into csv:%v", err)
   } csvFile.WriteString(bf.String())
  } csvFile.Close()   
  done <- struct{}{}

 }()}
```

gocsv 与自定义编写器一起使用，忽略从通道中获取的每个结果的头。

传入一个`done`通道来表示所有工作都已完成。

## 装配

在主函数中组装管道的时间，

```
catList := []string{}
//skip the reading and populating for the catListdone := make(chan struct{})
Save("results.csv", **Fanin(Fanout(5, Feed(catList)))**, done)for {
  select {
  case <-done:
   logrus.Infof("done")
   os.Exit(0)
  }
 }
```

由于仔细定义了管道函数的输入/输出，所以组装相对容易，尽管与其他语言相比可能不那么优雅；)

`for and select`回路用于等待保存完成的信号。