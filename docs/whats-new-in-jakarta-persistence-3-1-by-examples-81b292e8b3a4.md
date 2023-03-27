# Jakarta Persistence 3.1 的新特性示例

> 原文：<https://itnext.io/whats-new-in-jakarta-persistence-3-1-by-examples-81b292e8b3a4?source=collection_archive---------1----------------------->

Jakarta Persistence(又名 JPA) 3.1 带来了一系列改进。

*   `UUID`类现在被视为基本 Java 类型。为了支持`Entity`级的 UUID 型`ID`，JPA 引进了一种新的 UUID 发电机。
*   JPQL 和类型安全标准 API 中添加了几个数字函数和特定于日期/时间的函数。

更多详情请阅读[雅加达持久性 3.1 新特性](https://newsroom.eclipse.org/eclipse-newsletter/2022/march/what%E2%80%99s-new-jakarta-persistence-31)。

接下来，让我们通过编写一些真实的示例代码来探索这些特性。

![](img/29c027320a520928e823994777cb05c4.png)

照片由[&UTM _ medium = referral&UTM _ content = creditCopyText](”<a)">阿莱西奥·林< /a >于<a href = "[https://unsplash.com/s/photos/china-building?UTM _ source = Unsplash&UTM _ medium = referral&UTM _ content = creditCopyText](https://unsplash.com/s/photos/china-building?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)">Unsplash</a>

# 休眠 6.1

通过 [Maven Quickstart 原型](https://maven.apache.org/archetypes/maven-archetype-quickstart/)生成一个简单的 **Java 应用**项目。

```
mvn archetype:generate
    -DarchetypeGroupId=org.apache.maven.archetypes
    -DarchetypeArtifactId=maven-archetype-quickstart
    -DarchetypeVersion=1.4
```

有一些交互式的步骤来指导你设置项目信息，如 groupId，工件，版本等。在这个示例项目中，我们使用`com.example`作为 groupId，使用`demo`作为 artifactId。然后确认并开始生成项目源代码。

完成后，在 Java IDE 中打开项目，例如 IntelliJ IDEA(Community Edition 是免费的)，或者 Eclipse Java/Java EE bundle，或者 NetBeans IDE，或者一个简单的文本编辑器，例如 VS Code。

修改项目根文件夹中的 *pom.xml* ，添加 Hibernate 6.1、JUnit 等。到项目依赖项中，并设置 Maven 编译器插件来使用 Java 17 编译源代码。

最终的 *pom.xml* 如下所示。

```
<?xml version="1.0" encoding="UTF-8"?>
<project 
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
```

```
 <groupId>org.example</groupId>
    <artifactId>hibernate6</artifactId>
    <version>1.0-SNAPSHOT</version>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>jakartaee10-sandbox-parent</artifactId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>..</relativePath>
    </parent> <name>hibernate6</name>
    <description>Jakarta EE 10 Sandbox: Hibernate 6/JPA 3.1 example</description> <properties>
        <maven.compiler.release>17</maven.compiler.release> <!-- requires 6.1.2.Final or higher -->
        <hibernate.version>6.1.4.Final</hibernate.version>
        <h2.version>2.1.214</h2.version> <!-- test deps -->
        <junit-jupiter.version>5.9.1</junit-jupiter.version>
        <assertj-core.version>3.23.1</assertj-core.version> <slf4j.version>2.0.3</slf4j.version>
        <logback.version>1.4.4</logback.version>
    </properties> <dependencies>
        <dependency>
            <groupId>org.hibernate.orm</groupId>
            <artifactId>hibernate-core</artifactId>
            <version>${hibernate.version}</version>
        </dependency>
        <dependency>
            <groupId>jakarta.persistence</groupId>
            <artifactId>jakarta.persistence-api</artifactId>
            <version>3.1.0</version>
        </dependency> <!-- H2 Database -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>${h2.version}</version>
        </dependency> <!-- logging with logback -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jcl-over-slf4j</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
            <version>${logback.version}</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>${logback.version}</version>
        </dependency> <!-- test dependencies -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>${junit-jupiter.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.assertj</groupId>
            <artifactId>assertj-core</artifactId>
            <version>${assertj-core.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.hibernate.orm</groupId>
            <artifactId>hibernate-testing</artifactId>
            <version>${hibernate.version}</version>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>junit</groupId>
                    <artifactId>junit</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies> <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.10.1</version>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.0.0-M7</version>
            </plugin>
        </plugins>
    </build>
</project>
```

注意:要为所有基于特性的项目共享公共资源，创建一个父 POM 以将公共配置集中在一个地方，检查[父 pom.xml 文件](https://github.com/hantsy/jakartaee10-sandbox/blob/master/pom.xml)。

在这个示例项目中，我们使用 H2 嵌入式数据库进行测试。Hibernate 6.1 实现了 Jakarta Persistence 3.1 的特性，但是它在可传递依赖树中包含了一个 Jakarta Persistence 3.0 API。

要使用 Jakarta Persistence 3.1 API，我们必须显式添加`jakarta.persistence:jakarta.persistence-api` 3.1。

在*src/main/resources/META-INF*中，添加一个名为 *persistence.xml* 的新文件。

```
<persistence 
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="https://jakarta.ee/xml/ns/persistence https://jakarta.ee/xml/ns/persistence/persistence_3_1.xsd"
             version="3.1">
```

```
 <persistence-unit name="defaultPU" transaction-type="RESOURCE_LOCAL"> <description>Hibernate test case template Persistence Unit</description>
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider> <exclude-unlisted-classes>false</exclude-unlisted-classes> <properties>
            <property name="hibernate.archive.autodetection" value="class, hbm"/> <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
            <property name="hibernate.connection.driver_class" value="org.h2.Driver"/>
            <property name="hibernate.connection.url" value="jdbc:h2:mem:db1;DB_CLOSE_DELAY=-1"/>
            <property name="hibernate.connection.username" value="sa"/> <property name="hibernate.connection.pool_size" value="5"/> <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.hbm2ddl.auto" value="create-drop"/> <property name="hibernate.max_fetch_depth" value="5"/> <property name="hibernate.cache.region_prefix" value="hibernate.test"/>
            <property name="hibernate.cache.region.factory_class"
                      value="org.hibernate.testing.cache.CachingRegionFactory"/> <!--NOTE: hibernate.jdbc.batch_versioned_data should be set to false when testing with Oracle-->
            <property name="hibernate.jdbc.batch_versioned_data" value="true"/> <property name="jakarta.persistence.validation.mode" value="NONE"/>
            <property name="hibernate.service.allow_crawling" value="false"/>
            <property name="hibernate.session.events.log" value="true"/>
        </properties> </persistence-unit>
</persistence>
```

在这个项目中，我们使用 logback 作为日志框架。在 *src/main/resources* 中，添加一个 *logback.xml* 来配置 logback。

```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
```

```
 <property name="LOGS" value="./logs"/> <appender name="Console"
              class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <Pattern>
                %green(%d{ISO8601}) %highlight(%-5level) [%blue(%t)] %yellow(%C{1.}): %msg%n%throwable
            </Pattern>
        </layout>
    </appender> <appender name="RollingFile"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOGS}/app.log</file>
        <encoder
                class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <Pattern>%d %p %C{1.} [%t] %m%n</Pattern>
        </encoder> <rollingPolicy
                class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- rollover daily and when the file reaches 10 MegaBytes -->
            <fileNamePattern>${LOGS}/archived/app-%d{yyyy-MM-dd}.%i.log
            </fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy
                    class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>10MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
    </appender> <!-- LOG everything at INFO level -->
    <root level="info">
        <appender-ref ref="RollingFile"/>
        <appender-ref ref="Console"/>
    </root> <!-- Debug hibernate SQL,  see: [https://thorben-janssen.com/hibernate-logging-guide/](https://thorben-janssen.com/hibernate-logging-guide/) -->
    <logger name="org.hibernate.SQL" level="DEBUG"/>
    <logger name="org.hibernate.type.descriptor.sql" level="trace"/> <!-- Custom debug level for the application code -->
    <logger name="com.example" level="debug" additivity="false">
        <appender-ref ref="RollingFile"/>
        <appender-ref ref="Console"/>
    </logger>
</configuration>
```

我们将`org.hibernate.SQL`日志记录级别设置为`DEBUG`，将`org.hibernate.type.descriptor.sql`设置为`trace`，这将帮助您在运行时挖掘 Hibernate 生成的 sql。

# UUID 基本类型支持

JPA 3.1 允许使用 UUID 作为基本 Java 类型，特别是它增加了一个 UUID ID 生成器。

创建一个简单的`Entity`。

```
@Entity
public class Person {
    @Id
    @Column(name = "id", nullable = false)
    @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;
    private String name;
    private int age = 30;
```

```
 public Person() {
    } public Person(String name, int age) {
        assert age > 0;
        this.name = name;
        this.age = age;
        this.birthDate = LocalDateTime.now().minusYears(this.age);
    } // getters and setters
    // override equals and hashCode
}
```

一个实体类用一个`@Entity`注释，可选地，你可以指定实体名，并用一个额外的`@Table`注释添加表定义。

这里我们定义了一个 UUID 类型的 ID，并使用了一个 UUID 生成策略。

JPA 要求一个实体应该包含一个无参数的构造函数，如果你声明另一个有几个参数的构造函数，你应该显式声明这个无参数的构造函数。

创建一个简单的 JUnit 测试来验证 UUID 类型是否按预期工作。

```
class PersonUUIDTest {
    private static final Logger log = LoggerFactory.getLogger(PersonUUIDTest.class);
```

```
 private EntityManagerFactory entityManagerFactory; @BeforeEach
    void setUp() {
        entityManagerFactory = Persistence.createEntityManagerFactory("defaultPU");
        var entityManager = entityManagerFactory.createEntityManager();
        entityManager.getTransaction().begin();
        var deleteFromPerson = entityManager.createQuery("DELETE FROM Person").executeUpdate();
        log.debug("Deleted {} persons", deleteFromPerson);
        entityManager.getTransaction().commit();
        entityManager.close();
    } @Test
    @DisplayName("insert person and verify person")
    public void testInsertAndFindPerson() throws Exception {
        var person = new Person("John", 30);
        var entityManager = entityManagerFactory.createEntityManager();
        entityManager.getTransaction().begin();
        entityManager.persist(person);
        entityManager.getTransaction().commit();
        var id = person.getId();
        assertNotNull(id); try {
            var foundPerson = entityManager.find(Person.class, id);
            assertThat(foundPerson.getId()).isNotNull();
            assertThat(foundPerson.getName()).isEqualTo("John");
            assertThat(foundPerson.getAge()).isEqualTo(30);
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    } @AfterEach
    void tearDown() {
        entityManagerFactory.close();
    }
}
```

在`@BeforeEach`方法中，我们将创建一个`EntityManagerFactory`实例。在`@AfterEach`中，我们调用`EntityManagerFactory.close`来释放资源。

在`@BeforeEach`中，我们尝试清理个人数据。

现在在`testInsertAndFindPerson`测试中，我们插入一个新人，然后利用`entityManager.find`找到插入的人。

人员 id 用`@ID`和`@GeneratedValue`标注，当向表中插入人员时，hibernate 会自动生成一个 ID。持久化后，返回的实例将填充生成的 id，它不应为空。

# 数字函数

JPA 3.1 在文字 JPQL 查询和类型安全标准构建器 API 中添加了一组新的数值函数。

在上面的`Person`类中添加一些额外的属性。

```
public class Person{
    private Integer yearsWorked = 2;
    private LocalDateTime birthDate = LocalDateTime.now().minusYears(30);
    private BigDecimal salary = new BigDecimal("12345.678");
    private BigDecimal hourlyRate = new BigDecimal("34.56");
```

```
 // setters and getters
}
```

创建一个新的测试来验证新的数字函数:`ceiling`、`floor`、`round`、`exp`、`ln`、`power`、`sign`。

```
@Test
@DisplayName(">>> test numeric functions")
public void testNumericFunctions() throws Exception {
    var person = new Person("John", 30);
    var entityManager = entityManagerFactory.createEntityManager();
    entityManager.getTransaction().begin();
    entityManager.persist(person);
    entityManager.getTransaction().commit();
    var id = person.getId();
    assertNotNull(id);
```

```
 try {
        var queryString = """
                SELECT p.name as name,
                CEILING(p.salary) as ceiling,
                FLOOR(p.salary) as floor,
                ROUND(p.salary, 1) as round,
                EXP(p.yearsWorked) as exp,
                LN(p.yearsWorked) as ln,
                POWER(p.yearsWorked,2) as power,
                SIGN(p.yearsWorked) as sign
                FROM Person p
                WHERE p.id=:id
                """;
        var query = entityManager.createQuery(queryString);
        query.setParameter("id", id);
        var resultList = query.getResultList();
        log.debug("Result list: {}", resultList);
        resultList.forEach(result -> log.debug("result: {}", result));
    } catch (Exception ex) {
        ex.printStackTrace();
    }
}
```

接下来，让我们看看如何在标准 API 中使用它们。

```
@Test
@DisplayName(">>> test numeric functions")
public void testNumericFunctions() throws Exception {
    var person = new Person("John", 30);
    var entityManager = entityManagerFactory.createEntityManager();
    entityManager.getTransaction().begin();
    entityManager.persist(person);
    entityManager.getTransaction().commit();
    var id = person.getId();
    assertNotNull(id);
```

```
 try {
        // see: [https://hibernate.zulipchat.com/#narrow/stream/132096-hibernate-user/topic/New.20functions.20in.20JPA.203.2E1/near/289429903](https://hibernate.zulipchat.com/#narrow/stream/132096-hibernate-user/topic/New.20functions.20in.20JPA.203.2E1/near/289429903)
        var cb = (HibernateCriteriaBuilder) entityManager.getCriteriaBuilder();
        var query = cb.createTupleQuery();
        var root = query.from(Person.class); query.multiselect(root.get("name"),
                cb.ceiling(root.get("salary")),
                cb.floor(root.get("salary")),
                cb.round(root.get("salary"), 1),
                cb.exp(root.get("yearsWorked")),
                cb.ln(root.get("yearsWorked")),
                // see: [https://hibernate.atlassian.net/browse/HHH-15395](https://hibernate.atlassian.net/browse/HHH-15395)
                cb.power(root.get("yearsWorked"), 2),
                cb.sign(root.get("yearsWorked"))
        );
        query.where(cb.equal(root.get("id"), id));
        var resultList = entityManager.createQuery(query).getResultList();
        log.debug("Result list: {}", resultList); resultList.forEach(result ->
                log.debug(
                        "result: ({},{},{},{},{},{},{},{})",
                        result.get(0, String.class),
                        result.get(1, BigDecimal.class),
                        result.get(2, BigDecimal.class),
                        result.get(3, BigDecimal.class),
                        result.get(4, Double.class),
                        result.get(5, Double.class),
                        result.get(6, Double.class),
                        result.get(7, Integer.class)
                )
        );
    } catch (Exception ex) {
        fail(ex);
    }
}
```

注意，当使用 Hibernate 6.1 时，我们必须将`CriteriaBuilder`转换为`HibernateCriteriaBuilder`来体验新的数字函数。Hibernate 6.2 将与 JPA 3.1 保持一致并解决这个问题。

# 日期时间函数

JPA 3.1 增加了一系列日期时间函数，简化了 Java 8 日期时间 API 的使用。

```
@Test
@DisplayName(">>> test datetime functions")
public void testDateTimeFunctions() throws Exception {
    var person = new Person("John", 30);
    var entityManager = entityManagerFactory.createEntityManager();
    entityManager.getTransaction().begin();
    entityManager.persist(person);
    entityManager.getTransaction().commit();
    var id = person.getId();
    assertNotNull(id);
```

```
 try {
        var queryString = """
                SELECT p.name as name,
                LOCAL TIME as localTime,
                LOCAL DATETIME as localDateTime,
                LOCAL DATE as localDate
                FROM Person p
                """; var query = entityManager.createQuery(queryString);
        var resultList = query.getResultList();
        log.debug("Result list: {}", resultList);
        resultList.forEach(result -> log.debug("result: {}", result));
    } catch (Exception ex) {
        ex.printStackTrace();
    }
}
```

`LOCAL TIME`、`LOCAL DATETIME`、`LOCAL DATE`查询结果将直接作为 Java 8 `LocalTime`、`LocalDateTime`、`LocalDate`处理。

让我们看看在 CriteriaBuilder APIs 中的用法。

```
@Test
@DisplayName(">>> test datetime functions")
public void testDateTimeFunctions() throws Exception {
    var person = new Person("John", 30);
    var entityManager = entityManagerFactory.createEntityManager();
    entityManager.getTransaction().begin();
    entityManager.persist(person);
    entityManager.getTransaction().commit();
    var id = person.getId();
    assertNotNull(id);
```

```
 try {
        var cb = (HibernateCriteriaBuilder) entityManager.getCriteriaBuilder();
        var query = cb.createTupleQuery();
        var root = query.from(Person.class); query.multiselect(root.get("name"),
                cb.localTime(),
                cb.localDateTime(),
                cb.localDate()
        );
        query.where(cb.equal(root.get("id"), id)); var resultList = entityManager.createQuery(query).getResultList();
        log.debug("Result list: {}", resultList);
        resultList.forEach(result ->
                log.debug(
                        "result: ({},{},{},{})",
                        result.get(0, String.class),
                        result.get(1, LocalTime.class),
                        result.get(2, LocalDateTime.class),
                        result.get(3, LocalDate.class)
                )
        );
    } catch (Exception ex) {
        fail(ex);
    }
}
```

# `EXTRACT`功能

JPA 3.1 引入了一个`extract`函数来解码来自日期时间值的片段。

```
@Test
@DisplayName(">>> test `EXTRACT` functions")
public void testExtractFunctions() throws Exception {
    var person = new Person("John", 30);
    var entityManager = entityManagerFactory.createEntityManager();
    entityManager.getTransaction().begin();
    entityManager.persist(person);
    entityManager.getTransaction().commit();
    var id = person.getId();
    assertNotNull(id);
```

```
 try {
        var queryString = """
                SELECT p.name as name,
                EXTRACT(YEAR FROM p.birthDate) as year,
                EXTRACT(QUARTER FROM p.birthDate) as quarter,
                EXTRACT(MONTH FROM p.birthDate) as month,
                EXTRACT(WEEK FROM p.birthDate) as week,
                EXTRACT(DAY FROM p.birthDate) as day,
                EXTRACT(HOUR FROM p.birthDate) as hour,
                EXTRACT(MINUTE FROM p.birthDate) as minute,
                EXTRACT(SECOND FROM p.birthDate) as second
                FROM Person p
                """;
        var query = entityManager.createQuery(queryString); var resultList = query.getResultList();
        log.debug("Result list: {}", resultList);
        resultList.forEach(result -> log.debug("result: {}", result));
    } catch (Exception ex) {
        ex.printStackTrace();
    }
}
```

使用新的`extract`函数，我们可以在 JPQL 查询中从 Java 8 DateTime 类型属性中读取`year`、`quarter`、`month`、`week`、`day`、`hour`、`minute`、`second`值。

请注意，在 CriteriaBuilder APIs 中没有映射的提取函数，有关更多详细信息，请查看问题:[jakartaee/persistence # 356](https://github.com/jakartaee/persistence/pull/356)

# JakartaEE 运行时

接下来让我们一起去 Jakarta EE 10 兼容产品体验 JPA 3.1 的新特性。

首先，我们将准备一个 Jakarta EE 10 web 应用程序。

通过 [Maven Webapp 原型](https://maven.apache.org/archetypes/maven-archetype-webapp/)简单地生成一个 web 应用程序框架。

```
mvn archetype:generate
    -DarchetypeGroupId=org.apache.maven.archetypes
    -DarchetypeArtifactId=maven-archetype-webapp
    -DarchetypeVersion=1.4
```

然后将 Jakarta EE 10 依赖项添加到项目 pom.xml 中，我们来看看修改后的 pom.xml。

```
<project  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>jpa-examples</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>jakartaee10-sandbox-parent</artifactId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>..</relativePath>
    </parent>
```

```
 <name>jpa-examples</name>
    <description>Jakarta EE 10 Sandbox: Persistence 3.1 Examples</description>
    <properties>
    </properties> <dependencies>
        <dependency>
            <groupId>jakarta.platform</groupId>
            <artifactId>jakarta.jakartaee-api</artifactId>
        </dependency>
        <dependency>
            <groupId>org.eclipse.persistence</groupId>
            <artifactId>org.eclipse.persistence.jpa.modelgen.processor</artifactId>
            <version>4.0.0</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.jboss.arquillian.junit5</groupId>
            <artifactId>arquillian-junit5-container</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- see: [https://github.com/arquillian/arquillian-core/issues/248](https://github.com/arquillian/arquillian-core/issues/248) -->
        <!-- and [https://github.com/arquillian/arquillian-core/pull/246/files](https://github.com/arquillian/arquillian-core/pull/246/files) -->
        <dependency>
            <groupId>org.jboss.arquillian.protocol</groupId>
            <artifactId>arquillian-protocol-servlet-jakarta</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

在上面的 pom.xml 中，我们还在测试范围中添加了 JUnit 5 和 [Arquillian](https://arquillian.org) 相关的依赖项。通过容器特定的 Aquillian 适配器，我们可以在 Jakarta EE 应用服务器上运行测试。

在这个项目中，我们重用了在 Hibernate 小节中引入的`Person`实体。

现在让我们转到持久性配置。在*src/main/resources/META-INFO*文件夹中创建一个 *persistence.xml* 。

```
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="3.0" 
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="https://jakarta.ee/xml/ns/persistence https://jakarta.ee/xml/ns/persistence/persistence_3_0.xsd">
    <persistence-unit name="defaultPU" transaction-type="JTA">
        <jta-data-source>java:comp/DefaultDataSource</jta-data-source>
        <exclude-unlisted-classes>false</exclude-unlisted-classes>
        <properties>
            <property name="jakarta.persistence.schema-generation.database.action" value="drop-and-create"/>
```

```
 <!-- for  Glassfish/Payara/EclipseLink -->
            <property name="eclipselink.logging.level.sql" value="FINE"/>
            <property name="eclipselink.logging.level" value="FINE"/>
            <property name="eclipselink.logging.parameters" value="true"/> <!-- for WildFly/Hibernate -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
        </properties>
    </persistence-unit>
</persistence>
```

该配置与我们在 Hibernate 部分介绍的配置略有不同。

*   在容器环境中，我们希望选择`JTA`作为事务类型。
*   我们不设置数据库连接信息，而是配置一个内置的数据源。`java:comp/DefaultDataSource`是所有 Jakarta EE 兼容产品的默认数据源。

# 创建 Jakarta EE 示例应用程序

为了与我们的后端数据库进行交互，我们将创建一个简单完整的 JAXRS 应用程序，包括:

*   一个 EJB `@Stateless` bean 从数据库中读取数据
*   并通过简单的 JAXRS 资源公开数据

好，让我们创建用`@Stateless`标注的`PersonRepository`类。在这个类中，注入一个带有注释`@PersistenceContext`的`EntityManager` bean，并添加一个新方法`getAllResource`来执行 JPQL 查询以检索所有人员。

```
@Stateless
public class PersonRepository {
```

```
 @PersistenceContext
    EntityManager entityManager; public List<Person> getAllPersons() {
        return entityManager.createQuery("select p from Person p", Person.class)
                .getResultList();
    }
}
```

接下来，创建一个`PersonResource`来向客户端公开人员。

```
@RequestScoped
@Path("/persons")
public class PersonResource {
```

```
 @Inject
    PersonRepository personRepository; @Path("")
    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public Response allPersons() {
        var data = personRepository.getAllPersons();
        return Response.ok(data).build();
    }
}
```

用`RequestScoped`注释了`PersonResource`，它是一个 CDI bean，类上的`@Path`定义了这个类中所有子资源的根路径。当 HTTP 客户端请求与 HTTP GET 方法匹配，URI 为`/persons`且 HTTP 头接受与`application/json`兼容时，`allPersons`将以 JSON 格式生成所有人员给客户端。

要激活 JAXRS 特性，创建一个类来扩展 JAXRS `Application`，添加`@ApplicationPath`来指定所有 JAXRS 资源的根上下文路径。

```
@ApplicationPath("/rest")
public class RestActivator extends Application {
}
```

让我们创建一个 bean，在应用程序启动时添加一些样本数据。

```
@Startup
@Singleton
public class DataInitializer {
```

```
 @PersistenceContext
    EntityManager entityManager; @PostConstruct
    public void init() {
        List
                .of(
                        new Person("Jack", 20),
                        new Person("Rose", 18)
                )
                .forEach(entityManager::persist);
    }
}
```

# 部署到雅加达 EE 集装箱

将应用程序构建并打包到 war 档案中。打开终端，切换到项目根文件夹，并执行以下命令。

```
mvn clean package -DskipTests -D"maven.test.skip=true"
```

完成后，在路径*target/JPA-examples . war*中有 war 包准备好了。

## 玻璃鱼 7.0

1.  下载[最新的 GlassFish 7.0](https://github.com/eclipse-ee4j/glassfish/releases) ，将文件解压到一个位置，例如 D:\glassfish7，标记为 *GlassFish_install* 。
2.  要启动 GlassFish 和 Derby，打开一个终端，输入 *GlassFish_install/bin* ，运行`asadmin start-database`和`asadmin start-domain domain1`。
3.  将上述 war 包复制到*Glassfish _ install/Glassfish/domains/domain 1/auto deploy*文件夹中。
4.  打开*GlassFish _ install/GlassFish/domains/domain 1/logs/server . log*，等待部署完成。
5.  打开另一个终端窗口，执行`curl http://localhost:8080/jpa-examples/rest/persons`。您将在控制台中得到以下响应。

*   [{ "年龄":18，"出生日期":" 2004–11–06t 14:54:05.4504678 "，"性别":"男"，" hourlyRate":34.56，" id ":" d 8552d 71-ff7f-4650-b5 A0-ce 1c 5 FB 3 Fe 0 b "，"姓名":" Rose "，"薪水":12345.678，"工作年限":2}，{ "年龄":20，"出生日期:" "

1.  要停止 GlassFish 和 Derby，运行`asadmin stop-database`和`asadmin stop-domain domain1`

## 野花预览 27

1.  下载最新的 [WildFly 预览](https://wildfly.org)，解压文件到一个位置，例如:D:\ wild fly-Preview-27 . 0 . 0 . beta 1，标记为 *WildFly_install* 。
2.  打开一个终端，输入*wildly _ install/bin*，运行`standalone`以默认的独立概要配置启动 wildly。
3.  将构建好的 war 复制到*wildly _ install/standalone/deployments*。
4.  等待部署进度完成，您可以使用 GlassFish 部分中的 curl 来验证应用程序。
5.  在原 WildFly 启动控制台中发送一个`CTLR+C`组合键来停止 WildFly。

# 通过 Maven 插件部署应用程序

## 通过 Cargo 插件部署到 GlassFish

GlassFish 项目不包括管理 GlassFish 服务器的官方 Maven 插件。有一个名为`cargo-maven3-plugin`的 Maven 插件，可以用来管理所有流行的 Jakarta EE 应用服务器和 web 服务器。

添加下面的`profile`部分来使用 cargo 插件管理 GlassFish 服务器的生命周期。

```
<profile>
    <id>glassfish</id>
    <activation>
        <activeByDefault>true</activeByDefault>
    </activation>
    <properties>
        <cargo.zipUrlInstaller.downloadDir>${project.basedir}/../installs</cargo.zipUrlInstaller.downloadDir>
    </properties>
    <build>
        <plugins>
            <plugin>
                <groupId>org.codehaus.cargo</groupId>
                <artifactId>cargo-maven3-plugin</artifactId>
                <configuration>
                    <container>
                        <containerId>glassfish7x</containerId>
                        <!-- <artifactInstaller>
                            <groupId>org.glassfish.main.distributions</groupId>
                            <artifactId>glassfish</artifactId>
                            <version>${glassfish.version}</version>
                        </artifactInstaller> -->
                        <zipUrlInstaller>
                            <url>https://github.com/eclipse-ee4j/glassfish/releases/download/${glassfish.version}/glassfish-${glassfish.version}.zip</url>
                            <downloadDir>${cargo.zipUrlInstaller.downloadDir}</downloadDir>
                        </zipUrlInstaller>
                    </container>
                    <configuration>
                        <!-- the configuration used to deploy -->
                        <home>${project.build.directory}/glassfish7x-home</home>
                        <properties>
                            <cargo.remote.password></cargo.remote.password>
                            <cargo.glassfish.removeDefaultDatasource>true</cargo.glassfish.removeDefaultDatasource>
                        </properties>
                        <datasources>
                            <datasource>
                                <driverClass>org.apache.derby.jdbc.EmbeddedDriver</driverClass>
                                <url>jdbc:derby:derbyDB;create=true</url>
                                <jndiName>jdbc/__default</jndiName>
                                <username>APP</username>
                                <password>nonemptypassword</password>
                            </datasource>
                        </datasources>
                    </configuration>
                </configuration>
            </plugin>
        </plugins>
    </build>
</profile>
```

与 NetBeans IDE 或带有 GlassFish Pack 的 Eclipse IDE 中的方法不同，启动 GlassFish 将同时启动内置的 Derby。Cargo 没有像预期的那样启动内置的 Derby，要在我们的项目中使用默认数据源，清除默认数据源并添加一个新的基于嵌入式 Derby 的默认数据源。

运行以下命令。它将编译项目源代码并将应用程序打包到 war 档案中，然后启动托管的 GlassFish 服务器(使用新的`cargo-domain`)，然后将包部署到这个正在运行的服务器中。

```
mvn clean package cargo:run -DskipTests -Dmaven.test.skip=true
```

注意，当您第一次运行这个命令时，它将花费一些时间来下载 GlassFish 再发行版的副本，并将文件解压缩到 build 文件夹中。

在另一个终端窗口中，执行`curl http://localhost:8080/jpa-examples/rest/persons`来验证端点。

要停止服务器，只需在原来的 GlassFish 运行控制台中发送一个`CTRL+C`。

## 通过 WildFly 插件部署到 WildFly

WildFly 项目本身提供了一个官方的 WildFly Maven 插件，我们将在一个新的 Maven 配置文件中配置它。

> *货 maven 插件也支持野，勾选* [*货野 docs*](https://codehaus-cargo.github.io/cargo/WildFly+27.x.html) *。*

```
<profile>
    <id>wildfly</id>
    <properties>
        <!-- Wildfly server -->
        <wildfly.artifactId>wildfly-preview-dist</wildfly.artifactId>
        <jboss-as.home>${project.build.directory}/wildfly-preview-${wildfly.version}</jboss-as.home>
    </properties>
    <build>
        <plugins>
```

```
 <!-- unpack a copy of WildFly-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <version>${maven-dependency-plugin.version}</version>
                <executions>
                    <execution>
                        <id>unpack</id>
                        <phase>process-classes</phase>
                        <goals>
                            <goal>unpack</goal>
                        </goals>
                        <configuration>
                            <artifactItems>
                                <artifactItem>
                                    <groupId>org.wildfly</groupId>
                                    <artifactId>${wildfly.artifactId}</artifactId>
                                    <version>${wildfly.version}</version>
                                    <type>zip</type>
                                    <overWrite>false</overWrite>
                                    <outputDirectory>${project.build.directory}</outputDirectory>
                                </artifactItem>
                            </artifactItems>
                        </configuration>
                    </execution>
                </executions>
            </plugin> <!-- The WildFly plugin deploys your war to a local running WildFly container -->
            <!-- To use, run: mvn package wildfly:deploy -->
            <!-- For Jakarta EE 9, use `wildfly-preview-dist` as artifactId instead to start and deploy applications-->
            <!-- Run: mvn clean wildfly:run -PWildfly -Dwildfly.artifactId=wildfly-preview-dist -Dwildfly.version=22.0.0.Alpha1 -->
            <!-- or set the `jboss-as.home` to run: mvn clean wildfly:run -PWildfly -Djboss-as.home=D:\appsvr\wildfly-preview-22.0.0.Alpha1-->
            <plugin>
                <groupId>org.wildfly.plugins</groupId>
                <artifactId>wildfly-maven-plugin</artifactId>
                <version>${wildfly-maven-plugin.version}</version>
            </plugin>
        </plugins>
    </build>
    <repositories>
        <repository>
            <id>opensaml</id>
            <url>https://build.shibboleth.net/nexus/content/repositories/releases/</url>
        </repository>
    </repositories>
</profile>
```

有了 WildFly 插件，我们可以将应用程序部署到嵌入式 WildFly、托管 WildFly 服务器或远程运行 WildFly 服务器中。

```
mvn clean wildfly:run -Pwildfly -DskipTests -Dmaven.test.skip=true
```

默认情况下，如果我们不设置`jboss-as.home`或远程主机连接信息，它将引导一个嵌入式 WildFly，并使用嵌入式服务器运行应用程序。

这里我们配置 Maven dependency 插件来下载 wildly 的副本，将文件解压到项目构建目录，并设置一个`jboss-as.home`属性，值是 wildly 的位置。WildFly 插件将管理整个 WildFly 生命周期-启动 WildFly 服务器，将应用程序部署到正在运行的服务器中，(使用`CTRL+C`热键)停止服务器。

# 测试 JPA 特性

这里我假设你以前熟悉 [JUnit](https://www.junit.org) 和 [Arquillian](https://arquillian.org) 。

> *对于 Arqullian 框架的新开发者，请阅读官方*[*Arqullian 指南*](https://arquillian.org/guides) *开始你的第一步。请注意，这些教程有多种语言版本，包括简体中文。*
> 
> *进入我的* [*雅加达 EE 8 入门样板项目*](https://github.com/hantsy/jakartaee8-starter-boilerplate) *和* [*雅加达 EE 9 入门样板项目*](https://github.com/hantsy/jakartaee9-starter-boilerplate) *更新你的基础知识。*

从 Jakarta EE 9 开始，它使用新的`jakarta`名称空间，Arquillian 1.7.0.x 开始支持这些变化。

在接下来的步骤中，我们将配置一个托管的 GlassFish Arquillian 适配器来运行测试代码。

```
<profile>
    <id>arq-glassfish-managed</id>
    <properties>
        <skip.unit.tests>true</skip.unit.tests>
        <skip.integration.tests>false</skip.integration.tests>
    </properties>
    <dependencies>
        <!-- Jersey -->
        <dependency>
            <groupId>org.glassfish.jersey.media</groupId>
            <artifactId>jersey-media-sse</artifactId>
            <version>${jersey.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.glassfish.jersey.media</groupId>
            <artifactId>jersey-media-json-binding</artifactId>
            <version>${jersey.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.glassfish.jersey.inject</groupId>
            <artifactId>jersey-hk2</artifactId>
            <version>${jersey.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.glassfish.jersey.core</groupId>
            <artifactId>jersey-client</artifactId>
            <version>${jersey.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.github.hantsy.arquillian-container-glassfish-jakarta</groupId>
            <artifactId>arquillian-glassfish-managed-jakarta</artifactId>
            <version>${arquillian-glassfish-jakarta.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
        <testResources>
            <testResource>
                <directory>src/test/resources</directory>
            </testResource>
            <testResource>
                <directory>src/test/arq-glassfish-managed</directory>
            </testResource>
        </testResources>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <version>${maven-dependency-plugin.version}</version>
                <executions>
                    <execution>
                        <id>unpack</id>
                        <phase>pre-integration-test</phase>
                        <goals>
                            <goal>unpack</goal>
                        </goals>
                        <configuration>
                            <artifactItems>
                                <artifactItem>
                                    <groupId>org.glassfish.main.distributions</groupId>
                                    <artifactId>glassfish</artifactId>
                                    <version>${glassfish.version}</version>
                                    <type>zip</type>
                                    <overWrite>false</overWrite>
                                    <outputDirectory>${project.build.directory}</outputDirectory>
                                </artifactItem>
                            </artifactItems>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-failsafe-plugin</artifactId>
                <version>${maven-failsafe-plugin.version}</version>
                <configuration>
                    <environmentVariables>
                        <GLASSFISH_HOME>${project.build.directory}/glassfish7</GLASSFISH_HOME>
                    </environmentVariables>
                </configuration>
            </plugin>
        </plugins>
    </build>
</profile>
```

在上面的配置中，我们添加了`com.github.hantsy.arquillian-container-glassfish-jakarta:arquillian-glassfish-managed-jakarta`，这是我的官方 [Arquillian 容器 GlassFish 项目](https://github.com/arquillian/arquillian-container-glassfish6)的[叉子](https://github.com/hantsy/arquillian-container-glassfish-jakarta)。

我们在`pre-integration-test`阶段预发布了最新的 GlassFish 7.0 的副本。阿奎利亚测试将在`integretion-test`阶段进行。

让我们创建一个简单的 Arquillian 测试来验证 JPA 3.1 中的 UUID 基本类型特性。

```
@ExtendWith(ArquillianExtension.class)
public class UUIDStrategyTest {
```

```
 private final static Logger LOGGER = Logger.getLogger(UUIDStrategyTest.class.getName()); @Deployment
    public static WebArchive createDeployment() {
        return ShrinkWrap.create(WebArchive.class)
                .addClasses(Person.class, Gender.class)
                .addAsResource("test-persistence.xml", "META-INF/persistence.xml")
                .addAsWebInfResource(EmptyAsset.INSTANCE, "beans.xml");
    } @PersistenceContext
    private EntityManager em; @Inject
    UserTransaction ux; @BeforeEach
    public void before() throws Exception {
        startTx();
    } private void startTx() throws Exception {
        ux.begin();
        em.joinTransaction();
    } @AfterEach
    public void after() throws Exception {
        endTx();
    } private void endTx() throws Exception {
        try {
            if( ux.getStatus() == Status.STATUS_ACTIVE ) {
                ux.commit();
            }
        } catch (Exception e) {
            ux.rollback();
        }
    } @Test
    public void testPersistingPersons() throws Exception {
        final Person person = new Person();
        person.setName("Hantsy Bai");
        em.persist(person);
        endTx(); startTx();
        final Person foundPerson = em.find(Person.class, person.getId());
        assertNotNull(foundPerson.getId());
        LOGGER.log(Level.INFO, "Found person: {0}", foundPerson);
    }
}
```

测试类上的`@ExtendWith(ArquillianExtension.class)`注释支持 Arquillian 测试生命周期。

`@Deployment`带注释的静态方法定义了将被打包到测试档案中并被部署到被管理的 GlassFish 服务器中的资源。使用包膜很容易创建精细的部署单元。

您可以像在简单的 CDI bean 中一样，在 Arquillian 测试中注入`EntityManager`和`UserTransaction`bean。

在这个测试类中，我们设置了`@BeforeEach`和`@AfterEach`钩子来开始一个事务和结束事务。

测试方法`testPersistingPersons`看起来和普通的 JUnit 测试没有什么区别。首先，我们持久化一个 person 实体，并提交事务以确保它将按预期刷新到数据库中。然后执行一个简单的 JPA 查询来验证持久化数据。

执行以下命令来运行测试。

```
mvn clean verify -Parq-glassfish-managed
```

类似地，创建一个测试来验证 Jakarta rumtimes 中新的数字函数和日期时间函数。

```
@ExtendWith(ArquillianExtension.class)
public class JPQLFunctionsTest {
```

```
 private final static Logger LOGGER = Logger.getLogger(JPQLFunctionsTest.class.getName()); @Deployment
    public static WebArchive createDeployment() {
        return ShrinkWrap.create(WebArchive.class)
                .addClasses(Person.class, Gender.class)
                .addAsResource("test-persistence.xml", "META-INF/persistence.xml")
                .addAsWebInfResource(EmptyAsset.INSTANCE, "beans.xml");
    } @PersistenceContext
    private EntityManager em; @Inject
    UserTransaction ux; @BeforeEach
    public void before() throws Exception {
        clearPersons();
        startTx();
    } private void clearPersons() throws Exception {
        startTx();
        var builder = em.getCriteriaBuilder();
        var deletePersonQuery = builder.createCriteriaDelete(Person.class);
        var deletedPersons = em.createQuery(deletePersonQuery).executeUpdate();
        LOGGER.log(Level.INFO, "Deleted {0} persons", deletedPersons);
        endTx();
    } private void startTx() throws Exception {
        ux.begin();
        em.joinTransaction();
    } @AfterEach
    public void after() throws Exception {
        endTx();
    } private void endTx() throws Exception {
        LOGGER.log(Level.INFO, "Transaction status: {0}", ux.getStatus());
        try {
            if (ux.getStatus() == Status.STATUS_ACTIVE) {
                ux.commit();
            }
        } catch (Exception e) {
            ux.rollback();
        }
    } @Test
    @DisplayName(">>> test numeric functions")
    public void testNumericFunctions() throws Exception {
        var person = new Person("John", 30);
        em.persist(person);
        var id = person.getId();
        assertNotNull(id);
        endTx(); startTx();
        try {
            var queryString = """
                    SELECT p.name,
                    CEILING(p.salary),
                    FLOOR(p.salary),
                    ROUND(p.salary, 1),
                    EXP(p.yearsWorked),
                    LN(p.yearsWorked),
                    POWER(p.yearsWorked,2),
                    SIGN(p.yearsWorked)
                    FROM Person p
                    WHERE p.id=:id
                    """;
            var query = em.createQuery(queryString); query.setParameter("id", id);
            // for EclipseLinks
            query.setHint(QueryHints.RESULT_TYPE, ResultType.Map);
            List<Map<String, Object>> resultList = query.getResultList();
            LOGGER.log(Level.INFO, "result size:{0}", resultList.size());
            resultList.forEach(data -> {
                data.forEach((k, v) -> LOGGER.log(Level.INFO, "field:{0}, value: {1}", new Object[]{k, v}));
            });
        } catch (Exception ex) {
            fail(ex);
        }
    } @Test
    @DisplayName(">>> test nen datetime functions")
    public void testDateTimeFunctions() throws Exception {
        var person = new Person("John", 30);
        em.persist(person);
        assertNotNull(person.getId());
        endTx(); startTx();
        try {
            var queryString = """
                    SELECT p.name as name,
                    LOCAL TIME as localTime,
                    LOCAL DATETIME as localDateTime,
                    LOCAL DATE as localDate
                    FROM Person p
                    """;
            // for EclipseLinks
            var query = em.createQuery(queryString);
            // for EclipseLinks
            query.setHint(QueryHints.RESULT_TYPE, ResultType.Map);
            List<Map<String, Object>> resultList = query.getResultList();
            LOGGER.log(Level.INFO, "result size:{0}", resultList.size());
            resultList.forEach(data -> {
                data.forEach((k, v) -> LOGGER.log(Level.INFO, "field:{0}, value: {1}", new Object[]{k, v}));
            });
        } catch (Exception ex) {
            fail(ex);
        }
    } @Test
    @DisplayName(">>> test `EXTRACT` functions")
    public void testExtractFunctions() throws Exception {
        var person = new Person("John", 30);
        em.persist(person);
        assertNotNull(person.getId());
        endTx(); startTx();
        try {
            var queryString = """
                    SELECT p.name as name,
                    EXTRACT(YEAR FROM p.birthDate) as year,
                    EXTRACT(QUARTER FROM p.birthDate) as quarter,
                    EXTRACT(MONTH FROM p.birthDate) as month,
                    EXTRACT(WEEK FROM p.birthDate) as week,
                    EXTRACT(DAY FROM p.birthDate) as day,
                    EXTRACT(HOUR FROM p.birthDate) as hour,
                    EXTRACT(MINUTE FROM p.birthDate) as minute,
                    EXTRACT(SECOND FROM p.birthDate) as second
                    FROM Person p
                    """;
            var query = em.createQuery(queryString);
            // for EclipseLinks
            query.setHint(QueryHints.RESULT_TYPE, ResultType.Map);
            List<Map<String, Object>> resultList = query.getResultList();
            LOGGER.log(Level.INFO, "result size:{0}", resultList.size());
            resultList.forEach(data -> {
                data.forEach((k, v) -> LOGGER.log(Level.INFO, "field:{0}, value: {1}", new Object[]{k, v}));
            });
        } catch (Exception ex) {
            fail(ex);
        }
    }
}
```

或者，创建一个测试来验证标准 API。

```
@ExtendWith(ArquillianExtension.class)
public class JPQLCriteriaBuilderTest {
```

```
 private final static Logger LOGGER = Logger.getLogger(JPQLCriteriaBuilderTest.class.getName()); @Deployment
    public static WebArchive createDeployment() {
        return ShrinkWrap.create(WebArchive.class)
                .addClasses(Person.class, Gender.class)
                .addAsResource("test-persistence.xml", "META-INF/persistence.xml")
                .addAsWebInfResource(EmptyAsset.INSTANCE, "beans.xml");
    } @PersistenceContext
    private EntityManager em; @Inject
    UserTransaction ux; @BeforeEach
    public void before() throws Exception {
        clearPersons();
        startTx();
    } private void clearPersons() throws Exception {
        startTx();
        var builder = em.getCriteriaBuilder();
        var deletePersonQuery = builder.createCriteriaDelete(Person.class);
        var deletedPersons = em.createQuery(deletePersonQuery).executeUpdate();
        LOGGER.log(Level.INFO, "Deleted {0} persons", deletedPersons);
        endTx();
    } private void startTx() throws Exception {
        ux.begin();
        em.joinTransaction();
    } @AfterEach
    public void after() throws Exception {
        endTx();
    } private void endTx() throws Exception {
        LOGGER.log(Level.INFO, "Transaction status: {0}", ux.getStatus());
        try {
            if (ux.getStatus() == Status.STATUS_ACTIVE) {
                ux.commit();
            }
        } catch (Exception e) {
            ux.rollback();
        }
    } @Test
    @DisplayName(">>> test numeric functions")
    public void testNumericFunctions() throws Exception {
        var person = new Person("John", 30);
        em.persist(person);
        var id = person.getId();
        assertNotNull(id);
        endTx(); startTx();
        try {
            var cb = em.getCriteriaBuilder();
            var query = cb.createTupleQuery();
            var root = query.from(Person.class); query.multiselect(root.get("name"),
                    cb.ceiling(root.get("salary")),
                    cb.floor(root.get("salary")),
                    cb.round(root.get("salary"), 1),
                    cb.exp(root.get("yearsWorked")),
                    cb.ln(root.get("yearsWorked")),
                    cb.power(root.get("yearsWorked"), 2),
                    cb.sign(root.get("yearsWorked"))
            );
            query.where(cb.equal(root.get("id"), id)); var resultList = em.createQuery(query).getResultList();
            LOGGER.log(Level.INFO, "result size:{0}", resultList.size());
            resultList.forEach(result ->
                    LOGGER.log(
                            Level.INFO,
                            // see: [https://github.com/eclipse-ee4j/eclipselink/issues/1593](https://github.com/eclipse-ee4j/eclipselink/issues/1593)
                            // John,12,345,12,345,12,345,7.389,0.693,4,1
                            "tuple data :{0},{1},{2},{3},{4},{5},{6},{7}",
                            new Object[]{
                                    result.get(0, String.class),
                                    result.get(1, BigDecimal.class), // it should return BigDecimal
                                    result.get(2, BigDecimal.class),
                                    result.get(3, BigDecimal.class),
                                    result.get(4, Double.class),
                                    result.get(5, Double.class),
                                    result.get(6, Double.class),
                                    result.get(7, Integer.class)
                            }
                    )
            );
        } catch (Exception ex) {
            fail(ex);
        }
    } @Test
    @DisplayName(">>> test nen datetime functions")
    public void testDateTimeFunctions() throws Exception {
        var person = new Person("John", 30);
        em.persist(person);
        var id = person.getId();
        assertNotNull(id);
        endTx(); startTx();
        try {
            var cb = em.getCriteriaBuilder();
            var query = cb.createTupleQuery();
            var root = query.from(Person.class); query.multiselect(root.get("name"),
                    cb.localTime(),
                    cb.localDateTime(),
                    cb.localDate()
            );
            query.where(cb.equal(root.get("id"), id)); var resultList = em.createQuery(query).getResultList();
            LOGGER.log(Level.INFO, "result size:{0}", resultList.size());
            resultList.forEach(data ->
                    LOGGER.log(
                            Level.INFO,
                            "tuple data :{0},{1},{2},{3}",
                            new Object[]{
                                    data.get(0, String.class),
                                    data.get(1, java.time.LocalTime.class),
                                    data.get(2, java.time.LocalDateTime.class),
                                    data.get(3, java.time.LocalDate.class)
                            }
                    )
            );
        } catch (Exception ex) {
            fail(ex);
        }
    }
}
```

但遗憾的是，GlassFish 7.0.0 中有一个 bug——M9 将无法通过测试`JPQLFunctionsTest`，更多详情请查看 Github 问题 [GlassFish #24120](https://github.com/eclipse-ee4j/glassfish/issues/24120) 。

从我的 github 中查看 [Hibernate](https://github.com/hantsy/jakartaee10-sandbox/tree/master/hibernate) 和 [Jakarta Persistence](https://github.com/hantsy/jakartaee10-sandbox/tree/master/jpa) 的示例代码。