# 房间持久性库——在数据库操作后，使用可变数据观察来更新 UI

> 原文：<https://itnext.io/room-persistence-library-using-mutablelivedata-observable-to-update-the-ui-after-a-database-6836d25e8267?source=collection_archive---------3----------------------->

![](img/90136464f9d413c0f73752c07298c2e5.png)

几个月前，我写了一篇关于如何使用 delegate 在房间后台数据库任务之后更新 UI 的文章。然而，尽管这种方法可行，但它可能会导致一些与活动生命周期和内存管理相关的问题。因此，在收到一些反馈后，我决定写一篇使用 ViewModel 和 MutableLiveData 实现相同结果的更好的方法。还有其他选项，如使用 rxjava 和新的 alpha 和 beta room 版本功能，但在这个示例中，我将只使用核心 Android 功能和 room 稳定版本。

如果您正在使用 room persistence 库来处理 android 应用程序上的 Sqlite 数据库操作，您应该知道该库强制开发人员在后台线程中执行所有的 db 调用。因此，如果您正在使用 DAO 模式，并且希望保留数据库逻辑，包括封装在 DAO 类中的异步执行，那么您需要以一种方式设计它们，使它们能够在后台进程完成后通知调用活动。之后，UI 更新可以在主线程上进行。

现在，我将向您展示如何通过使用 ViewModel 和 MutableLiveData 来实现这一点。对于这个例子，我们需要编写 5 个类和 1 个接口，我会试着解释最重要部分的代码。解释基本的房间概念不是本文的目的，我们将在这里集中讨论主题。在我们继续之前，我想澄清一下，我并不是想重新发明轮子。这个例子中使用的一些概念是众所周知的，我只是把我们需要的东西放在一起。所以，我们走吧。

首先，让我们为我们在这个例子中使用的模型创建一个简单的类。

```
import android.arch.persistence.room.Entity;
import android.arch.persistence.room.PrimaryKey;[@Entity](http://twitter.com/Entity)
public class Person {[@PrimaryKey](http://twitter.com/PrimaryKey)(autoGenerate = true)
    private int id;
    private String name;public int getId() {
        return id;
    }public void setId(int id) {
        this.id = id;
    }public String getName() {
        return name;
    }public void setName(String name) {
        this.name = name;
    }}
```

现在，我们需要一个 Dao 接口来定义 DB 基本操作。

```
import android.arch.lifecycle.LiveData;
import android.arch.persistence.room.Dao;
import android.arch.persistence.room.Insert;
import android.arch.persistence.room.Query;

import java.util.List;

@Dao
public interface PersonDao {

    @Insert
    void insert(Person person);

    @Query("select * from Person order by name")
    LiveData<List<Person>> getAllPersons();

}
```

之后，我们来写数据库类。

```
import android.arch.persistence.room.Room;
import android.arch.persistence.room.RoomDatabase;
import android.arch.persistence.room.Database;
import android.content.Context;[@Database](http://twitter.com/Database)(entities = {Person.class}, version = 1)
public abstract class AppDatabase extends RoomDatabase {private static volatile AppDatabase INSTANCE;private static final String DATABASE_NAME = "PersonDB";public abstract PersonDao personDao();public static synchronized AppDatabase getInstance(Context context) {
         if (INSTANCE == null) {
             //Create database object
             INSTANCE = Room.databaseBuilder(context,       AppDatabase.class, DATABASE_NAME).build();}return INSTANCE;
     }
}
```

正如您所看到的，我们对房间数据库使用了单例模式，通过这样做，我们可以对整个应用程序使用单个数据库实例。

下一步是编写一个存储库类来处理 UI 持久性调用。除了提供一种方法来处理应用程序中的多个数据源之外，这种设计类型对于让 DAO 只执行数据库操作非常有用。为了澄清这个概念，假设您的应用程序需要在本地数据库和后端 API 上进行持久化操作。repository 类将负责管理对这两层的调用。也就是说，我们可以继续存储库编码。

```
import android.arch.lifecycle.MutableLiveData;
import android.content.Context;

public class PersonRepository {

    private final PersonDao personDao;
    private MutableLiveData<Integer> insertResult = new MutableLiveData<>();

    public PersonRepository(Context context) {
        AppDatabase db = AppDatabase.*getInstance*(context);
        personDao = db.personDao();
    }

    public void insert(Person person) {
        insertAsync(person);
    }

    public MutableLiveData<Integer> getInsertResult() {
        return insertResult;
    }

    private void insertAsync(final Person person) {

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    personDao.insert(person);
                    insertResult.postValue(1);
                } catch (Exception e) {
                    insertResult.postValue(0);
                }
            }
        }).start();

    }

}
```

让我们通过解释部分代码来理解我们在这里做了什么。

```
private final PersonDao personDao;
    private MutableLiveData<Integer> insertResult = new MutableLiveData<>();

    public PersonRepository(Context context) {
        AppDatabase db = AppDatabase.*getInstance*(context);
        personDao = db.personDao();
    }

    public void insert(Person person) {
        insertAsync(person);
    }

    public MutableLiveData<Integer> getInsertResult() {
        return insertResult;
    }
```

