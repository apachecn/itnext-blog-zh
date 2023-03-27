# 使用 JPA 时连接到数据库

> 原文：<https://itnext.io/connection-to-database-when-using-jpa-50ab09b4016?source=collection_archive---------3----------------------->

![](img/2a58ecc5ad17e50c3b530b36f1d0caa2.png)

*照片由* [*格瑞拉科齐*](https://unsplash.com/photos/uGS_R4r46Cw?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) *上* [*下*](https://unsplash.com/search/photos/connection?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

撰写本文时，考虑将 EclipseLink 作为 JPA 提供者，将 MySql 作为数据库。

实例化 EntityManagerFactory 时，javax . persistence . persistence
createentitymanager factory(String persistence unitname)和重载方法
createentitymanager factory(String persistence unitname，Map properties)提供了两个 API

1.  到物理数据库的连接属性可以在 persistence.xml
    中提供，如下所示:

```
<?xml version=”1.0" encoding=”UTF-8"?><persistence version=”2.0" xmlns=”[http://java.sun.com/xml/ns/persistence](http://java.sun.com/xml/ns/persistence)"
 xmlns:xsi=”[http://www.w3.org/2001/XMLSchema-instance](http://www.w3.org/2001/XMLSchema-instance)" 
 xsi:schemaLocation=”[http://java.sun.com/xml/ns/persistence](http://java.sun.com/xml/ns/persistence) 
 [http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd](http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd)">

 <persistence-unit name=”Eclipselink_JPA” transaction-type=”RESOURCE_LOCAL”>

 <class>com.nitesh.e2e.jpa.entity.TBook</class><properties>
 <property name=”javax.persistence.jdbc.url” value=”jdbc:mysql://localhost:3306/jpadb”/>
 <property name=”javax.persistence.jdbc.user” value=”root”/>
 <property name=”javax.persistence.jdbc.password” value=”password”/>
 <property name=”javax.persistence.jdbc.driver” value=”com.mysql.jdbc.Driver”/>
 <property name=”eclipselink.logging.level” value=”FINE”/>
 <property name=”eclipselink.ddl-generation” value=”create-tables”/>
 </properties>

 </persistence-unit>
</persistence>
```

这里我们连接到运行在 localhost:3306 上的 mysql 数据库

然后，可以通过传递持久性单元名称来实例化 EntityManagerFactory，如下所示:

```
Persistence.createEntityManagerFactory(“Eclipselink_JPA”)
```

2.我们还可以在映射中提供与我们 paas 给重载的 createEntityManagerFactory 方法相同的细节。

```
Map<String, Object> props = new HashMap<String, Object>();
 props.put(“javax.persistence.jdbc.url” , “jdbc:mysql://localhost:3306/jpadb”);
 props.put(“javax.persistence.jdbc.user” , “root”);
 props.put(“javax.persistence.jdbc.password” , “password”);
 props.put(“javax.persistence.jdbc.driver” , “com.mysql.jdbc.Driver”);EntityManagerFactory emfactory = Persistence.createEntityManagerFactory(“Eclipselink_JPA”, props); 
```

3.另一种连接到物理数据库的方法是创建 javax.sql.DataSource 对象。
https://docs . Oracle . com/javase/7/docs/API/javax/SQL/data source . html

通常这是**连接数据库的首选**方式，因为这可以从代码中具体化。

> *数据源接口由驱动厂商实现。*
> 根据文档，这是对 DriverManager 工具的替代，DataSource 对象是获得连接的首选方式。实现数据源接口的对象通常会向基于 Java 命名和目录(JNDI) API 的命名服务注册。在代码中，我们做了一个 JNDI 查找并得到了 DataSource 对象。

然后，这个 DataSource 对象被添加到属性映射中，并用于创建 EntityManagerFactory，如下所示:

```
MysqlDataSource ds = new MysqlDataSource();
 ds.setURL(“jdbc:mysql://localhost:3306/jpadb”);
 ds.setUser(“root”);
 ds.setPassword(“password”);

 Map<String, Object> props = new HashMap<String, Object>();
 props.put(“javax.persistence.nonJtaDataSource”, ds);EntityManagerFactory emfactory = Persistence.createEntityManagerFactory(“Eclipselink_JPA”, props);
```

在上面的代码中，我们创建了 MysqlDataSource 对象，它扩展了 javax.sql.DataSource 对象。
*在现实场景中，当我们创建数据库模式并将其绑定到一个命名的数据源时，该数据源被绑定到 JNDI 注册表。在代码中，我们通过 JNDI 查找来访问这个数据源。*

感谢阅读:)