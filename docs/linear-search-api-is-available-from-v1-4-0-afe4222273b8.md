# 线性搜索 API 从 1.4.0 版开始提供

> 原文：<https://itnext.io/linear-search-api-is-available-from-v1-4-0-afe4222273b8?source=collection_archive---------0----------------------->

![](img/2e869ae0fe8a6bbd5f8b072797d9146c.png)

[从 v1.4](https://vdaas-vald.medium.com/release-announcement-v1-4-0-5c65a48aaa25) 开始，Vald 支持线性搜索 API，因为 NGT 从 [v1.13.8](https://github.com/yahoojapan/NGT/releases/tag/v1.13.8) 发布了线性搜索 CAPI。这篇文章介绍了什么是线性搜索和线性搜索 API 的用法。

注意:

```
Current Linear Search API has **a timeout setting as the mandatory setting**. Please set a long time as `Search_Config.timeout`.
This configuration will be optional soon.
```

# 什么是线性搜索？

线性搜索是最简单的最近邻搜索算法。它通过计算查询向量(输入)和已经存在于一些数据库或类似数据库中的所有目标向量之间的距离来搜索最近的向量。由于计算所有存储数据之间的距离，输入查询的搜索结果将总是相同的，因此搜索精度可以是恒定的。此外，处理时间与存储向量的数量及其维数成比例增加。

# 在 Vald 中使用线性搜索的情况

本捕手介绍线性搜索 API 的使用场景。

## 使用它作为搜索方法

第一种是用它做线性搜索。Vald 将向量索引到多路 Vald 代理盒中，允许您并行执行线性搜索。它比人工神经网络(使用搜索 API)需要更多的搜索时间，但它比使用一个向量集的一般线性搜索执行得更快。

## 调整人工神经网络搜索的准确性验证

另一种方法是在使用搜索 API 时将其用于参数调整。您可以通过设置搜索 API 的参数来调整搜索精度和搜索处理时间。例如，搜索配置参数之一 radius 和 epsilon 指定搜索范围。(半径和ε是 NGT 特有的搜索参数。详情请参考[此处](https://github.com/yahoojapan/NGT/wiki)。

# 如何使用 Vald 的线性搜索

本章将以`vald-client-go`为例，展示如何使用线性搜索 API。Vald 提供了客户端库并发布了 gRPC proto 文件，因此您可以将它们与您喜欢的编程语言一起使用。

## 示例代码

下面是使用线性搜索 API 的示例代码。该示例使用`fashion-mnist-784-euclidean`作为样本数据集。

```
package main

// import packages
import (
	"context"
	"encoding/json"
	"flag"
	"os"
	"time"

	"github.com/kpango/fuid"
	"github.com/kpango/glg"
	"github.com/vdaas/vald-client-go/v1/payload"
	"github.com/vdaas/vald-client-go/v1/vald"
	"gonum.org/v1/hdf5"
	"google.golang.org/grpc"
)

const (
	insertCount = 10000     // the number of insert vectors
	testCount   = 10        // the number of search queries
	Num         = 5         // the number of nearest neighbors per search query
	Radius      = -1        // search area parameter of NGT. -1 means infinite circle.
	Timeout     = 100000000 // deadline time for searching
)

var (
	datasetPath         string
	grpcServerAddr      string
	indexingWaitSeconds uint
)

func init() {
	flag.StringVar(&datasetPath, "path", "fashion-mnist-784-euclidean.hdf5", "dataset path")
	flag.StringVar(&grpcServerAddr, "addr", "localhost:8081", "gRPC server address")
	flag.UintVar(&indexingWaitSeconds, "wait", 300, "indexing wait seconds")
	flag.Parse()
}

func main() {
        // load dataset
	ids, train, test, err := load(datasetPath)
	if err != nil {
		glg.Fatal(err)
	}

        // create context
	ctx := context.Background()

        // create session to Vald cluster
	conn, err := grpc.DialContext(ctx, grpcServerAddr, grpc.WithInsecure())
	if err != nil {
		glg.Fatal(err)
	}

        // create new client to contact with Vald cluster.
	client := vald.NewValdClient(conn)

	glg.Infof("Start Inserting %d training vector to Vald", insertCount)
	for i := range ids[:insertCount] {
                // Send the insert vector request with vector object and configuration to Vald server via gRPC
		_, err := client.Insert(ctx, &payload.Insert_Request{
                        // vector object
			Vector: &payload.Object_Vector{
				Id:     ids[i],
				Vector: train[i],
			},
                        // configuration
			Config: &payload.Insert_Config{
				SkipStrictExistCheck: true,
			},
		})
		if err != nil {
			glg.Fatal(err)
		}
		if i%10 == 0 {
			glg.Infof("Inserted: %d", i+10)
		}
	}
	glg.Info("Finish Inserting dataset. \n\n")

        // wait for indexing to finish
	wt := time.Duration(indexingWaitSeconds) * time.Second
	glg.Infof("Wait %s for indexing to finish", wt)
	time.Sleep(wt)

	/**
	Gets exact nearest vectors, which is based on the value of `SearchConfig`, from the indexed tree based on the training data.
	In this example, Vald gets 5 approximate vectors each search vector.
	**/
	glg.Infof("Start searching %d times", testCount)
	for i, vec := range test[:testCount] {
                // Send linear search request with query(vector) to Vald server via gRPC
		res, err = client.LinearSearch(ctx, &payload.Search_Request{
                        // search query
			Vector: vec,
                        // serach config
			Config: &payload.Search_Config{
				Num:     Num,
				Radius:  Radius,
				Timeout: Timeout,
			},
		})
		if err != nil {
			glg.Fatal(err)
		}

                // convert result to buffer
		b, _ = json.MarshalIndent(res.GetResults(), "", " ")
		glg.Infof("%d - Linear Search Results : %s\n\n", i+1, string(b))
		time.Sleep(1 * time.Second)
	}
	glg.Infof("Finish searching %d times", testCount)

	// Delete indexed 10000 vectors from vald cluster.
	glg.Info("Start removing vector")
	for i := range ids[:insertCount] {
                // Send delete request with vector's id to Vald server via gRPC
		_, err := client.Remove(ctx, &payload.Remove_Request{
                        // the id of vector which will be deleted
			Id: &payload.Object_ID{
				Id: ids[i],
			},
		})
		if err != nil {
			glg.Fatal(err)
		}
		if i%10 == 0 {
			glg.Infof("Removed: %d", i+10)
		}
	}
	glg.Info("Finish removing vector")
}

/**
load function loads training and test vector from hdf file.
The size of ids is same to the number of training data.
Each id, which is an element of ids, will be set a random number.
This code is same as [https://github.com/vdaas/vald/blob/master/example/client/main.go](https://github.com/vdaas/vald/blob/master/example/client/main.go)
**/
func load(path string) (ids []string, train, test [][]float32, err error) {
	var f *hdf5.File
	f, err = hdf5.OpenFile(path, hdf5.F_ACC_RDONLY)
	if err != nil {
		return nil, nil, nil, err
	}
	defer f.Close()

	readFn := func(name string) ([][]float32, error) {
		d, err := f.OpenDataset(name)
		if err != nil {
			return nil, err
		}
		defer d.Close()

		sp := d.Space()
		defer sp.Close()

		dims, _, _ := sp.SimpleExtentDims()
		row, dim := int(dims[0]), int(dims[1])

		vec := make([]float32, sp.SimpleExtentNPoints())
		if err := d.Read(&vec); err != nil {
			return nil, err
		}

		vecs := make([][]float32, row)
		for i := 0; i < row; i++ {
			vecs[i] = make([]float32, dim)
			for j := 0; j < dim; j++ {
				vecs[i][j] = float32(vec[i*dim+j])
			}
		}

		return vecs, nil
	}

	train, err = readFn("train")
	if err != nil {
		return nil, nil, nil, err
	}

	test, err = readFn("test")
	if err != nil {
		return nil, nil, nil, err
	}

	ids = make([]string, 0, len(train))
	for i := 0; i < len(train); i++ {
		ids = append(ids, fuid.String())
	}

	return
}
```

## 差异结果

让我们检查搜索结果。条件如下:

*   数据集:时尚-mnist-784-欧几里德
*   插入索引数:10，000
*   搜索向量的数量:10
*   数字:5
*   半径:-1
*   ε:0.1(仅搜索 API 使用)
*   超时:100，000，000 纳秒

索引的数据很小，所以没有明显的区别。但是，一些搜索结果不同于使用相同搜索查询的线性搜索结果。

*   搜索结果(前 5 名)

```
[
 {
  "id": "esg96afc89l5uc5rkvlg",
  "distance": 1200.5337
 },
 {
  "id": "esg96afc89l5uc5rk6u0",
  "distance": 1268.7761
 },
 {
  "id": "esg96afc89l5uc5rgaug",
  "distance": 1325.6569
 },
 {
  "id": "esg96afc89l5uc5rkffg",
  "distance": 1337.0516
 },
 {
  "id": "esg96afc89l5uc5rjke0",
  "distance": 1344.2522
 }
]
```

*   线性搜索结果(前 5 名)

```
[
 {
  "id": "esg96afc89l5uk5rl44g",
  "distance": 1120.9625
 },
 {
  "id": "esg96afc89l5uc5rkvlg",
  "distance": 1200.5337
 },
 {
  "id": "esg96afc89l5uc5rk6u0",
  "distance": 1268.7761
 },
 {
  "id": "esg96afc89l5uc5rijig",
  "distance": 1302.7283
 },
 {
  "id": "esg96afc89l5uc5rgaug",
  "distance": 1325.6569
 }
]
```

如上所述，你可以看到线性搜索结果比搜索结果更接近距离矢量。根据这个结果，您可以调整搜索参数和 NGT 配置，使搜索结果接近线性搜索结果。

线性搜索的结果是精确的最近邻向量。然而，考虑到从十亿级数据中搜索最近的向量，线性搜索将花费更长的时间。我们建议对您的服务使用搜索，对优化配置使用线性搜索。

如果您有任何问题或要求，请随时联系我们🙂

[](https://join.slack.com/t/vald-community/shared_invite/zt-db2ky9o4-R_9p2sVp8xRwztVa8gfnPA) [## 松弛的

### 编辑描述

join.slack.com](https://join.slack.com/t/vald-community/shared_invite/zt-db2ky9o4-R_9p2sVp8xRwztVa8gfnPA) [](https://vald.vdaas.org/docs/contributing/contributing-guide/#contributing-issue) [## 投稿指南

### Vald 是一个开源项目。我们感谢您的帮助！我们使用 Github 问题来跟踪这个库中的问题…

vald.vdaas.org](https://vald.vdaas.org/docs/contributing/contributing-guide/#contributing-issue) 

下期见:)

相关帖子

[](https://vdaas-vald.medium.com/release-announcement-v1-4-0-5c65a48aaa25) [## 发布公告:1.4.0 版

### 我们将在本周发布 1.4.0 版。

vdaas-vald.medium.com](https://vdaas-vald.medium.com/release-announcement-v1-4-0-5c65a48aaa25) 

其他员额

[](https://medium.com/geekculture/vald-a-highly-scalable-distributed-fast-approximate-nearest-neighbour-dense-vector-search-engine-af1946a4a37) [## 瓦尔德。一个高度可扩展的分布式快速近似最近邻密集向量搜索引擎。

### Vald 简介:云原生向量搜索引擎

medium.com](https://medium.com/geekculture/vald-a-highly-scalable-distributed-fast-approximate-nearest-neighbour-dense-vector-search-engine-af1946a4a37) [](https://vdaas-vald.medium.com/a-new-world-created-by-similar-search-cases-where-vald-can-be-used-15c768e49bb) [## 相似搜索创造的新世界:可以使用 Vald 的案例。

### 你用 Vald 做什么？:案例和案例研究

vdaas-vald.medium.com](https://vdaas-vald.medium.com/a-new-world-created-by-similar-search-cases-where-vald-can-be-used-15c768e49bb) [](https://vdaas-vald.medium.com/a-super-easy-way-to-try-similarity-search-using-vald-88fd7e8b87e9) [## 使用 Vald 尝试相似性搜索的一个超级简单的方法

### 如何在 5 分钟内在 k3d 上部署 Vald

vdaas-vald.medium.com](https://vdaas-vald.medium.com/a-super-easy-way-to-try-similarity-search-using-vald-88fd7e8b87e9)