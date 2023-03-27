# 使用 Django-自动预取优化 Django REST 框架性能

> 原文：<https://itnext.io/optimizing-django-rest-framework-performance-with-django-auto-prefetching-c70ce316463e?source=collection_archive---------3----------------------->

*tldr:django-auto-prefetching 是一个在使用 django-rest-framework 时，通过使用* `*prefetch_related*` *和* `*select_related*` *从数据库中提取正确对象来自动优化您的端点的库。你可以在*[*PyPI*](https://pypi.org/project/django-auto-prefetching/)*和* [*Github*](https://github.com/GeeWee/django-auto-prefetching) 上找到

# Django REST 框架和 n+1 问题

Django REST 框架(DRF)是一个快速构建健壮 REST API 的框架。然而，当获取具有嵌套关系的模型时，我们会遇到性能问题。DRF 变得*迟钝*。

这不是因为 DRF 本身，而是因为 n+1 问题。当我们有了一个模型，比如说`ItalianChef`有了类似`FavouriteDish`的关系，我们希望获取所有意大利厨师及其相应的最喜爱的菜肴。这意味着我们可能会进行大量的查询，因为 DRF 首先会获取所有的厨师，然后为每个厨师分别获取他们最喜欢的菜肴。这意味着，如果有 20 个厨师，我们将最终为喜爱的菜肴进行 20 次单独的数据库调用。

Django 对这个问题有一个内置的解决方案，`select_related`和`prefetch_related`，它告诉 ORM 你需要什么相关的对象。这意味着我们可以只做一个，而不是做一堆数据库调用。

然而，这有两个问题——很难准确跟踪最终将遍历什么关系，手动编写`select_related`和`prefetch_related`调用非常耗时，更不用说在其他地方的代码发生变化时保持它们的更新了。

# django 简介——自动预取

这个库的目标是尽可能轻松地确保您不会因为 n+1 问题而遇到性能问题。我们从小规模开始，为 DRF 序列化程序自动预取。

# 它是做什么的？

Django-auto-prefetching 有一个 ModelViewSet mixin，它查看任何给定序列化程序使用的字段，并自动计算正确的`select_related`和`prefetch_related`调用。

这意味着一行代码就是您的许多视图所需要的全部优化！

# 生产准备好了吗？

django-自动预取正在丹麦初创公司 [reccoon](https://www.reccoon.dk/) 内部使用，在 Django 代码库中大约有 20k 行代码。我们有许多未优化的端点，我们已经看到，仅仅通过从 AutoPrefetchMixin 继承我们的视图集，我们的整个 API 的响应时间就普遍加快了 30–40%

# 下一步是什么？

我认为在自动预取方面还有很多改进，而且我认为它有可能解决 Django 的很多痛点。通过更多的工程努力，应该可以追踪来自特定`QuerySet`的模型的生命周期，并自动预取所需的相关对象。它可能永远不会完美，但 Django 在许多方面都非常聪明，我们仍然手写这一部分似乎很疯狂。

自己去试试`django-auto-prefetching`吧。你可以在 [PyPI](https://pypi.org/project/django-auto-prefetching/) 上找到它，并确保在 [Github](https://github.com/GeeWee/django-auto-prefetching) 上报告你发现的任何问题