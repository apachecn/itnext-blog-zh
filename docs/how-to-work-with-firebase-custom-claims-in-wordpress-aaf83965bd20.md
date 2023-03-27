# 如何在 WordPress 中使用 Firebase 自定义声明

> 原文：<https://itnext.io/how-to-work-with-firebase-custom-claims-in-wordpress-aaf83965bd20?source=collection_archive---------2----------------------->

![](img/0f9d2b743fbba0dd896a26e6ef25186e.png)

> 如果你对 Integrate Firebase PRO 版本感兴趣，请阅读完整更新的文档:[https://firebase-wordpress-docs.readthedocs.io/](https://firebase-wordpress-docs.readthedocs.io/)

【https://wordpress.dalenguyen.me/】演示:[T5](https://wordpress.dalenguyen.me/)

*   文章 1: [如何将 Firebase 集成到 WordPress](/how-to-integrate-firebase-and-wordpress-b017ee274687)
*   第 2 篇:[如何从 Firestore 检索数据并显示在 WordPress 上](/how-to-retrieve-data-from-firestore-and-display-on-wordpress-8638854a762e)
*   第三篇:[**如何在 WordPress**](https://medium.com/@dalenguyen/how-to-work-with-firebase-custom-claims-in-wordpress-aaf83965bd20?sk=85786e3739d42b18c3e2c7344bc5f436) 中使用 Firebase 自定义声明
*   第 4 条:[将数据从 WordPress 保存到 Firebase(实时+ Firestore)](/how-to-save-data-from-wordpress-to-firebase-realtime-firestore-2eda917d01fb)
*   第五条: [Firebase WordPress 用户集成](/firebase-wordpress-user-integration-c18a28e41cbd)
*   第六篇:[如何在 WordPress 仪表盘中管理 Firebase 用户](/firebase-users-management-in-wordpress-dashboard-61b4a1ca066#d4c2-1605c6edec5f)
*   第 7 篇:[如何将数据从 WordPress 同步到 Firebase](/sync-data-from-wordpress-to-firebase-d6e5860d3a06)
*   第 8 条:[一键登录 WordPress & Firebase 或通过电子邮件链接](https://medium.com/@dalenguyen/one-click-login-to-wordpress-firebase-or-via-email-link-d7610d71cd23)
*   第 9 条:[从 WordPress 上传文件到云存储](https://medium.com/@dalenguyen/upload-files-to-cloud-storage-from-wordpress-e8acc8ce70cd)
*   第十条:[远程 URL 登录到 Firebase & WordPress](/remote-url-login-to-firebase-wordpress-2027fad7c159)
*   第 11 条: [2 种给 WordPress 添加 Firebase 认证的方法& WooCommerce](https://dalenguyen.medium.com/2-ways-to-add-firebase-authentication-to-wordpress-woocommerce-df500c3b104e)
*   第十二条:[如何将 WooCommerce 购买数据发送到 Firebase](https://dalenguyen.medium.com/how-to-send-woocommerce-purchase-data-to-firebase-8c8b4c8cff39)
*   第 13 条:[从 WordPress](https://dalenguyen.medium.com/create-manage-firebase-database-from-wordpress-13347d8ffb2e) 创建&管理 Firebase 数据库

很久以前，我写了一个插件，让[将 Firebase 集成到 WordPress](https://github.com/dalenguyen/firebase-wordpress-plugin) 中，但是还没有一个合适的指南来指导如何使用它。在本教程中，我将向你展示如何在 WordPress 中使用 Firebase 自定义声明。这是[如何从 Firestore 检索数据并显示在 WordPress](/how-to-retrieve-data-from-firestore-and-display-on-wordpress-8638854a762e) 上的下一部分

我们已经知道如何获取数据并显示给 WordPress。如果您想向管理员用户显示数据(当然是通过[自定义声明](https://firebase.google.com/docs/auth/admin/custom-claims))，向其他人显示其他内容或者什么都不显示，该怎么办？

这是检索和显示数据的原始代码。

函数将在 WordPress 上显示数据。现在，我们将在海关索赔检查下包装它。

在检查用户的自定义声明，并确保它的管理员。然后我们将调用**showFirestoreDatabase()**函数。

本教程不局限于从 Firestore 检索数据，您可以在检查自定义声明后添加任何功能或特性。

在下一次更新中，我将更新插件，通过短码支持自定义声明。