# 如何使用 JPA 和 EclipseLink 将 PostgreSQL 数组类型映射到列表

> 原文：<https://itnext.io/how-to-map-postgresql-array-type-to-list-using-jpa-and-eclipselink-b4e25ca13490?source=collection_archive---------0----------------------->

![](img/154b5798221a43a9ffa1ee9a2516bb92.png)

在 PostgreSQL 数据库中，可以将表中的列作为任意类型(内置或用户定义的基本类型、枚举类型、复合类型、范围类型或域)的可变长度多维[数组](https://www.postgresql.org/docs/current/arrays.html):

```
CREATE TABLE product
(
  id        bigint,
  images    text[]
)
```

这个功能类似于[甲骨文](https://docs.oracle.com/cd/B28359_01/appdev.111/b28370/collections.htm#CHDEIJHD) `[VARRAY](https://docs.oracle.com/cd/B28359_01/appdev.111/b28370/collections.htm#CHDEIJHD)` [类型](https://docs.oracle.com/cd/B28359_01/appdev.111/b28370/collections.htm#CHDEIJHD)。您可以按以下格式将数据插入到该类型的列中:

```
INSERT INTO product(id, images)
   VALUES (1, '{"url1", "url2", "url3"}');
```

您可以使用 PostgreSQL 数组类型可用的[函数和操作符](https://www.postgresql.org/docs/9.1/functions-array.html#ARRAY-OPERATORS-TABLE)来获取数据:

```
SELECT * FROM PRODUCT WHERE images @> ARRAY['url2']
```

您必须使用针对数组数据类型的 JDBC API 的 [PostgreSQL 扩展](https://jdbc.postgresql.org/documentation/head/arrays.html),以便在 PostgreSQL 的 where 子句中将数组用作列类型、函数参数和标准。你可以在这里阅读关于 PostgreSQL 数据类型。

不幸的是，JPA 不支持映射数组数据类型，我们有三种选择:

*   在 JPA 中使用一种替代方法来建模一对多关系(而不是数组数据类型),如 [ElementCollection](https://en.wikibooks.org/wiki/Java_Persistence/ElementCollection) 或 [OneToMany](https://en.wikibooks.org/wiki/Java_Persistence/OneToMany) 或…
*   对数组数据类型使用原生查询和对 JDBC API 的 PostgreSQL 扩展。
*   对数组数据类型使用 EclipseLink 或 Hibernate 特定的特性。

EclipseLink 和 hibernate 支持以不同的方式映射该数据类型。在下面的链接中，您可以找到一篇关于如何使用 Hibernate Types 项目在 PostgreSQL 中映射数组数据类型的好文章。

[](https://vladmihalcea.com/postgresql-array-java-list/) [## 如何用 JPA 和 Hibernate 将 PostgreSQL 数组映射到 Java 列表

### 在本文中，我将向您展示如何映射 PostgreSQL 数组列类型(例如，text、int、double、enum、date…

vladmihalcea.com](https://vladmihalcea.com/postgresql-array-java-list/) 

## 使用@Struct 和@Array 在 EclipseLink 中映射数组数据类型

就像 Hibernate 一样，在 EclipseLink 中有一个映射数组数据类型的特定方法。为此，您必须使用`[@Struct](https://www.eclipse.org/eclipselink/documentation/2.4/jpa/extensions/a_struct.htm)`引入一个类来映射到数据库`Struct`类型，并在映射到数组数据类型的集合属性上使用`[@Array](https://www.eclipse.org/eclipselink/documentation/2.4/jpa/extensions/a_array.htm)`:

```
@Entity
@Struct*(*name = "images"*)* public class Product implements Serializable *{* @Id
 *@GeneratedValue(strategy = GenerationType.IDENTITY)* private long id;

    @Column*(*columnDefinition="TEXT[]"*)* @Array*(*databaseType="varchar"*)* private List*<*String*>* images;
.
.
.
```

您还可以使用`[@Struct](https://www.eclipse.org/eclipselink/documentation/2.4/jpa/extensions/a_struct.htm)`来映射 Oracle `VARRAY`数据类型。

我不得不提到，你应该使用`@Column`和`@Array`属性来引入你的数组的类型。

现在，您可以使用 EclipseLink 轻松存储您的图像阵列:

```
Product product1 = new Product*()*;
product1.setImages*(*Arrays.*asList(*"url1", "url2", "url3"*))*;em.persist*(*product1*)*;
```

您还可以通过使用 JPA 原生查询，使用 PostgreSQL 数组类型的特定[函数和操作符](https://www.postgresql.org/docs/9.1/functions-array.html#ARRAY-OPERATORS-TABLE)来获取数据:

```
List*<*String*>* images = Arrays.*asList(*"url2"*)*;
String imagesStr = images.stream*()*.collect*(joining(*",", "{", "}"*))*;

Query query = em.createNativeQuery*(* "SELECT * FROM PRODUCT WHERE images @> CAST(? AS text[])",
        Product.class
*)*.setParameter*(*1, imagesStr*)*;

List*<*Product*>* result = query.getResultList*()*;
```