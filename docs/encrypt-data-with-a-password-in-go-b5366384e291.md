# 在 Go 中使用密码加密数据

> 原文：<https://itnext.io/encrypt-data-with-a-password-in-go-b5366384e291?source=collection_archive---------1----------------------->

![](img/d5ae30e61536c23cd4839666c1f50b9d.png)

照片由[谢恩·艾弗里](https://unsplash.com/@shaneavery?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄

# 介绍

当我们加密数据时，通常我们会创建一个随机密钥来解密该数据。在某些特定情况下，人们希望使用用户指定的密钥来解密数据，就像密码一样。然而，用于加密算法的密钥通常需要至少 32 个字节。但是，很可能我们的密码达不到这个标准，所以我们需要一个解决方案。最近，我需要这样一种方法，在这篇文章中，我将列出我所做的来解决这个问题。但在我们进入实质之前。

**免责声明**:我不是加密专家，我已经提到了我在这篇文章中提供的解决方案的来源。我恳求你阅读/观看这些资料来更好地理解它。同样，如果帖子/代码中有任何错误，请让我知道或留下评论，以便我可以更新它，这样就不会有错误的方法/技术的延续。

好了，既然我们已经解决了这个问题，让我们开始吧！(注:你也可以在这里查看这个帖子[)](https://bruinsslot.jp/post/golang-crypto)

# 加密

让我们首先从加密我们的数据开始。我们将从创建接受一个`key`和一个`data`参数的`Encrypt`函数开始。基于此，我们将使用`key`对可以解密的数据进行加密。首先，我们将使用 32 个随机字节生成密钥，稍后我们将用我们的密码替换它。下面显示了能够加密我们的数据的代码，由生成的密钥提供。

```
import (
    "crypto/aes"
    "crypto/cipher"
    "crypto/rand"
)

func Encrypt(key, data []byte) ([]byte, error) {
    blockCipher, err := aes.NewCipher(key)
    if err != nil {
        return nil, err
    }

    gcm, err := cipher.NewGCM(blockCipher)
    if err != nil {
        return nil, err
    }

    nonce := make([]byte, gcm.NonceSize())
    if _, err = rand.Read(nonce); err != nil {
        return nil, err
    }

    ciphertext := gcm.Seal(nonce, nonce, data, nil)

    return ciphertext, nil
}
```

所以，让我们检查一下代码，看看我们在做什么。

```
func Encrypt(key, data []byte) ([]byte, error)
```

首先，我们开始创建我们的`Encrypt`函数，它将接受一个`key`和一个`data`参数。我们将使用一个`byte`切片代替一个`io.Reader`作为`data`参数。而使用`io.Reader`将允许我们对实现`io.Reader`接口的所有其他类型使用`Encrypt`函数。(Ryer [2015](https://gist.github.com/erroneousboat/d01e57e3c0470c645e72ae663c7955fa#ref-ryer2015) )然而，正是因为`io.Reader`作为数据流的性质，当我们想要解密`ciphertext`时，我们需要看到它的整体。一种解决方案是将流分成离散的块，然而这将大大增加问题的复杂性。(iso[2015](#efb4))

```
blockCipher, err := aes.NewCipher(key)
```

我们正在根据我们提供的`key`初始化分组密码。这里我们使用的是`crypto/aes` [](#63b9)包，它实现了 AES [](#5627)[⁴](#bf25) (高级加密标准)加密算法。AES 是一种对称密钥加密算法，对于现代用例来说足够安全。此外，AES 在大多数平台上使用硬件加速，因此使用起来会非常快。(坦克斯利 [2016](#985d) )

```
gcm, err := cipher.NewGCM(blockCipher)
```

这里我们用特定的*模式*包装分组密码。我们这样做是因为我们不应该直接使用一个`cipher.Block`接口。这是因为分组密码只加密 16 字节的数据，仅此而已。所以如果你调用`blockCiper.Encrypt()`，它只会加密前 16 个字节。因此，我们需要在此之上的东西，包装分组密码，这些被称为*模式*。同样，我们有几种*模式*可供选择，这里我们将使用*伽罗瓦计数器模式* (GCM) [⁵](#4df4) ，具有标准随机数长度。

只有 GCM 提供认证加密，它实现了`cipher.AEAD`接口(认证加密和相关数据) [⁶](#810e) 。认证加密意味着你的数据不仅是机密的、秘密的和加密的，而且现在是防篡改的。如果有人改变了`ciphertext`，你将无法有效地解密它。当你使用认证加密时，如果有人篡改了你的数据，它就无法解密。(坦克斯利[2016](#985d)；2015 年

```
nonce := make([]byte, gcm.NonceSize())
if _, err = rand.Read(nonce); err != nil {
    return nil, err
}
```

在加密我们的字节之前，我们需要生成一个随机化的`nonce`，它的长度由 GCM 指定。`nonce`代表:*号曾经使用过*，是一段不应该重复的数据，并且只能与任何特定的键组合使用一次。含义:不要重复一个`key`和一个`nonce`的组合超过一次。但是，你如何保持跟踪呢？如果我们对一个`nonce`使用足够大的数字，我们可能对这个用例没问题。(Isom[2015](#efb4)；Viega 和 Messier [2003](#be38) ，134-35)我们通过使用 Go 的`crypto/rand`包将随机字节读入`nonce`字节片来实现。 [⁷](#c9f0)

```
encryptedData := gcm.Seal(nonce, nonce, bData, nil)
```

我们将用来加密数据的`nonce`也需要用来解密数据。所以我们需要能够在解密的时候引用它，策略之一就是把它加到加密的数据里。在这个例子中，我们将把`nonce`加到加密数据的前面。我们通过将`nonce`作为`Seal`函数的第一个参数`dst`传入来实现这一点，因此加密的数据将被附加到它的后面。 [⁸](#1aa0) 我们可以做到这一点，因为`nonce`不一定要保密，它只需要独一无二。(坦克斯利 [2016](#985d) )

# 解释

现在，我们能够加密我们的数据，让我们实现`Decrypt`函数。

```
import (
    "crypto/aes"
    "crypto/cipher"
)func Decrypt(key, data []byte) ([]byte, error) {
    blockCipher, err := aes.NewCipher(key)
    if err != nil {
        return nil, err
    } gcm, err := cipher.NewGCM(blockCipher)
    if err != nil {
        return nil, err
    } nonce, ciphertext := data[:gcm.NonceSize()], data[gcm.NonceSize():] plaintext, err := gcm.Open(nil, nonce, ciphertext, nil)
    if err != nil {
        return nil, err
    } return plaintext, nil
}
```

让我们再次检查代码，看看它做了什么。它与`Encrypt`函数的代码基本相同，所以让我们检查不同的部分。

```
nonce, ciphertext := data[:gcm.NonceSize()], data[gcm.NonceSize():]
```

还记得在上一节中，我们使用`gcm.Seal`在`data`前添加了`nonce`来创建`ciphertext`吗？现在我们需要拆分这些部分，这样我们就可以独立使用它们了。我们通过根据`gcm`提供的`nonce`的大小分割`data`来创建这些部分。

```
plaintext, err := gcm.Open(nil, nonce, ciphertext, nil)
```

现在，我们使用`gcm.Open`将`ciphertext`解密成`plaintext`。 [⁹](#9327)

# 钥匙

我们已经向`Encrypt`和`Decrypt`函数传递了一个`key`，但是我们还没有创建它，所以让我们开始吧。

```
import (
    "crypto/rand"
)func GenerateKey() ([]byte, error) {
    key := make([]byte, 32) _, err := rand.Read(key)
    if err != nil {
        return nil, err
    } return key, nil
}
```

这里我们使用 Go 的`crypto/rand`包生成一个随机的`key`。对于 AES，我们需要一个长度为 32 字节的`key`,所以我们制作一个大小为 32 的字节片。然后我们让`rand.Read()`用随机字节填充切片。 [⁰](#dcc0)

现在我们已经有足够的信息来加密和解密一些数据，所以让我们把它们放在一起并进行测试:

```
// crypto.go
package mainimport (
    "crypto/aes"
    "crypto/cipher"
    "crypto/rand"
    "encoding/hex"
    "fmt"
    "log"
)func Encrypt(key, data []byte) ([]byte, error) {
    blockCipher, err := aes.NewCipher(key)
    if err != nil {
        return nil, err
    } gcm, err := cipher.NewGCM(blockCipher)
    if err != nil {
        return nil, err
    } nonce := make([]byte, gcm.NonceSize())
    if _, err = rand.Read(nonce); err != nil {
        return nil, err
    } ciphertext := gcm.Seal(nonce, nonce, data, nil) return ciphertext, nil
}func Decrypt(key, data []byte) ([]byte, error) {
    blockCipher, err := aes.NewCipher(key)
    if err != nil {
        return nil, err
    } gcm, err := cipher.NewGCM(blockCipher)
    if err != nil {
        return nil, err
    } nonce, ciphertext := data[:gcm.NonceSize()], data[gcm.NonceSize():] plaintext, err := gcm.Open(nil, nonce, ciphertext, nil)
    if err != nil {
        return nil, err
    } return plaintext, nil
}func GenerateKey() ([]byte, error) {
    key := make([]byte, 32) _, err := rand.Read(key)
    if err != nil {
        return nil, err
    } return key, nil
}func main() {
    data := []byte("our super secret text") key, err := GenerateKey()
    if err != nil {
        log.Fatal(err)
    } ciphertext, err := Encrypt(key, data)
    if err != nil {
        log.Fatal(err)
    } fmt.Printf("ciphertext: %s\n", hex.EncodeToString(ciphertext)) plaintext, err := Decrypt(key, ciphertext)
    if err != nil {
        log.Fatal(err)
    } fmt.Printf("plaintext: %s\n", plaintext)
}
```

并且，我们可以使用以下代码运行此示例:

```
$ go run crypto.go
```

现在，我们有足够的东西用一个随机密钥来加密和解密我们的`data`。这很酷，现在我们有了一个允许我们加密和解密我们的`data`的`key`。但是这意味着`key`现在变成了我们的密码，我们不能自己选择，另外它有 32 个字节的长度。

但是，正如帖子开头提到的，我们希望能够通过提供我们自己的`key`即我们选择使用的密码来加密和解密数据。我们将在下一节中完成这项工作。

# 密码

现在，`aes.NewCipher()`需要一个 16、24 或 32 字节的`key`，在这个例子中我们使用的是一个 32 字节的`key`。然而，我们的密码可能不会是 32 字节。所以我们需要将我们的密码转换成一个合适的`key`。我们通过使用*密钥派生函数* (KDF) [](#89bb)来“延伸”密码，使其成为合适的加密密钥。这个按键拉伸 [](#3f07) 的特点就是慢。这样做是为了使攻击者需要花费大量的资源来试图对密码进行暴力攻击。我们有几个 KDF 的选项:Argon2 [](#ea51)、scrypt [⁴](#a85f) 、bcrypt [⁵](#38e8) 和 pbkdf2 [⁶](#af62) 。选择哪一个取决于几个因素，但主要是它有多安全。[⁷](#3055)t20】⁸t22】⁹t24】⁰

典型地，在 KDF 中，我们有一个`password`、一个`salt`和一个`iterations`自变量。`salt` [](#fcb3)用于防止攻击者仅存储密码/密钥对，并防止攻击者预先计算派生密钥的字典，因为不同的 salt 产生不同的输出。每个密码都必须用用于导出密钥的`salt`进行检查。(Isom[2015](#efb4)；维基百科 [2020](#7ac3) )其中`salt`与`nonce`相关，也需要随机生成。和`nonce`一样，`salt`不需要保密，它需要独一无二。`iterations`参数或*难度参数*，表示重复该过程的次数。这是因为，即使有了`salt`，一个*字典攻击*仍然是可能的，但是有了`iterations`计数，它将减慢从一个密码计算一个`key`的时间。(维耶加和梅西耶 [2003](#be38) ，141-42)

在这个例子中，我们将使用 *scrypt* ，所以让我们看看如何在我们的程序中实现它。

```
import (
    "crypto/rand" "golang.org/x/crypto/scrypt"
)func DeriveKey(password, salt []byte) ([]byte, []byte, error) {
    if salt == nil {
        salt = make([]byte, 32)
        if _, err := rand.Read(salt); err != nil {
            return nil, nil, err
        }
    } key, err := scrypt.Key(password, salt, 1048576, 8, 1, 32)
    if err != nil {
        return nil, nil, err
    } return key, salt, nil
}
```

让我们再次检查代码，看看它做了什么。

```
func DeriveKey(password, salt []byte) ([]byte, []byte, error)
```

这里我们接受作为参数的一部分字节的密码，并返回结果`key`和`salt`。

```
salt := make([]byte, 32)
if _, err := rand.Read(salt); err != nil {
    return err
}
```

就像我们的`Encrypt`函数一样，我们将用 32 个随机字节创建`salt`。

```
key, err := scrypt.Key(password, salt, 1048576, 8, 1, 32)
```

这里我们使用来自`golang.org/x/`库的`scrypt`包。 [⁴](#e5f2) 从文档中我们可以看到`Key`函数接受以下参数:

```
func Key(password, salt []byte, N, r, p, keyLen int) ([]byte, error)
```

论点`password`和`salt`不言自明。`N`是迭代的次数。在 C. Percival 给出的演示中，建议对于交互式登录使用 16384 (2^14)迭代，对于文件加密使用 1048576 (2^20)迭代。(珀西瓦尔 [2005a](#bbcd) ，[2005 b](#3f6a)；Isom [2015](#efb4) 自变量`r`和`p`必须满足*r***p*<2^30，如果不满足限制，函数返回一个`nil`字节片和一个错误。(Golang 文档 [2020](#ea28) )。`r`参数定义了相对内存开销参数，它控制底层哈希中的块大小，建议值为 8。`p`参数是相对 CPU 成本参数，其推荐值为 1。(Isom[2015](#efb4)；Percival [2005a](#bbcd) )参数`keyLen`定义了作为键返回的字节长度，如前所述，这将是 32 个字节。

# 结果

现在我们已经创建了我们的`DeriveKey`函数，我们需要更新我们的代码来支持它。让我们这样做，它应该类似于下面的代码:

```
// scrypt.go
package mainimport (
    "crypto/aes"
    "crypto/cipher"
    "crypto/rand"
    "crypto/sha256"
    "encoding/hex"
    "fmt"
    "log" "golang.org/x/crypto/scrypt"
)func Encrypt(key, data []byte) ([]byte, error) {
    key, salt, err := DeriveKey(key, nil)
    if err != nil {
        return nil, err
    } blockCipher, err := aes.NewCipher(key)
    if err != nil {
        return nil, err
    } gcm, err := cipher.NewGCM(blockCipher)
    if err != nil {
        return nil, err
    } nonce := make([]byte, gcm.NonceSize())
    if _, err = rand.Read(nonce); err != nil {
        return nil, err
    } ciphertext := gcm.Seal(nonce, nonce, data, nil) ciphertext = append(ciphertext, salt...) return ciphertext, nil
}func Decrypt(key, data []byte) ([]byte, error) {
    salt, data := data[len(data)-32:], data[:len(data)-32] key, _, err := DeriveKey(key, salt)
    if err != nil {
        return nil, err
    } blockCipher, err := aes.NewCipher(key)
    if err != nil {
        return nil, err
    } gcm, err := cipher.NewGCM(blockCipher)
    if err != nil {
        return nil, err
    } nonce, ciphertext := data[:gcm.NonceSize()], data[gcm.NonceSize():] plaintext, err := gcm.Open(nil, nonce, ciphertext, nil)
    if err != nil {
        return nil, err
    } return plaintext, nil
}func DeriveKey(password, salt []byte) ([]byte, []byte, error) {
    if salt == nil {
        salt = make([]byte, 32)
        if _, err := rand.Read(salt); err != nil {
            return nil, nil, err
        }
    } key, err := scrypt.Key(password, salt, 1048576, 8, 1, 32)
    if err != nil {
        return nil, nil, err
    } return key, salt, nil
}func main() {
    var (
        password = []byte("mysecretpassword")
        data     = []byte("our super secret text")
    ) ciphertext, err := Encrypt(password, data)
    if err != nil {
        log.Fatal(err)
    } fmt.Printf("ciphertext: %s\n", hex.EncodeToString(ciphertext)) plaintext, err := Decrypt(password, ciphertext)
    if err != nil {
        log.Fatal(err)
    } fmt.Printf("plaintext: %s\n", plaintext)
}
```

而且，我们能够运行和测试它:

```
# First we need to get the scrypt package
$ go get -u golang.org/x/crypto/scrypt$ go run scrypt.go
```

我们已经更新了一些部分，所以让我们过一遍。

```
key, salt, err := DeriveKey(key, nil)
```

在`Encrypt`函数中，我们通过传递包含在`key`参数中的密码来创建我们的密钥。我们将`nil`作为 salt 参数传入，这是因为我们想要创建`salt`，因为这是我们第一次加密`data`。

```
ciphertext = append(ciphertext, salt...)
```

此外，在`Encrypt`函数中，我们将`salt`追加到`ciphertext`中。

```
salt, data := data[len(data)-32:], data[:len(data)-32]
```

而且，因为我们将`salt`追加到了`ciphertext`，我们需要在`Decrypt`函数中对其进行分割，因为我们将在`DeriveKey`函数中使用它。

```
key, _, err := DeriveKey(key, salt)
```

正如您在这里看到的，我们将 salt 传递给`DeriveKey`函数，我们将能够检索到我们用来加密数据的`key`。

# 结论

因此，我们创建了两种方法来加密和解密 Go 中的数据。首先，我们使用 AES 加密算法对数据进行加密，为此我们创建了一个用于解密数据的随机密钥。随后，我们更新了代码以支持使用密码作为我们的密钥。我们已经通过使用*密钥派生函数*对我们的密码进行*密钥扩展*来实现这一点，并且我们已经使用 *scrypt* 来实现这一点。希望这篇文章对你有用，我再次建议你阅读和观看我列出的资源，并查看其他资源，以全面了解如何正确安全地加密数据，如果你有任何建议，请告诉我。

# 参考

Golang 文档。2020."包加密。"2020.[https://godoc.org/golang.org/x/crypto/scrypt](https://godoc.org/golang.org/x/crypto/scrypt)。

伊索姆凯尔。2015.*实用密码术用 Go* 。Leanpub。[https://leanpub.com/gocrypto/read](https://leanpub.com/gocrypto/read)。

帕西瓦尔，C. 2005a。" scrypt:一个新的密钥派生函数."[https://www . BSD can . org/2009/schedule/attachments/86 _ scrypt _ slides . pdf](https://www.bsdcan.org/2009/schedule/attachments/86_scrypt_slides.pdf)。

— -.2005 年 b"通过顺序记忆硬函数的强密钥派生."https://www.tarsnap.com/scrypt/scrypt.pdf。

Ryer，M. 2015。“木卫一。深度读者。”2015.[https://medium . com/@ matryer/golang-advent-calendar-day-seventh-io-reader-in-depth-6f 744 bb 4320 b](https://medium.com/@matryer/golang-advent-calendar-day-seventeen-io-reader-in-depth-6f744bb4320b)。

坦克斯利，G. 2016。“去找密码开发人员。”[https://youtu.be/2r_KMzXB74w](https://youtu.be/2r_KMzXB74w)。

Viega，j .和 M. Messier。2003.*C 和 C++* 安全编程指南。奥莱利媒体。

维基百科。2020."密钥推导函数——维基百科，免费的百科全书."2020.[https://en.wikipedia.org/wiki/Key_derivation_function](https://en.wikipedia.org/wiki/Key_derivation_function)。

# 脚注

[1]如何用 streams 实现加密的一些参考: [Reddit 评论](https://www.reddit.com/r/golang/comments/87wcik/how_do_you_encrypt_large_ioreader_streams/dwp2k0g?utm_source=share&utm_medium=web2x)；[迈克尔·特纳——加密围棋流](https://medium.com/blend-engineering/encrypting-streams-in-go-6cff6062a107)； [Minio — Github 发行](https://github.com/minio/sio/issues/26#issuecomment-378012290)

[2] [Go 文档:AES](https://golang.org/pkg/crypto/aes/)

【3】[维基百科:AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)

[4] [电脑爱好者— AES 讲解](https://www.youtube.com/watch?v=O4xNJsjtN6E)

[5] [维基百科:GCM](https://en.wikipedia.org/wiki/Galois/Counter_Mode)

[6] [Go 文档:AEAD](https://golang.org/pkg/crypto/cipher/#AEAD)

[7] [Go 文档:兰德。阅读](https://golang.org/pkg/crypto/rand/#Read)

[8] [Go 源代码:GCM](https://golang.org/src/crypto/cipher/gcm.go)

[9] [围棋源代码:GCM](https://golang.org/src/crypto/cipher/gcm.go)

[10] [围棋文献:兰德。阅读](https://golang.org/pkg/crypto/rand/#Read)

[11] [维基百科:KDF](https://en.wikipedia.org/wiki/Key_derivation_function)

[12] [维基百科:按键拉伸](https://en.wikipedia.org/wiki/Key_stretching)

[13] [维基百科:Argon2](https://en.wikipedia.org/wiki/Argon2)

[14] [维基百科:scrypt](https://en.wikipedia.org/wiki/Scrypt)

[15] [维基百科:bcrypt](https://en.wikipedia.org/wiki/Bcrypt)

[16] [维基百科:pbkdf2](https://en.wikipedia.org/wiki/PBKDF2)

[17] [Michele Prezioso —密码散列:PBKDF2，Scrypt，Bcrypt](https://medium.com/@mpreziuso/password-hashing-pbkdf2-scrypt-bcrypt-and-argon2-e25aaf41598e)

[18][Michele Prezioso——密码散列:Scrypt、Bcrypt 和 ARGON2](https://medium.com/@mpreziuso/password-hashing-pbkdf2-scrypt-bcrypt-and-argon2-e25aaf41598e)

[19][stack exchange—pbk df 和 SHA 有什么区别，为什么要一起使用？](https://crypto.stackexchange.com/questions/35275/whats-the-difference-between-pbkdf-and-sha-and-why-use-them-together/61306#61306)

[20] [Github — Bitwarden 第 52 期](https://github.com/bitwarden/jslib/issues/52)

[21] [Github — Bitwarden 第 589 期](https://github.com/bitwarden/server/issues/589)

[22] [维基百科:pbkdf2 替代品](https://en.wikipedia.org/wiki/PBKDF2#Alternatives_to_PBKDF2)

[23] [维基百科:Salt(密码学)](https://en.wikipedia.org/wiki/Salt_(cryptography))

[24] [Go 文档:scrypt](https://godoc.org/golang.org/x/crypto/scrypt)

*原载于*[*https://bruinsslot.jp/post/golang-crypto/*](https://bruinsslot.jp/post/golang-crypto/)*。*