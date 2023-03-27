# 用嵌入式 Jetty 编写 Web Socket 服务器

> 原文：<https://itnext.io/writing-a-web-socket-server-with-embedded-jetty-46fe9ab1c435?source=collection_archive---------0----------------------->

![](img/e96af51d3917fca1529920f2dff896c6.png)

我们使用 web socket 进行双向通信

也许第一个问题是“为什么我们需要 web socket？”

原因可能因人而异。但我们可以总结为:

1.  我们需要双向交流
2.  我们需要长期的联系，这样我们就可以传递更多的信息

我个人的原因是想了解它是如何工作的，因为[信号](https://signal.org/)使用它。因此，为了更好地理解它，我想写一个简单的基本的工作示例。我会用嵌入式 Jetty 写下来，就像(我猜)Signal 做的那样。我会把它写下来，这样我的同事项目团队就可以遵循同样的理解过程。

> 在继续之前，我想解释一下，我不会自己实现 JSR-356，因为这将是另起炉灶。相反，我使用了 Jetty 中可用的特定 API。

# 用例优先

我们希望拥有满足以下要求的基本消息服务:

1.  用户在做其他事情之前需要登录
2.  用户将能够向其他人发送消息
3.  一旦接收者在线，消息服务器应该将消息传送给接收者

# 设置

首先，我们需要设置环境。这包括:创建 maven 项目，编辑 pom.xml 文件，第一次构建项目，只是为了下载库文件。

1.  使用 Netbeans 创建新的 Maven 项目。在我的例子中，我将其命名为 JettyWebSocket。
2.  编辑 pom.xml 文件。放入`jettyVersion`属性，然后添加 web-socket-server 和 web-socket-api 作为依赖项。pom.xml 文件如下所示:

```
<?xml version="1.0" encoding="UTF-8"?>
<project ae lk" href="http://maven.apache.org/POM/4.0.0" rel="noopener ugc nofollow" target="_blank">http://maven.apache.org/POM/4.0.0" xmlns:xsi="[http://www.w3.org/2001/XMLSchema-instance](http://www.w3.org/2001/XMLSchema-instance)" xsi:schemaLocation="[http://maven.apache.org/POM/4.0.0](http://maven.apache.org/POM/4.0.0) [http://maven.apache.org/xsd/maven-4.0.0.xsd](http://maven.apache.org/xsd/maven-4.0.0.xsd)">
    <modelVersion>4.0.0</modelVersion>
    <groupId>id.amrishodiq</groupId>
    <artifactId>JettyWebSocket</artifactId>
    <version>1.0.1</version>
    <packaging>jar</packaging>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <jettyVersion>**9.4.9.v20180320**</jettyVersion>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.eclipse.jetty.websocket</groupId>
            <artifactId>**websocket-api**</artifactId>
            <version>${jettyVersion}</version>
        </dependency>
        <dependency>
            <groupId>org.eclipse.jetty.websocket</groupId>
            <artifactId>**websocket-server**</artifactId>
            <version>${jettyVersion}</version>
        </dependency>
        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>**gson**</artifactId>
            <version>2.8.2</version>
        </dependency>
    </dependencies>

    <build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.0</version>
            <configuration>
              <source>1.7</source>
              <target>1.7</target>
            </configuration>
        </plugin>
    </plugins>
  </build>
</project>
```

3.使用 Netbeans 构建。菜单运行|构建项目。

# 写代码

该计划将由以下主要部分组成:

1.  将服务器作为主类应用程序
2.  MessagingAdapter，它将在客户端打开连接、接收数据(无论是文本还是二进制)、关闭连接、每个客户端连接发生错误时捕获事件
3.  跟踪每个用户会话的 UserSession 接口
4.  消息传递逻辑我们放置消息传递逻辑的地方
5.  数据模型
6.  将处理所有数据相关操作的存储库

我们将从独立的类开始，在这里是数据模型。希望你熟悉 Java。如果我有`User`类，并且包名为`id.amrishodiq.jettywebsocket.data`，这意味着文件应该是`<project-directory>/id/amrishodiq/jettywebsocket/data/User.java.`

类别用户

```
package id.amrishodiq.jettywebsocket.data;/**
 * User representation of the messaging platform.
 * [@author](http://twitter.com/author) amrishodiq
 */
public class User {
    public String username;
    public String password;public User(String username, String password) {
        this.username = username;
        this.password = password;
    }
}
```

班级信息

```
package id.amrishodiq.jettywebsocket.data;/**
 * Message representation.
 * [@author](http://twitter.com/author) amrishodiq
 */
public class Message {
    public String from;
    public String to;
    public String body;
    public long sent;
}
```

类别数据

```
package id.amrishodiq.jettywebsocket.data;/**
 * Data is a wrapper for transmittable object between client and server.
 * [@author](http://twitter.com/author) amrishodiq
 */
public class Data {
    public static final int AUTHENTICATION_LOGIN = 1;

    public static final int MESSAGING_SEND = 101;

    public int operation;
    public User user;
    public Message message;
    public String session;
}
```

类别库

```
package id.amrishodiq.jettywebsocket.data;import java.util.HashMap;/**
 * Repository responsible for all operation related to data saving or retrieval.
 * [@author](http://twitter.com/author) amrishodiq
 */
public class Repository {
    private static Repository instance;
    public static Repository getInstance() {
        if (instance == null) {
            instance = new Repository();
        }
        return instance;
    }

    private final HashMap<String, User> users = new HashMap<>();
    public final String DUMMY_SESSION = "SOMEVALIDSESSION";

    private Repository() {
        initDummyUser();
    }

    private void initDummyUser() {
        User user = new User("alice", "123456");
        users.put(user.username, user);
        user = new User("bob", "654321");
        users.put(user.username, user);
        user = new User("charlie", "qwerty");
        users.put(user.username, user);
    }

    public boolean isValid(User user) {
        User item = users.get(user.username);
        if (item != null && item.password.equals(user.password)) {
            return true;
        }

        return false;
    }

    public boolean isValid(String session) {
        if (DUMMY_SESSION.equals(session)) {
            return true;
        }
        return false;
    }
}
```

对于数据相关类来说足够了。现在，我们来看主要部分。

界面用户会话

```
package id.amrishodiq.jettywebsocket;import id.amrishodiq.jettywebsocket.data.User;/**
 * UserSession is an interface to track every client connection.
 * [@author](http://twitter.com/author) amrishodiq
 */
public interface UserSession {
    void receiveText(String text) throws Exception;
    void setCurrentUser(User user);
    void disconnect(int status, String reason);
}
```

消息逻辑类

```
package id.amrishodiq.jettywebsocket;import id.amrishodiq.jettywebsocket.data.Repository;
import com.google.gson.Gson;
import id.amrishodiq.jettywebsocket.data.Data;
import id.amrishodiq.jettywebsocket.data.Message;
import id.amrishodiq.jettywebsocket.data.User;
import java.util.HashMap;/**
 * MessagingLogic will be responsible for parsing received data, do 
 * authentication and forward the messages between users.
 * [@author](http://twitter.com/author) amrishodiq
 */
public class MessagingLogic {

    private static MessagingLogic instance;
    public static MessagingLogic getInstance() {
        if (instance == null) {
            instance = new MessagingLogic();
        }
        return instance;
    }

    private HashMap<String, UserSession> userSessions = new HashMap<>();
    private Gson gson = new Gson();

    private MessagingLogic() {
    }

    public void receiveText(UserSession session, String text) {
        try {
            receiveData(session, gson.fromJson(text, Data.class));
        } catch (Throwable t) {
        }
    }

    private void receiveData(UserSession session, Data data) {
        if (data == null) return;

        // for all operation except login, do session checking
        if (data.operation != Data.AUTHENTICATION_LOGIN) {
            if (!validateSession(data.session)) {
                session.disconnect(401, "Invalid session");
            }
        }

        switch (data.operation) {
            case Data.AUTHENTICATION_LOGIN:
                login(session, data.user);
                break;
            case Data.MESSAGING_SEND:
                send(data.message);
                break;
            default:
                session.disconnect(404, "Wrong operation");
        }
    }

    private void login(UserSession session, User user) {
        if (user == null) {
            session.disconnect(401, "Give username and password!");
        }

        if (Repository.getInstance().isValid(user)) {
            userSessions.put(user.username, session);

            session.setCurrentUser(user);

            Data data = new Data();
            data.operation = Data.AUTHENTICATION_LOGIN;
            data.session = Repository.getInstance().DUMMY_SESSION;

            try {
                session.receiveText(gson.toJson(data));
            } catch (Exception ex) {
                ex.printStackTrace();
            }

        } else {
            session.disconnect(401, "Wrong username or password!");
        }
    }

    private boolean validateSession(String session) {
        return Repository.getInstance().isValid(session);
    }

    private void send(Message message) {
        if (isUserOnline(message.to)) {
            System.out.println("User is online, try to send message");
            UserSession userSession = userSessions.get(message.to);
            try {
                userSession.receiveText(gson.toJson(message));
            } catch (Exception ex) {
                // put to offline message
                System.out.println("User is offline");
            }
        } else {
            // todo put to offline message
            System.out.println("User is offline");
        }
    }

    private boolean isUserOnline(String username) {
        return userSessions.containsKey(username);
    }

    public void setOffline(String username) {
        userSessions.remove(username);
        System.err.println("Set "+username+" offline.");
    }

}
```

类别消息适配器

```
package id.amrishodiq.jettywebsocket;import id.amrishodiq.jettywebsocket.data.User;
import org.eclipse.jetty.websocket.api.Session;
import org.eclipse.jetty.websocket.api.WebSocketAdapter;/**
 * MessagingAdapter responsible to handle connection, receiving data, forward 
 * the data to [@see](http://twitter.com/see) id.amrishodiq.jettywebsocket.MessagingLogic and receive 
 * data from [@see](http://twitter.com/see) id.amrishodiq.jettywebsocket.MessagingLogic to be 
 * delivered to recipient.
 * [@author](http://twitter.com/author) amrishodiq
 */
public class MessagingAdapter extends WebSocketAdapter implements UserSession {

    private Session session;
    private User currentUser;

    [@Override](http://twitter.com/Override)
    public void onWebSocketConnect(Session session) {
        super.onWebSocketConnect(session); 

        this.session = session;
    } @Override
    public void onWebSocketClose(int statusCode, String reason) {
        MessagingLogic.getInstance().setOffline(currentUser.username);

        this.session = null;

        System.err.println("Close connection "+statusCode+", "+reason);

        super.onWebSocketClose(statusCode, reason); 
    } @Override
    public void onWebSocketText(String message) {
        super.onWebSocketText(message); 

        MessagingLogic.getInstance().receiveText(this, message);
    } @Override
    public void receiveText(String text) throws Exception {
        if (session != null && session.isOpen()) {
            session.getRemote().sendString(text);
        }
    } @Override
    public void setCurrentUser(User user) {
        this.currentUser = user;
    } @Override
    public void disconnect(int status, String reason) {

        session.close(status, reason);
        disconnect(status, reason);
    }

}
```

类消息服务

```
package id.amrishodiq.jettywebsocket;import org.eclipse.jetty.websocket.servlet.WebSocketServlet;
import org.eclipse.jetty.websocket.servlet.WebSocketServletFactory;/**
 * Well, it's just a servlet declaration.
 * [@author](http://twitter.com/author) amrishodiq
 */
public class MessagingServlet extends WebSocketServlet { @Override
    public void configure(WebSocketServletFactory factory) {
        factory.register(MessagingAdapter.class);
    }

}
```

类别消息服务器

```
package id.amrishodiq.jettywebsocket;import org.eclipse.jetty.server.Server;
import org.eclipse.jetty.server.ServerConnector;
import org.eclipse.jetty.servlet.ServletContextHandler;/**
 * The executable main class.
 * [@author](http://twitter.com/author) amrishodiq
 */
public class MessagingServer {

    private Server server;

    public void setup() {
        server = new Server();
        ServerConnector connector = new ServerConnector(server);
        connector.setPort(8080);
        server.addConnector(connector);

        ServletContextHandler handler = new ServletContextHandler(ServletContextHandler.SESSIONS);
        handler.setContextPath("/");
        server.setHandler(handler);

        handler.addServlet(MessagingServlet.class, "/messaging");
    }

    public void start() throws Exception {
        server.start();
        server.dump(System.err);
        server.join();
    }

    public static void main(String args[]) throws Exception {
        MessagingServer theServer = new MessagingServer();
        theServer.setup();
        theServer.start();
    }
}
```

MessagingServer 是可执行类(参见 main 方法)。我们可以使用 Netbeans 的右键单击并运行来运行该类。然而，我们需要一个可执行文件。jar 文件。目前，我们没有。的。我们拥有的 jar 文件还不可执行。

现在，我们需要修改 pom.xml 以生成 deliverable .jar。

```
<?xml version="1.0" encoding="UTF-8"?>
<project ae lk" href="http://maven.apache.org/POM/4.0.0" rel="noopener ugc nofollow" target="_blank">http://maven.apache.org/POM/4.0.0" xmlns:xsi="[http://www.w3.org/2001/XMLSchema-instance](http://www.w3.org/2001/XMLSchema-instance)" xsi:schemaLocation="[http://maven.apache.org/POM/4.0.0](http://maven.apache.org/POM/4.0.0) [http://maven.apache.org/xsd/maven-4.0.0.xsd](http://maven.apache.org/xsd/maven-4.0.0.xsd)">
    <modelVersion>4.0.0</modelVersion>
    <groupId>id.amrishodiq</groupId>
    <artifactId>JettyWebSocket</artifactId>
    <version>1.0.1</version>
    <packaging>jar</packaging>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        **<jettyVersion>9.4.9.v20180320</jettyVersion>**
    </properties>
    <dependencies>
        **<dependency>
            <groupId>org.eclipse.jetty.websocket</groupId>
            <artifactId>websocket-api</artifactId>
            <version>${jettyVersion}</version>
        </dependency>
        <dependency>
            <groupId>org.eclipse.jetty.websocket</groupId>
            <artifactId>websocket-server</artifactId>
            <version>${jettyVersion}</version>
        </dependency>
        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>2.8.2</version>
        </dependency>**
    </dependencies>

    <build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.3</version>
                <configuration>
                    <createDependencyReducedPom>true</createDependencyReducedPom>
                    <filters>
                        <filter>
                            <artifact>*:*</artifact>
                            <excludes>
                                <exclude>META-INF/*.SF</exclude>
                                <exclude>META-INF/*.DSA</exclude>
                                <exclude>META-INF/*.RSA</exclude>
                            </excludes>
                        </filter>
                    </filters>
                </configuration>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    **<mainClass>id.amrishodiq.jettywebsocket.MessagingServer</mainClass>**
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
        </plugin>
    </plugins>
  </build>
</project>
```

再次构建项目，然后您将有一个可执行文件。`<project-directory>/target`里面的 jar 文件。我的名字是`JettyWebSocket-1.0.1.jar`。

# 功能测试

从`<project-directory>/target`运行`java -jar JettyWebSocket-1.0.1.jar`，您将在端口 8080 上拥有一个正在运行的 web socket 服务器。

用一些 Web Socket 客户端测试它，比如[简单 Web Socket 客户端](https://chrome.google.com/webstore/detail/simple-websocket-client/pfdhoblngboilpfeibdedpjgfnlcodoo?hl=en)，它是谷歌 Chrome 的扩展。连接到`ws://localhost:8080/messaging`。发送此文本:

```
{
  "operation": 1,
  "user": {
    "username": "alice",
    "password": "123456"
  }
}
```

服务器将响应:

```
{"operation":1,"session":"SOMEVALIDSESSION"}
```

这些步骤用于验证消息服务器。一旦你有了会话字符串，就用它来发送消息。发送以下消息:

```
{
  "operation": 101,
  "session": "SOMEVALIDSESSION", 
  "message": {"from":"alice","to":"bob","body":"sembarang aja nih biar seru","sent":1234567}
}
```

MessagingLogic 会发现 bob 是否在线。如果他是，那么这个消息将被转发给 bob。但遗憾的是不在线。

为了测试 bob 何时在线，您需要另一个 web socket 客户机实例。你需要打开新的谷歌浏览器窗口，然后再次打开简单的网络插座插件。打开相同的地址，但使用以下数据登录:

```
{
  "operation": 1,
  "user": {
    "username": "bob",
    "password": "654321"
  }
}
```

然后，使用以下数据向爱丽丝发送消息

```
{
  "operation": 101,
  "session": "SOMEVALIDSESSION",
  "message": {
    "from": "bob",
    "to": "alice",
    "body": "sembarang aja",
    "sent": 1234567
  }
}
```

看，在之前的简单 Web Socket 插件 alice 登录时，她会收到:

```
{"from":"bob","to":"alice","body":"sembarang aja","sent":1234567}
```

尝试愉快。如果你发现任何问题，就告诉我。我们可以讨论一下。