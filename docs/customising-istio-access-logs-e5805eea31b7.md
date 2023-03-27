# 定制 istio 访问日志

> 原文：<https://itnext.io/customising-istio-access-logs-e5805eea31b7?source=collection_archive---------4----------------------->

**注**:以下文章反映了在使用`istio` `1.7.4`运行于`k8s`版本`1.17`的 GKE 之上的环境中进行的调查

![](img/ad30f63f81994d4e78cbfdc5aab4b89e.png)

# 需要定制 istio 访问日志

在 [Workable](http://www.workable.com) 中，我们使用 GKE 作为我们的核心基础设施，并通过以下方式处理传入的请求:

a) CloudFlare 代理我们的大部分服务
b) CloudDNS 在`k8s`之上为我们的负载平衡器创建 DNS 条目
c) `[istio](https://istio.io)`作为我们的服务网格，为通过`istio-ingressgateway`进入我们的`k8s`集群的流量创建入口端点

在我们最近必须执行的一些调查过程中，我们需要跟踪来自 CloudFlare(并由 cloud flare 代理)的请求/将这些请求关联到我们的`k8s`基础架构。

为了支持这个过程，CloudFlare 使用了一个名为`cfray`的自定义[头](https://support.cloudflare.com/hc/en-us/articles/203118044-Gathering-information-for-troubleshooting-sites#h_f7a7396f-ec41-4c52-abf5-a110cadaca7c)，它被注入到一个传入的请求中，并正式建议记录这个头以支持相关的故障排除。所以在这种背景下我们反对解决以下问题:

虽然`cfray`在 CloudFlare 日志中可用，但对于`istio-ingressgateway`却不是这样。因此，我们必须找到一种方法让`istio`记录特定的头，以便我们能够跟踪从 CloudFlare 进入我们的`k8s`集群的传入请求。

# Istio 测井

彻底掌握`istio`的日志行为有点棘手。

我们首先通过实验验证它在报头传播方面的表现；事实证明，它确实传播了所有的标题(这是通过让我们的一个应用程序在标题从其`istio-proxy`边车到达时打印其所有标题来测试的，并且`cfray`在那里),默认情况下它不记录它们。

然后我们转向`istio`的官方文档，并开始我们的主要调查过程以实现`cfray`记录。

与主题相关的官方`istio` [文档](https://istio.io/v1.7/docs/tasks/observability/logs/access-log/)，只需提及如何打开或关闭访问日志记录以及选择日志记录格式(文本 vs `json`)即可，用户可以自定义`IstioOperator`的`meshConfig.accessLogFormat`字段中记录的字段，我们通过自定义配置安装这些字段。

文件还指出:

> 最简单的`istio`日志记录是特使访问日志记录。Envoy 代理将访问信息打印到标准输出中。然后可以通过`kubectl` logs 命令打印出 Envoy 容器的标准输出。

这可能导致假设如果没有应用额外的设置，`istio`将使用 envoy 的默认访问日志格式；根据`envoy’` s [文档](https://www.envoyproxy.io/docs/envoy/latest/configuration/observability/access_log/usage#default-format-string)，如下:

```
 [%START_TIME%] “%REQ(:METHOD)% %REQ(X-ENVOY-ORIGINAL-PATH?:PATH)% %PROTOCOL%” %RESPONSE_CODE% %RESPONSE_FLAGS% %BYTES_RECEIVED% %BYTES_SENT% %DURATION% %RESP(X-ENVOY-UPSTREAM-SERVICE-TIME)% “%REQ(X-FORWARDED-FOR)%” “%REQ(USER-AGENT)%” “%REQ(X-REQUEST-ID)%” “%REQ(:AUTHORITY)%” “%UPSTREAM_HOST%”\n
```

然而，这与我们在集群中观察到的情况相差甚远，因为我们的`istio-proxy` 日志如下:

```
 istio-ingressgateway-4493400kkdhjrh-b45h1 istio-proxy {
 “response_flags”: “NR”,
 “requested_server_name”: “service-1.environment-name-1.stg-gke.ourdomain.net”,
 “bytes_received”: “0”,
 “istio_policy_status”: “-”,
 “route_name”: “-”,
 “user_agent”: “Mozilla/5.0 (compatible; Cloudflare-Traffic-Manager/1.0; +[https://www.cloudflare.com/traffic-manager/](https://www.cloudflare.com/traffic-manager/); pool-id: 393949487478575)”,
 “start_time”: “2021–01–27T15:02:15.128Z”,
 “method”: “GET”,
 “request_id”: “123456-cff3–9cfd-1234–494048938722”,
 “upstream_host”: “-”,
 “x_forwarded_for”: “10.16.32.1”,
 “bytes_sent”: “0”,
 “upstream_cluster”: “-”,
 “downstream_remote_address”: “10.16.32.1:1763”,
 “path”: “/health”,
 “authority”: “service-1.environment-name-1.stg-gke.ourdomain.net”,
 “protocol”: “HTTP/1.1”,
 “upstream_service_time”: “-”,
 “upstream_local_address”: “-”,
 “duration”: “0”,
 “downstream_local_address”: “10.16.32.27:8443”,
 “upstream_transport_failure_reason”: “-”,
 “response_code”: “404”
}
```

其中`service-1.environment-name-1.stg-gke.ourdomain.net`是 GCP 的 CloudDNS 公开的 url。

日志中的`json`格式是我们设置的，因为在我们的自定义`IstioOperator`配置中我们设置了:

```
meshConfig:
  accessLogFile: “/dev/stdout”
  accessLogEncoding: “JSON”
```

**更新**:原来在最新的`istio`版本中(`1.9`)文档被更新以反映实际的测井格式；不幸的是，我们在调查过程中无法获得这一信息。

鉴于上述情况，我们必须解决以下问题:

a)这个默认的日志格式设置
在哪里 b)在`meshConfig.accessLogFormat`中增加`json`字段会有什么影响？这些字段会被追加到默认的访问日志中，还是会完全覆盖它？

**查找默认访问日志格式的配置**

默认访问格式设置的位置似乎没有记录(或者至少很难找到——你可以顺便在`istio`的 GH repo [这里](https://github.com/istio/istio/issues/30242)跟随相关讨论)。但是，在任何时间点(假设默认的访问日志格式没有被覆盖)，您都可以找到执行到`istio-proxy` pod 并解析其配置转储所使用的实际默认日志格式，如

```
kubectl exec -it istio-ingressgateway-4493400kkdhjrh-b45h1 -c istio-proxy -n istio-system — curl -s localhost:15000/config_dump | grep format -A 10
```

# 修改日志格式并捕获`cfray`头

发现我们的`istio`配置使用的实际(默认)日志格式后，下一步是我们提供适当的修改(设置为`IstioOperator`的`meshConfig.accessLogForma` t 字段的值)，以便我们能够记录 CloudFlare 的`cfray`头。

第一步是定义如何在`IstioOperator`清单中适当地格式化`meshConfig.accessLogFormat`；这也有点棘手，但由于在另一期 GH [中推出的一个讨论中的评论，我们使用了以下配置(实际上有效):](https://github.com/istio/istio/issues/24318#issuecomment-688672926)

```
accessLogFormat: |               
   {               
    "accessLogFormat": "{                                         
                        \"cfray\": \"%REQ(CF-RAY)%\",                                                   
                        \"authority\": \"%REQ(:AUTHORITY)%\",                                     
                        \"start_time\": \"%START_TIME%\",                                    
                        \"bytes_received\": \"%BYTES_RECEIVED%\",                                    
                        \"bytes_sent\": \"%BYTES_SENT%\",                                           
                        \"downstream_local_address\": \"%DOWNSTREAM_LOCAL_ADDRESS%\",                                    
                        \"downstream_remote_address\": \"%DOWNSTREAM_REMOTE_ADDRESS%\",                                    
                        \"duration\": \"%DURATION%\",                                    \"istio_policy_status\": \"%DYNAMIC_METADATA(istio.mixer:status)%\",                                    
                        \"method\": \"%REQ(:METHOD)%\",                                    
                        \"path\": \"%REQ(X-ENVOY-ORIGINAL-PATH?:PATH)%\",         
                        \"protocol\": \"%PROTOCOL%\",                                       
                        \"request_id\": \"%REQ(X-REQUEST-ID)%\",                                    
                        \"requested_server_name\": \"%REQUESTED_SERVER_NAME%\",                                    
                        \"response_code\": \"%RESPONSE_CODE%\",                                    
                        \"response_flags\": \"%RESPONSE_FLAGS%\",                                    
                        \"route_name\": \"%ROUTE_NAME%\",                                    
                        \"start_time\": \"%START_TIME%\",                                    
                        \"upstream_cluster\": \"%UPSTREAM_CLUSTER%\",                                    
                        \"upstream_host\": \"%UPSTREAM_HOST%\",                                    
                        \"upstream_local_address\": \"%UPSTREAM_LOCAL_ADDRESS%\",                                    
                        \"upstream_service_time\": \"%RESP(X-ENVOY-UPSTREAM-SERVICE-TIME)%\",                                    
                        \"upstream_transport_failure_reason\": \"%UPSTREAM_TRANSPORT_FAILURE_REASON%\",                                    
                        \"user_agent\": \"%REQ(USER-AGENT)%\",                                    
                        \"x_forwarded_for\": \"%REQ(X-FORWARDED-FOR)%\"                                  
                       }"               
}
```

不幸的是，这确实有点乏味，因为它不是基于追加的，也就是说，无论你在那个部分下放置什么，最终都是完整的访问日志，因此格式的所有字段都必须重写。

至于捕获任何自定义标题，这遵循以下模式:

```
“namethatwillappearinistiolog”:”%REQ(CUSTOM-HEADER-TO-BE-LOGGED)%
```

这里是一个样本日志，应用了定制的`IstioOperator`

```
 istio-ingressgateway-4493400kkdhjrh-b45h1 stio-proxy {
 “cfray”: “38djr9lk3091100309091x”,
 “response_flags”: “NR”,
 “requested_server_name”: “service-1.environment-name-1.stg-gke.ourdomain.net”,
 “bytes_received”: “0”,
 “istio_policy_status”: “-”,
 “route_name”: “-”,
 “user_agent”: “Mozilla/5.0 (compatible; Cloudflare-Traffic-Manager/1.0; +[https://www.cloudflare.com/traffic-manager/](https://www.cloudflare.com/traffic-manager/); pool-id: 2da0260c20e19b2e)”,
 “start_time”: “2021–01–27T15:02:15.128Z”,
 “method”: “GET”,
 “request_id”: “123456-cvt5–9cfd-8289–4961432c5feab”,
 “upstream_host”: “-”,
 “x_forwarded_for”: “10.16.32.1”,
 “bytes_sent”: “0”,
 “upstream_cluster”: “-”,
 “downstream_remote_address”: “10.16.32.1:1763”,
 “path”: “/health”,
 “authority”: “service-1.environment-name-1.stg-gke.ourdomain.net”,
 “protocol”: “HTTP/1.1”,
 “upstream_service_time”: “-”,
 “upstream_local_address”: “-”,
 “duration”: “0”,
 “downstream_local_address”: “10.16.32.27:8443”,
 “upstream_transport_failure_reason”: “-”,
 “response_code”: “404”
}
```