# 使用 Inverno 框架的 JSON 对象签名和加密(JWS、JWE、JWT)

> 原文：<https://itnext.io/json-object-signature-and-encryption-jws-jwe-jwt-with-inverno-framework-5e6900091504?source=collection_archive---------3----------------------->

![](img/24315f79df29ba8033e55c01664acac3.png)

致力于安全性的 [Inverno Framework](https://inverno.io) 的最新版本包括 JSON 对象签名和加密(JOSE)规范的完整 Java 实现，包括 JSON Web Key ( [JWK](https://datatracker.ietf.org/doc/html/rfc7517) )、JSON Web Signature ( [JWS](https://datatracker.ietf.org/doc/html/rfc7515) )、JSON Web Encryption ( [JWE](https://datatracker.ietf.org/doc/html/rfc7516) )和 JSON Web Token ( [JWT](https://datatracker.ietf.org/doc/html/rfc7519) )。

JOSE 对象主要用于传送数字签名(完整性)和/或加密(隐私)的内容。他们对实现基于 JSON Web 令牌的简单分布式身份验证越来越感兴趣，其中包含一组不能伪造的声明的数字签名令牌用于对资源服务器的请求进行身份验证。这种技术被广泛记录并在多个标准中使用，如 [OAuth2](https://oauth.net/2/) 和 [OpenID Connect](https://openid.net/connect/) 。

Inverno security [JOSE 模块](https://search.maven.org/artifact/io.inverno.mod/inverno-security-jose)允许创建、验证或解密遵循上述规范的数字保护内容。它是一个 Inverno 模块，可以很容易地集成到一个 Inverno 应用程序中，但由于 Inverno 框架的非普适性，它也可以用作任何 Java 应用程序中的独立库。

在本文中，我将概述 JOSE 模块中公开的四个服务，分别用于操作 JSON Web 密钥(JWK)、JSON Web 签名(JWS)对象、JSON Web 加密(JWE)对象和 JSON Web 令牌(JWT)。该模块相当完整，并提供了多种功能，如果您想深入了解，请参考 [Inverno 框架参考文档](https://inverno.io/docs/release/reference/html/index.html#json-object-signing-and-encryption)。

# 初始化何塞模块

在 Inverno 应用程序中，声明对模块的依赖就足以在需要的地方注入 JOSE 服务。在常规的 Java 应用程序中，模块必须像任何 Java 对象一样进行初始化。

以下代码显示了如何获得标准的`Jose`模块实例:

注意，JSON 和文本转换器是由 Inverno [boot](https://search.maven.org/artifact/io.inverno.mod/inverno-boot) 模块提供的，它们用于根据内容类型自动转换 JSON 和文本有效载荷。使用媒体类型转换器初始化模块不是强制性的，因为在创建或解析 JOSE 对象时也可以指定特定的编码器或解码器，但是强烈建议这样做，因为这样可以大大简化有效载荷的编码和解码。

这样得到的`Jose`实例暴露了`JWKService`、`JWSService`、`JWEService`和`JWTService`:

这些服务可以用来操作 JOSE 对象。

# JWK 服务

JWK 服务用于管理由 [RFC7517](https://datatracker.ietf.org/doc/html/rfc7517) 和 [RFC7518](https://datatracker.ietf.org/doc/html/rfc7518) 指定的加密密钥，它可以从 JWK 密钥库、Java 密钥库、JWK 集合 URL 或原始密钥值加载密钥。它还能够为 JOSE 规范中定义的各种算法生成密钥。

以下示例显示了如何生成简单的对称密钥:

可以使用一个`ObjectMapper`将密钥序列化为 JSON，得到下面的 JSON 对象:

这样获得的 JWK 然后可以用于使用 JWS、JWE 或 JWT 服务来签名或加密 JOSE 对象。

JWK 服务目前支持对称、RSA、椭圆曲线、Edward 曲线和 PBES2 密钥类型。通过在初始化 JOSE 模块时提供定制的 JWK 工厂实现，可以添加对额外的或定制的键类型的支持。

以下示例显示了如何从原始密钥值加载 RSA 密钥:

# JWS 服务

现在我们有了一个密钥，我们可以创建和读取 JSON Web Signature (JWS)对象，这些对象提供由 [RFC7515](https://datatracker.ietf.org/doc/html/rfc7515) 定义的完整性保护。

以下示例显示了如何使用`HS512`算法创建一个简单的 JSON Web 签名，并使用 JWS 紧凑符号对其进行序列化:

内容类型在 JOSE 头中指定，这允许 JWS 服务确定使用哪个转换器来编码有效载荷。用于对 JWS 进行数字签名的 JWK 是显式指定的，但是当密钥存储在模块的 JWK 存储或 Java 密钥存储中时，它也可以通过 id 来解析。

然后，可以将压缩的 JWS 传送给接收者，接收者可以使用相同的密钥来验证数字签名。

在上面的例子中，我使用了对称算法，这意味着双方必须知道密钥，非对称签名算法，如 RSA 或椭圆曲线算法显然也可以使用。

该规范还规定了高度不安全的`NONE`算法。必须特别注意防止应用程序用这种算法接受 JWS 对象，因为 JOSE 模块遵循规范，在这种情况下不会产生任何错误。

# JWE 服务

JSON Web Encryption (JWE)对象除了完整性之外还提供机密性，如 [RFC7516](https://datatracker.ietf.org/doc/html/rfc7516) 所定义的。它通常使用两种算法:一种是加密、包装或导出生成的内容加密密钥(CEK ),另一种是使用 CEK 实际签署和加密 JWE。

以下示例显示了如何使用`RSA-OAEP`密钥管理算法和`A256GCM`加密算法创建 JSON Web 加密对象，并使用 JWE 紧凑符号对其进行序列化:

这里，在 JOSE 报头中没有指定内容类型，但是当构建 JWE 以选择用于编码有效载荷的转换器时，可以明确地指定特定的编码器来绕过媒体类型转换器。至于 JWS，用于加密生成的内容加密密钥的 JWK 是显式指定的，但是当密钥存储在模块的 JWK 存储或 Java 密钥存储中时，它也可以通过 id 来解析。

然后，可以将紧凑的 JWE 传送给接收方，接收方可以使用私钥来验证和解密内容:

在上面的例子中，我使用了一个非对称密钥管理算法，因此只需要公钥来创建只能用私钥解密的 JWE。

# JWT 服务

根据 RFC7519 的定义，JSON Web 令牌只不过是一个 JWS 或 JWE，其有效载荷是 JWT 声明集。它提供了一组标准声明和自定义声明，可用于传递有效性信息:过期时间，而不是之前，受众…以及身份信息:发布者，主题…

JSON Web 令牌被广泛用于在无状态分布式环境中实现认证，其通过使用可信密钥来验证只能由可信机构生成的数字签名，并验证限制令牌有效性的 JWT 声明集，以防止基于被盗令牌的攻击。

下面的例子显示了如何创建一个由`joe`发布的 JSON Web 令牌，并在一天内作为 JWS 对象到期:

然后，可以将紧凑的 JWT 传送给接收者，该接收者可以使用可信密钥来验证数字签名，并验证 JWT 声明集:

在上面的代码中，JWS 签名在读取 JWT 契约表示时被验证，JWT 声明集在使用前被验证。默认情况下，过期时间是有效的，我们在发行者上显式添加了一个必须是`bob`的检查，当 JWT 无效时，会抛出一个错误。

# 摘要

本文展示了如何使用 Inverno security JOSE 模块来操作 JOSE 对象，尤其是 JSON Web 令牌。该模块完全集成在 Inverno 框架中，特别用于基于令牌的身份验证，但是您也可以在任何 Java 应用程序中使用它，例如 Spring Boot 应用程序，对于该应用程序，很容易将上述服务作为可注入的 beans 公开。

该库相当完整，并严格实现了以下规范:

*   [RFC 7515](https://datatracker.ietf.org/doc/html/rfc7515) JSON 网络签名(JWS)
*   [RFC 7516](https://datatracker.ietf.org/doc/html/rfc7516) JSON 网页加密(JWE)
*   [RFC 7517](https://datatracker.ietf.org/doc/html/rfc7517) JSON Web Key (JWK)
*   JSON 网络算法(JWA)
*   JSON 网络令牌(JWT)
*   [RFC 7638](https://datatracker.ietf.org/doc/html/rfc7638) JSON Web 密钥(JWK)指纹
*   [RFC 7797](https://datatracker.ietf.org/doc/html/rfc7797) JSON Web 签名(JWS)未编码有效负载选项
*   [RFC 8037](https://datatracker.ietf.org/doc/html/rfc8037) CFRG 椭圆曲线 Diffie-Hellman (ECDH)和 JSON 对象签名和加密(JOSE)中的签名
*   [RFC 8812](https://datatracker.ietf.org/doc/html/rfc8812) 针对 Web 认证(WebAuthn)算法的 CBOR 对象签名和加密(COSE)以及 JSON 对象签名和加密(JOSE)注册

它还提供了一些有趣的功能，例如:

*   使用媒体类型转换器的自动有效载荷编码和解码
*   通过来自 JWK 存储、Java 密钥存储或 JWK 集合 URL 的密钥 id 或指纹自动解析密钥
*   证书路径验证

它是可扩展的，允许实现对附加密钥类型和算法的支持。还可以定义特定的验证器来验证 JWT 声明集。

请参考 [Inverno 框架参考文档](https://inverno.io/docs/release/reference/html/index.html#json-object-signing-and-encryption)以全面了解如何在您的应用中使用该模块。