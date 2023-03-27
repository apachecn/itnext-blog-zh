# 在 Python 中监控您的华硕路由器

> 原文：<https://itnext.io/monitor-your-asus-router-in-python-171693465fc1?source=collection_archive---------4----------------------->

使用华硕路由器的可用但未记录的 web API 来监控使用情况和性能。

![](img/f703c7ba8110eab6e46b53f9cb4bea24.png)

拉斯·金勒拍摄的照片(图片来源: [Unsplash](https://unsplash.com/@larskienle)

我家网络的心脏是一台华硕 RT-AC68U 路由器。要监控它的运行和健康状况，可以使用华硕路由器 App。该应用程序提供了关于连接性、连接设备等的良好概述。它甚至有一个漂亮的图表显示实时交通状况。

那么，我们如何在自己的应用程序和脚本中使用这些信息呢？没有文档化的 web 或 REST 接口可用，但当应用程序可以获取数据时，我们不也可以吗？我们的第一步是在 Android 设备上使用数据包嗅探器，看看我们是否能看到发生了什么。这比预期的要容易得多…

当应用启动时，端点 */login.cgi* 被调用。在这次登录之后，端点 */appGet* 被使用不同的有效负载调用多次。有效载荷似乎标识了可以调用的消息。请注意，我们只能这样做，因为路由器不使用 SSL，它们是普通的、老式的 HTTP 请求。安全方面的问题；您网络上的任何人都可以从您的应用程序中看到登录凭据，并使用它们来访问路由器。

## 执行登录

但是在我们变得过于热情之前，让我们首先尝试执行一个成功的登录调用。检查捕获的包，我们看到

```
REQUEST
=======
POST /login.cgi HTTP/1.1
Host: <ROUTER IP>
Accept: */*
Authorization: Basic <AUTHKEY>
User-Agent: asusrouter-Android-DUTUtil-1.0.0.201
Content-Length: 44
Content-Type: application/x-www-form-urlencodedPayload:
login_authorization=<AUTHKEY>REPLY
=====
HTTP/1.0 200 OK
Model_Name: RT-AC66U_B1
Content-Type: application/json:charset=UTF-8
Set-cookie: asus_token=<TOKEN>; HttpOnly;
....{ 
  "asus_token": "<TOKEN>"
}
```

似乎我们可以使用 */login.cgi* 端点从路由器获得一个令牌。有效负载让我想起了在其他网站上看到的 Base64 编码的用户名/密码。这是一种非常常见的认证形式。

因此，让我们尝试从路由器获取一个令牌:

我们尝试在路由器的 ip 地址上调用 */login.cgi* 端点。HTTP 请求的有效负载包含登录授权，如数据包捕获中所示。我们需要将账户信息<用户名> : <密码>转换成 Base64 编码字符串。首先，我们用*str . encode(‘ascii’)将字符串编码成一个字节数组。*该字节数组使用 *base64.b64encode()* 进行 Base64 编码。为了创建这个编码字节数组的字符串表示形式，该数组被编码为 ascii 字符串。这个 ascii 字符串(名为 *login* )被添加到登录请求的有效负载中。用户名和密码与登录路由器 web 界面时相同。

第一次尝试没有使用标题，导致了一个 HTML 错误页面:

```
<HTML><HEAD><script>top.location.href='/Main_Login.asp';</script>
</HEAD></HTML>
```

当我们查看捕获的包时，我们看到一个非常具体的用户代理，指定了某种 Android 客户端。添加此用户代理会产生上面的代码并成功执行，从而从路由器产生一个令牌供进一步使用:

```
{ "asus_token":"<token>" }
```

## 从路由器获取信息

现在我们有了一个令牌，让我们看看是否可以从路由器获得任何信息。网络嗅探器中的第一个请求显示了一个 *uptime()* 调用。该呼叫作为*挂钩*被发送到端点 */appGet.cgi* 。

```
REQUEST
=======
POST /appGet.cgi HTTP/1.1
Host: 192.168.2.1
Accept: */*
user-Agent: asusrouter-Android-DUTUtil-1.0.0.245
cookie: asus_token=wwyCQ78dUYSXY7VSfjdS4fqwuEy8npj
Content-Length: 220
Content-Type: application/x-www-form-urlencodedPayload:
hook=uptime();REPLY
=====
Server: httpd/2.0
Cache-Control: no-store, no-cache, no-store, must-revalidate
Pragma: no-cache, no-cache
Model_Name: RT-AC66U_B1
Date: Sat, 24 Jul 2021 15:27:26 GMT
Expires: 0
Content-Type: application/json;charset=UTF-8
Connection: close{
  "uptime":Sat, 24 Jul 2021 17:27:26 +0200(558288 secs since boot)
}
```

让我们试着调用这个函数:

我们使用类似于" *hook=uptime()* "的有效负载调用 endpoint /appGet.cgi，同时使用与第一次调用相同的用户代理并添加 *asus_token* 作为 cookie，我们获得以下结果:

```
{
"uptime":Sat, 24 Jul 2021 17:27:26 +0200(558288 secs since boot)
}
```

厉害！我们现在能够从我们的华硕路由器检索信息！一些测试表明，使用与调用端点完全相同的头进行登录至关重要。检查包嗅探器，我们确定以下方法:

```
uptime()
memory_usage()
cpu_usage()
get_clientlist()
netdev(appobj)
wanlink()
nvram_get(time_zone|time_zone_dst|time_zone_x|time_zone_dstoff|
          time_zone|ntp_server0|acs_dfs|productid|apps_sq|
          lan_hwaddr|lan_ipaddr|lan_proto|x_Setting|label_mac|
          lan_netmask|lan_gateway|http_enable|https_lanport
          wl0_country_code|wl1_country_code|wps_enable|wps_sta_pin
          enable_samba|st_samba_mode|enable_ftp|st_ftp_mode)
dhcpLeaseMacList() 
```

很可能会有更多的钩子可用，更多的选项，尤其是 *nvram_get* 钩子。

## 网络带宽

利用识别出的钩子，我们已经构建了一些很好的函数，例如，一个带宽计算器，其数据来自钩子 *netdev(appobj):*

```
{
  'netdev': {
    'INTERNET_rx': '0x3f4b8f6e',
    'INTERNET_tx': '0xc06bba74',
    'BRIDGE_rx': '0x253239bec',
    'BRIDGE_tx': '0x1add76312',
    'WIRED_rx': '0xa1097d399',
    'WIRED_tx': '0x191085a63',
    'WIRELESS0_rx': '0x51340e36',
    'WIRELESS0_tx': '0x110b00a5',
    'WIRELESS1_rx': '0x3e3e76cc',
    'WIRELESS1_tx': '0x66f13802'
  }
}
```

这些数字表示通过 WAN 连接、网桥连接、有线连接和无线连接发送和接收的数据量。多次调用该方法显示，数字只会增加，因此它似乎是一个连续的摘要。这些数字每两秒钟更新一次。

一些计算表明，十六进制数代表总字节数。将差值乘以 8 得到位，除以(1024*1024)得到兆位，最后除以 2 得到每秒，得到的数字与应用程序给出的带宽使用量相同。

因此，计算 WAN 网络流量的代码是:

```
TX Mbit/s : 0.09597015380859375
RX Mbit/s : 1.3212623596191406
```

## 创建 RouterInfo 类

创建一个助手类 **RouterInfo** ，以方便使用路由器功能。初始化之后，它提供了对路由器细节的访问，包括一些简化用户工作的附加方法。

在创建时，创建类并从路由器获得令牌。路由器的 IP 和该令牌被存储以供其他用户使用。创建一个私有的 *__get()* 方法，该方法包含调用路由器钩子函数的通用代码。该类的所有方法都使用它。完整的类可以在 Github 上找到:[LME ulen/AsusRouterMonitor](https://github.com/lmeulen/AsusRouterMonitor)。

一个示例方法是 *get_uptime()* 方法。实现首先通过 *__get()* 方法调用钩子 *uptime()* 。结果有点奇怪:

```
{\n"uptime":Sat, 24 Jul 2021 17:27:26 +0200(558288 secs since boot)\n}
```

在将它返回给调用者之前，以秒为单位的最后启动时间和正常运行时间被分成两个不同的字段。上次启动时间是“:”和“(”之间的字符串，而正常运行时间是“(”和第一个空格之间的部分。带有这两个字段的 JSON 被创建并返回给调用者。

在编写**的时候，RouterInfo** 类有以下方法:

```
get_uptime           - Uptime and last time of boot
get_uptime_secs      - Uptime
get_memory_usage     - Memory usage statistics
get_cpu_usage        - CPU usage statistics
get_settings         - get set of most important router settings
get_clients_fullinfo - All info of all connected clients
get_clients_info     - Get most important info on all clients
get_client_info(cid) - Get info of specified client (MAC address)
get_traffic_total    - Total network usage since last boot
get_traffic          - Current network usage and total usage
get_status_wan       - Get WAN status info
is_wan_online        - WAN connected True/False
get_lan_ip_adress    - Get router IP address for LAN
get_lan_netmask      - Get network mask for LAN
get_lan_gateway      - Get gateway address for LAN
get_dhcp_list        - List of DHCP leases given out
get_online_clients   - Get list of online clients (MAC address)
get_clients_info     - Get most important info on all clients
get_client_info(cid) - Get info of specified client (MAC address)
```

随着时间的推移，这个类将会被其他钩子和用户友好的函数所扩展。关注[lmeulen/AsusRouterMonito](https://github.com/lmeulen/AsusRouterMonitor)r 保持最新。

## 可能的使用案例

那么我们能利用这些信息做什么呢？浮现在脑海中的一些想法是:

*   **网络监控** —定期从路由器获取最重要的统计数据，将其存储在 InfluxDB 中，并创建一个 Grafana 仪表板来监控您的网络信息。
*   **检查设备状态** —检查特定设备是否在线，并将其用于家庭自动化。例如，当智能手机在线时，一个人在家。或者在重要设备离线时收到通知
*   **监控您的 ISP** —跟踪您的 ISP 提供的 WAN 连接的可靠性
*   **网络分析** —你的网络感觉拥挤吗？找到最大的数据用户并验证网络和 WAN 容量。
*   **监控连接的设备** —监控所有连接的设备，并在未知设备连接时发出警报。

## 最后的话

我希望你喜欢这篇文章。要获得灵感，请查看我的其他文章:

*   [Python 中字符串的并排比较](https://towardsdatascience.com/side-by-side-comparison-of-strings-in-python-b9491)
*   [用 Python 删除文本中的个人信息](https://towardsdatascience.com/remove-personal-information-from-text-with-python-232cb69cf074)
*   [使用 Python 的并行 web 请求](https://towardsdatascience.com/parallel-web-requests-in-python-4d30cc7b8989)
*   所有公共交通工具都通向乌得勒支，而不是罗马
*   [使用 OTP 和 QGIS 可视化行程时间](https://towardsdatascience.com/visualization-of-travel-times-with-otp-and-qgis-3947d3698042)

如果你喜欢这个故事，请点击关注按钮！

*免责声明:本文包含的观点和意见仅归作者所有。*