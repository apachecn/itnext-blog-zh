# 使用证书的有用命令

> 原文：<https://itnext.io/useful-commands-to-work-with-certificates-402dac84bb75?source=collection_archive---------6----------------------->

![](img/80072d1d09a44558a1143196eace70a9.png)

我们好像终于切换到 *https* 了。这意味着我们无时无刻不在使用证书，却常常不太注意它们。你如何*看到*一个证书的内容？将其转换为不同的格式？有许多有用的命令很难记住，除非你经常使用它们(或者，方便的话，找一个关于这个主题的帖子)。我想分享其中的一些，重点是从命令行做事情。也许你已经看过[这篇关于 OpenSSL 命令的精彩文章](https://www.sslshopper.com/article-most-common-openssl-commands.html)。我希望我能提供更多的背景。

回顾一下，在使用 *https* 时，我们通常使用证书来验证服务器的身份。但是，如果您使用 mTLS，客户端也需要提供证书。X.509 格式是我们在野外常见的格式。

我们将使用一些方便的工具来完成这项工作，这些工具可以与 [brew](https://brew.sh/) 一起安装，例如:

*   [mkcert](https://github.com/FiloSottile/mkcert)
*   [openssl](https://www.openssl.org/)
*   base64

# 创建测试证书

要开始，我们需要一个证书。如果您拥有一个域，您将希望使用 [LetsEncrypt](https://letsencrypt.org/) 以编程方式生成一个域。出于演示的目的，我将使用`mkcert`。

```
mkcert -install 
mkcert example.com app1.example.com app2.example.com
```

结果存储在`example.com+2-key.pem`和`example.com+2.pem`下

这个证书是为三个域颁发的，使用所谓的 [SAN](https://en.wikipedia.org/wiki/Subject_Alternative_Name) ，它扩展了[通用名称](https://support.dnsimple.com/articles/what-is-common-name/)，这样您就可以指定多个域。CN 限于 64 个字符，这对于有很多子域的内部证书来说可能是个问题(不要问我是怎么学会这个的！).

即使它是自签名的，我们仍然可以验证它:

```
openssl verify -CAfile ~/Library/Application\ Support/mkcert/rootCA.pem example.com+2.pem
```

# 阅读证书

您可以使用以下方法读取证书:

```
openssl x509 -in example.com+2.pem -text -noout
```

它显示了这样的内容:

```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            d7:98:98:19:27:dd:52:d0:1f:1f:07:b5:ea:85:54:eb
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: O=mkcert development CA, OU=me@my-computer (My Name), CN=mkcert me@my-computer (My Name)
        Validity
            Not Before: Jun  1 00:00:00 2019 GMT
            Not After : Apr  6 20:21:32 2030 GMT
        Subject: O=mkcert development certificate, OU=me@my-computer (My Name)
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:b1:b7:1b:14:f9:61:79:90:be:21:bc:91:12:78: (...)
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage: 
                TLS Web Server Authentication
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Authority Key Identifier: 
                keyid:34:0F:21:0C:F8:7D:D5:DF:E4:15:B6:15:B9:94:C9:2B:F3:E0:24:AD

            X509v3 Subject Alternative Name: 
                DNS:example.com, DNS:app1.example.com, DNS:app2.example.com
    Signature Algorithm: sha256WithRSAEncryption
         a8:52:3b:19:dc:74:4f:e3:6b:9c:cd:0c:59:c3:fe:e8:0b:2d: (...)
```

我们可以在那里看到所有相关的信息。 *X509v3 主题别名*往往是最有趣的部分，因为您可以看到证书覆盖的域。

# 编码证书

通常，证书以 Base64 ASCII 中的 PEM 格式存储。你通过开头的`---- BEGIN CERTIFICATE----`线认出他们。例如，这就是`mkcert`给我们的。

有时，PEM 证书本身以 Base64 编码传输。假设我们通过网络接收证书。我们把它存放在`/tmp/b64-cert`下，它看起来是这样的:

```
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUVlakNDQXVLZ0F3SUJBZ0lSQU5lWW1Ca24zVkxRSHg4SHRlcUZWT3N3RFFZSktvWklodmNOQVFFTEJRQXcKZ1lNeEhqQWNCZ05WQkFvVEZXMXJZMlZ5ZENCa1pYWmxiRzl3YldWdWRDQkRRVEVzTUNvR0ExVUVDd3dqYldabApjbTVoYm1SbGVrQm9hWE52YTJFZ0tFMWhjbWx2SUVabGNtNWhibVJsZWlreE16QXhCZ05WQkFNTUttMXJZMlZ5CmRDQnRabVZ5Ym1GdVpHVjZRR2hwYzI5cllTQW9UV0Z5YVc4Z1JtVnlibUZ1WkdWNktUQWVGdzB4T1RBMk1ERXcKTURBd01EQmFGdzB6TURBME1EWXlNREl4TXpKYU1GY3hKekFsQmdOVkJBb1RIbTFyWTJWeWRDQmtaWFpsYkc5dwpiV1Z1ZENCalpYSjBhV1pwWTJGMFpURXNNQ29HQTFVRUN3d2piV1psY201aGJtUmxla0JvYVhOdmEyRWdLRTFoCmNtbHZJRVpsY201aGJtUmxlaWt3Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLQW9JQkFRQ3gKdHhzVStXRjVrTDRodkpFU2VMNENTL3JET3RxbGZGN0syN1g3NERuQ0ExeHpDMlkwTkJxa1hPVWFka2VRZ1MrSgo4NVFZQlYrcWFFRGhZNVNMZm1MTGxkMmFWQnIvSG9MYVFuRDc4VXJuVHNHM1JJb2dEYlI0R25CQnJUc0ltQWk3CnRNeU1kVVZSc21EM1RYcmUxdFM4dmowL3RQNzE1UEQvRTd1Tk94M2VyYjUrQmtFY0JodElnbVU4ZUVybnpLMnEKNkV0N0tsUERwQm5mL2pySEM0ZU1URFBiRDhqeGZBRmhvZ2R0YUgrTGRxeDZ6Uk5mN2pTZXNXMTJNMW5JenpsTwpLa0IxaHBhOEZqc3Y3Ry9ONytFeHFzQzMrQjdaZHB5UWl5Zm8ydU1KL1lJQWJMN29sbnV5THF5UkZPNWNaNnp3Cjg5MG5lV1gweE5hWmE5bWo3RmNwQWdNQkFBR2pnWk13Z1pBd0RnWURWUjBQQVFIL0JBUURBZ1dnTUJNR0ExVWQKSlFRTU1Bb0dDQ3NHQVFVRkJ3TUJNQXdHQTFVZEV3RUIvd1FDTUFBd0h3WURWUjBqQkJnd0ZvQVVOQThoRFBoOQoxZC9rRmJZVnVaVEpLL1BnSkswd09nWURWUjBSQkRNd01ZSUxaWGhoYlhCc1pTNWpiMjJDRUdGd2NERXVaWGhoCmJYQnNaUzVqYjIyQ0VHRndjREl1WlhoaGJYQnNaUzVqYjIwd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dHQkFLaFMKT3huY2RFL2phNXpOREZuRC91Z0xMYlNlV0NsVjFMSnFNU3NLbE16VXZ4aVlmKzdzeDZRUmxZcGtYeWpndVFUNwppRWZMWUhQZG8zeTJMNmFIaW9tbU1FUjNmWjAyMjJWUWJhV2hjUkJmUDgzdFhnQmdqN0pUZjArUUxjVERXZG9YCnd2KzZQNHNuNmVqV2xWc3drZTBQcjhLMm9DM1NyazVibjJFVVpQSU1oQnNMZFdmczBZbkZhRTFCY2ZlWGNVV0QKU3QzckRZcjFlSW1mQ1FTQ1BkU0ZwTU5HWS81dkdtWm5lVTNxbXlhS2hLNWZRREhkWVpQYlQ5NFE4ZDBVVXRaMgpEeWdxQmFmTG5oMWxmVHRRNi9NUjdIQlNWZCtWajcwRnFCajJlaEJlTXZhUDUrQlRCTEFCbWdoK0VMVHp6VTRUCmdHMHZuSzF2OUhvd1FDaVdadDkxZDRtdFRHbldJQVVHWGpKcDBjMmVHbU5NOUN4eVM0QTJNRXRJcngwZy9ESWQKNU9rMXZodnVJOTV4cWVOZ1E4UDBrOEF0VWlYdVp4Uk53M1FubHdLRzloMDhmVGJRc04rZEh1T2lhRWNPcWY4WgoxcStxakxQMlVsbmZYaGwyN25aUVgwZ3F5ZnROUldRV3pvc0V6cGh0RVpwdGJrVzZPUExhby9DNHhVRU5lQT09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
```

是的，那是一条很长的线。`openssl`不能开箱读取，所以我们必须先解码。此外，可能是因为一旦解码，行太长，这使`openssl`感到困惑。我们将解码并折叠该字符串，以便可以再次读取证书:

```
base64 -d /tmp/b64-cert | fold -64 | openssl x509 -text -noout
```

# 二进制证书

有时候你的证书是用二进制格式的 [DER](https://wiki.openssl.org/index.php/DER) 编码的。`JVM`世界真的很喜欢用这种方式运证。您会注意到，如果您检查这样的证书，它将主要是一堆特殊字符和胡言乱语。幸运的是，`openssl`可以在两个方向上从 DER 转换到 PEM。

```
openssl x509 -in example.com+2.pem -outform DER -out example.com+2.der
openssl x509 -inform DER -in example.com+2.der -text -noout
```

# 以 base64url 编码的二进制证书

如果您有一个 DER 格式的证书，并且已经用 [base64url](https://base64.guru/standards/base64url) 进行了编码，该怎么办？你的第一个想法可能是，“这是一个奇怪的特定组合”。但这真的会发生。如果您正在实现自己的 ACME 服务器，请求证书是[协议的一部分](https://tools.ietf.org/html/rfc8555#section-7.1.5)。我最近花了相当多的时间试图弄明白这一点。

没有现成的工具来解码 Base64 URL，但是您可以使用类似于[这个](https://raw.githubusercontent.com/Moodstocks/moodstocks-api-clients/master/bash/base64url.sh)的脚本。您可以通过结合脚本和前面的调用来阅读这个神秘的证书:

```
./base64url.sh decode `cat /tmp/b64url-cert | tr -d '\n'` | openssl x509 -inform DER -text -noout
```

你可能永远也不会遇到这种情况，但是如果你遇到了，这将会节省你大量的时间。

# 检查远程服务器上的证书

如果您没有证书，但想从远程服务器获取它，该怎么办？openssl 能够胜任这项任务。

```
openssl s_client -connect google.com:443 -servername google.com | openssl x509 -text -noout
```

使用`s_client`命令，我们请求特定端口(通常是 443)上的主机提供的证书。在这种情况下，我们正在检查来自*google.com*的证书。我们可以将输出通过管道传输到读取证书的标准命令中。

你可能想知道`-servername`选项。那就是与 [SNI](https://en.wikipedia.org/wiki/Server_Name_Indication) 有关。如果目标为同一 IP 地址下的多个域提供服务，您可能希望选择与某个域相关联的 IP 地址。我最近在测试一个提供自签名证书的 Kubernetes [nginx ingress](https://github.com/kubernetes/ingress-nginx) 时需要这个选项，除非我指定了这个选项。

# 我们只是触及了表面

这篇文章的重点是阅读现有的证书，但是你可以用`openssl`做更多的事情，包括写证书、提取密钥等等。更不用说[证书签名请求](https://en.wikipedia.org/wiki/Certificate_signing_request)。然而，阅读是熟悉证书的好地方。

*原载于 2020 年 4 月 8 日 https://hceris.com**的* [*。*](https://hceris.com/useful-commands-to-work-with-certificates/)