在上面的代码片段中，我们有两个成员变量，一个用于 DAO 实例(personDAO ),另一个用于将要观察的 MutableLiveData。之后，我们有一个构造函数，它接收一个应用程序上下文并实例化一个数据库类，从中获取所需的 DAO 以存储在 personDAO 变量中。然后，我们有 insert 方法，它接收一个模型对象(person)并调用私有方法 insertAsync 传递 person 对象。结束上面的代码片段，有一个用于 MutableLiveData 成员变量的 getter。

从现在开始，我将尝试解释 repository 类的最后一部分，它包含一个私有方法，用于通过线程执行房间库所需的后台数据库操作。

```
private void insertAsync(final Person person) {

    new Thread(new Runnable() {
        @Override
        public void run() {
            try {
                personDao.insert(person);
                insertResult.postValue(1);
            } catch (Exception e) {
                insertResult.postValue(0);
            }
        }
    }).start();

}
```

基本上，我们这里有一个后台线程，它在 Try-Catch 中调用 Dao insert 方法。如果插入没有错误，则使用 postValue 方法将 MutableLiveData 值设置为 1。如果出现错误，该值将被设置为 0。请注意，我们使用的是 MutableLiveData postValue 方法，因为我们在后台线程中设置值。如果要从主线程中为 MutableLiveData 对象设置一个值，应该使用 setValue。

现在，我将在下面的代码中展示 ViewModel 类。

```
import android.app.Application;
import android.arch.lifecycle.AndroidViewModel;
import android.arch.lifecycle.LiveData;
import android.support.annotation.NonNull;

public class PersonViewModel extends AndroidViewModel {

    private PersonRepository personRepository;
    private LiveData<Integer> insertResult;

    public PersonViewModel(@NonNull Application application) {
        super(application);
        personRepository = new PersonRepository(application);
        insertResult = personRepository.getInsertResult();
    }

    public void insert(Person person) {
        personRepository.insert(person);
    }

    public LiveData<Integer> getInsertResult() {
        return insertResult;
    }
}
```

那么，ViewModel 类有什么大不了的呢？嗯，除了使用它的设计模式优势之外，ViewModel 类能够轻松处理活动生命周期。因此，当引用 ViewModel 的活动被销毁或重新创建时，ViewModel 可以自动重新连接到它，并传递任何未决的通知。

我们的 ViewModel 类将负责调用存储库任务并将结果发送给活动。在构造函数中，我们得到一个存储库实例和一个存储在成员变量中的 MutableLiveData 对象实例。除此之外，我们有一个 insert 方法，它被暴露给活动并调用存储库 insert。最后，我们有一个 LiveData observable 的 getter。

最后，让我们编写 Activity 类，这是所有过程开始的地方。

```
import android.arch.lifecycle.Observer;
import android.arch.lifecycle.ViewModelProviders;
import android.support.annotation.Nullable;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {

    private PersonViewModel personViewModel;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.*activity_main*);

        personViewModel = ViewModelProviders.*of*(this).get(PersonViewModel.class);

        Person person = new Person(); person.setName("Someone"); personViewModel.getInsertResult().observe(this, new Observer<Integer>() {
            @Override
            public void onChanged(@Nullable Integer result) {
                if (result == 1) {
                    Toast.*makeText*(MainActivity.this, "Person successfully saved", Toast.*LENGTH_SHORT*).show();
                } else {
                    Toast.*makeText*(MainActivity.this, "Error saving person", Toast.*LENGTH_SHORT*).show();
                }
            }
        });

        personViewModel.insert(person);
    }
}
```

上面的活动非常简单，它所做的唯一一件事就是通过 ViewModel 实例在数据库中插入一条 person 记录。正如我们所看到的，个人数据是硬编码的，没有供用户输入数据的表单。对于这个例子来说，这已经足够了。这里我们需要实现的就是在执行一个辅助线程之后，使用主线程更新用户界面。这是通过观察我们从视图模型中获得的实时数据来完成的。也就是说，下面是最重要的代码部分:

```
personViewModel = ViewModelProviders.*of*(this).get(PersonViewModel.class);
```

初始化 ViewModel 并将其连接到 acitivity，如上面的代码片段所示。

```
personViewModel.getInsertResult().observe(this, new Observer<Integer>() {
            @Override
            public void onChanged(@Nullable Integer result) {
                if (result == 1) {
                    Toast.*makeText*(MainActivity.this, "Person successfully saved", Toast.*LENGTH_SHORT*).show();
                } else {
                    Toast.*makeText*(MainActivity.this, "Error saving person", Toast.*LENGTH_SHORT*).show();
                }
            }
        });
```

观察 LiveData 的价值。这样，我们可以从后台线程接收通知，并在数据库操作后按照我们的要求正确地更新 UI。

就这样，我们到了这篇文章的结尾。我希望我说得很清楚，你可以利用这个例子。感谢阅读。