# 证书颁发机构这么可靠吗？

> 原文：<https://itnext.io/zijn-certificate-authorities-wel-zo-betrouwbaar-fe1318636932?source=collection_archive---------1----------------------->

![](img/0ba1007d1d92a1afdbe30277d5f7fbdc.png)

为了安全地使用互联网，政府和银行不断提醒我们使用安全连接，并在网站地址的绿色锁上注明。但这是否和录取通知一样安全？安全连接的一部分是证书颁发机构(CA)。这些公司提供数位凭证(以下称「认证」)以确保所使用网站的真实性。本文介绍了这些 CA 的组织方式，以使用户了解潜在的安全问题。

## 证书颁发机构

如上所述，CA 向网站所有者颁发证书，以保证网站的真实性。如此一来，使用者就知道自己不会浏览以[www . link . nl](http://www.linkit.nl)开头的 malify 网站。除了验证之外，CA 还是连接加密(SSL/TLS)的一部分。CA 是一个博物馆。当我们进入博物馆的时候我们假设这些画是真的并且是由保守派控制的。

## 不确定性

但是，在 2011 年 DigiNotar 和 Comodo 的黑客攻击中，CA 的可靠性并不总是确定的。Google、Mozilla 和 Microsoft 提供了欺诈性认证，允许网络罪犯拦截加密的数据(中间人攻击)。此外，凭证中的 CA 还是常用的 SHA-1 hash 函数，也就是一种让资料无法读取的方法，[因此不再被视为](https://threatpost.com/final-report-diginotar-hack-shows-total-compromise-ca-servers-103112/77170/)安全。

CA 通常都是由政府发起的一些非营利组织(gei)组成的商业企业，有时来自有争议的[互联网使用道德规范的国家(T5)。本组织远非透明，不幸的是，没有受到监管。认证基础建设安全性不需要符合任何标准或规范。](http://tweakers.net/nieuws/102273/mozilla-vertrouwt-certificaten-chinese-certificaatautoriteit-cnnic-niet-meer.html)

此外，过期、黑客或吊销的证书也是一个很大的安全风险，这些证书已过期且仍被浏览器接受为可信任。此类认证的注册和资格不是集中管理的。

## 备用轮胎

IT 安全部门发起了各种旨在解决安全问题的应急磁带计划:

*   扩展验证(EV):允许您请求符合特定身份验证要求的证书。不幸的是，这并不能说明 CA 环境的安全性。
*   在线证书状态协议(ocsp):ocsp 验证证书是否仍然有效且未被吊销。抱歉，并非所有应用程序或浏览器都支持 OCSP。另外，OCSP 易受 [DNS 欺骗](https://en.wikipedia.org/wiki/DNS_hijacking)的攻击。
*   认证透明度(CT)日志:Google 和 Symantec 等厂商发布包含 CA 或不再信任的认证的日志。这类日志的问题是，它们经常落后于实际情况。
*   认证 pin:某些厂商要求仅使用预先定义的 CA。Google 就是最好的例子，只有由 CA “Google Internet Authority G2”授予 google.com 和 gmail.com 的 Google 自己的证书才被接受。
*   CA/Browser 论坛:由 CA 供应商和浏览器制造商组成的联盟，旨在提高证书颁发的质量和可靠性。

## 你的建议

对于 CA，建议将证书从常规服务器移动到硬件安全模块(HSM)，以显着降低成功攻击的风险。HSM 是专为保护资料而设计的实体装置。

另一个建议是仅使用支持证书锁定、透明日志和 OCSP 的浏览器。此类浏览器的示例包括 Firefox、Chrome 或相关选项，如 Epic 隐私浏览器和 Comodo Dragon/ICE Dragon。在这些浏览器中正确配置安全选项非常重要。

别銡拟梓妎桴萸腔证书衄恀栀ㄛ寀郔疑祥猁硐岆绮谨秏洘ㄛ奥茼脤艘峈妦系祥诿忳蚬证书﹝。

## 结论是

您可能会发现，使用授权凭证并不安全。事实上，使用安全连接(如 SSL 和 HTTPS)仍然比使用标准 HTTP 连接安全得多。

但是，如果制定了一项必须遵守的全球条约，这将极大地提高可靠性。强制使用经过验证的安全框架和标准。为此的一个良好起点是 CA/Browser 论坛制定的“公开可信证书的查询和管理的基准要求”[。必须定期和独立地审查合规性。](https://cabforum.org/baseline-requirements-documents/)