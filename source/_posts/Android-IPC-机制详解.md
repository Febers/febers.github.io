---
title: Android IPC 机制详解
date: 2019-02-23 19:12:35
tags:
- Android
- 多进程通信
categories:
- Android
---

## 引言
IPC（Inter-Process Communication），进程间通信或者跨进程通信，指不同的进程之间进行数据交换的过程。按照操作系统的描述，进程是指一个执行单元，在 PC 或移动设备上指一个程序/应用。容易混淆的另一个概念是线程，线程是 CPU 调度的最小单元，属于有限的系统资源。一个进程可以包含多个线程。<!--more-->

## 应用场景
只有在多进程场景下才需要 IPC 机制。

多进程的情况分为两种。第一种情况是应用因为某些原因需要采用多进程模式来实现，比如一些模块由于特殊原因需要运行在单独的进程中、为了加大可使用的内存需要多进程来获取多份内存空间（Android 对单个应用可使用的内存做了限制，早期为16MB）；第二种情况则是当前应用需要向其他应用获取数据，比如使用 ContentProvider 去查询数据。

使用多进程会造成如下的问题：
- 静态成员和单例模式完全失效
- 线程同步机制完全失效
- SharedPreferences 可靠性下降
- Application 会多次创建

对于第四点，当一个组件跑在一个新的进程中，由于系统要在创建新的线程同时分配虚拟机，这一过程相当于启动一个应用，所以会创建一个新的 Application。

## 基础概念

### Serializable 接口
Java 提供的一个序列化接口，是一个空接口，为对象提供标准的序列化和反序列化操作。使用 Serializable 实现序列化很简单，只需要实现接口，提供一个`seriaVersionUID`即可（不提供会对反序列化过程产生影响），系统便会自动完成默认的序列化过程。

```Java
public class User implements Serializable {
    private static final long seriaVersionUID = 519067123721295773L
    ...
}

//序列化过程
User user = new User("name", 18);
ObjectOutputStream out = new ObjectOutputStream(
        new FileOutputStream("file.txt"));
out.writeObject(user);
out.close

//反序列化过程
ObjectInputStream in = new ObjectInputStream(
        new FileInputStream("file.txt"));
User user = (User) in.readObject();
in.close
```
使用 Serializable 实现序列化需要注意，static 和 transient 关键字修饰的变量不会被序列化。使用 static 修饰的变量的值存储在 JVM 中。

Java 还提供了另一种序列化的方式：Externalizable接口，该接口继承自 Serializable 而且优先级更高，使用之后序列化的细节需要开发人员自己实现，同时 transient 关键字将失去作用。

### Parcelable 接口
Android 官方提供的序列化接口，实现接口之后，类的对象即可进行一个“从对象存储为 Parcel ， 从 Parcel 读取为对象”的读写过程。Parcel 包装了可序列化的数据，可以在Binder中自由传输。

其与 Serializable 的区别在于
- Parcelable 使用更复杂，后者如上面的代码，足够简洁。
- 使用内存时，Parcelable 比 Serializable 性能高，因为 Serializable 序列化的时候会产生大量临时变量，引起频繁的GC。
- 使用本地存储的时候，Serializable 更有保障性，因为 Parcelable 在外界有变化的情况下不能很好的保证数据的持续性。

标准写法如下
```Java
public class User implements Parcelable {

    private String name;
    private int id;

    protected User(Parcel in) {
        name = in.readString();
        id = in.readInt();
    }

    /*
        实例化静态变量CREATOR
        从Parcel中构造一个实现了Parcelable的类的实例,Parcel来自writeToParcel方法
     */
    public static final Creator<User> CREATOR = new Creator<User>() {
        @Override
        public User createFromParcel(Parcel in) {
            return new User(in);
        }

        @Override
        public User[] newArray(int size) {
            return new User[size];
        }
    };

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(name);
        dest.writeInt(id);
    }
}
```

通过 Intent 传递和获取数据
```Java
//传递
User user = new User("Name", 18);
Intent intent = new Intent();
intent.putExtra("data", user);

//使用
User user = getIntent().getParcelableExtra("data");
```

### Binder
Binder 是一个实现了 IBinder 接口的类。从 IPC 角度，Binder 是 Android 中的一种跨进程通信方式，Binder 还可以理解为一种虚拟的物理设备，它的设备驱动是/dev/binder，该通信方式在 Linux 中没有。

- 从 Android Framework 角度，Binder 是 ServiceManager 连接各种 Manager(ActivityManager、WindowManager 等等)和相应 ManagerService 的桥梁。
- 从 Android 应用层角度，Binder 是客户端和服务端进行通信的媒介，当我们 bindService 的时候，服务端会返回一个包含了服务端业务调用的 Binder 对象，客户端通过它获取服务端提供的服务（普通服务和基于 AIDL 的服务）或者数据。

更多关于 Binder：[图解Android - Binder 和 Service](https://www.cnblogs.com/samchen2009/p/3316001.html)

## 实现方式

### Bundle
Bundle 可以在四大组件之间通过 Intent 传递数据，它本身实现了 Parcelable 接口，能够存储基本数据类型、String、实现了 Parcelable 的对象以及以上类型的数组。

### 文件共享
利用多进程同时读写同个外部文件达到是数据交互的目的。存储形式没有限制，可以是 XML、文本、对象序列化等等。

缺点在于，Linux 系统对文件并发读写没有限制，会导致数据不同步的问题，所以该方式只适合于对数据同步要求不高的进程间通信。

### Socket
Socket也称为“套接字”，是网络通信中的概念，它分为流式套接字和用户数据套接字两种，分别应于网络的传输控制层中的TCP和UDP协议。

### AIDL

#### 概念
Android Interface Definition Language，Android 接口定义语言。

- 数据类型：作为一门“语言”，它有一些与 Java 不同的地方。除了基本数据类型、String 和 CharSequence 之外，使用其他的数据类型都需要显式导入，无论 aidl 接口是否与目标类在同一个包中。其支持的数据类型如下

> Java 中的八种基本数据类型、String、CharSequence。<br>
> List类型：其中的元素必须是AIDL支持的类型之一，或者是一个其他AIDL生成的接口，或者是定义的 Parcelable 类型。List可以使用泛型。<br>
> Map类型：其中的元素必须是AIDL支持的类型之一，或者是一个其他AIDL生成的接口，或者是定义的parcelable。Map不支持泛型。

- 定向tag：AIDLv中的定向 tag 表示了在跨进程通信中数据的流向，其中 in 表示数据只能由客户端流向服务端， out 表示数据只能由服务端流向客户端，而 inout 则表示数据可在服务端与客户端之间双向流通。其中，数据流向是针对在客户端中的那个传入方法的对象而言的。

>in 为定向 tag 时，表现为服务端将会接收到一个对象的完整数据，但是客户端的对象不会因为服务端对传参的修改而发生变动；<br>out 为定向 tag 时，表现为服务端将会接收到对象的的空对象，但是在服务端对接收到的空对象有任何修改之后，客户端将会同步变动；<br>inout 为定向 tag 时，服务端将会接收到客户端传来对象的完整信息，并且客户端将会同步服务端对该对象的任何变动。

关于更多：[你真的理解AIDL中的in，out，inout么?](https://blog.csdn.net/luoyanglizi/article/details/51958091)

- 两种类型：所有的 AIDL 文件可分为两类。一类定义 Parcelable 对象，以供其他 AIDL 文件使用非默认支持的数据类型。一类定义方法接口，以供系统完成跨进程通信。<br>可以看到，两类文件都是在“定义”，而不涉及具体的实现，这就是为什么它叫做“Android 接口定义语言”。 
  注：所有的非默认支持数据类型必须通过第一类 AIDL 文件定义才能被使用。

#### 实现
第一类 AIDL 文件：
```Java
// Book.aidl
// 这个文件的作用是引入了一个序列化对象 Book 供其他的AIDL文件使用
// 注意：Book.aidl 与 Book.java的包名应当是一样的
package com.febers.aidldemo;

//注意parcelable是小写
parcelable Book;
```

> 自定义 Parcelable 对象只支持为 in 的定向 tag 。原因在于生成的类中只有 `writeToParcel`方法，而如果要支持为 out 或者 inout 的定向 tag，还需要实现`readFromParcel`方法，而该方法并没有在 Parcelable 接口中定义，需要我们从头写。

第二类 AIDL 文件：
```Java
// BookManager.aidl
package com.lypeer.ipcclient;
//导入所需要使用的非默认支持数据类型的包
import com.febers.aidldemo.Book;

interface BookManager {

    //所有的返回值前都不需要加任何东西，不管是什么数据类型
    List<Book> getBooks();
    Book getBook();
    int getBookCount();

    //传参时，Java基本数据类型、String 以及 CharSequence 之外的类型
    //都需要在前面加上定向tag，具体加什么量需而定
    void setBookPrice(in Book book , int price)
    void setBookName(in Book book , String name)
    void addBookIn(in Book book);
    void addBookOut(out Book book);
    void addBookInout(inout Book book);
}
```
完成上面的步骤之后，`build`项目，编译器就会为我们生成相应的 BookManager 类。此时就可以开始服务端、客户端的代码编写。

服务端
```Java

public class AIDLService extends Service {

    public final String TAG = this.getClass().getSimpleName();

    //包含Book对象的list
    private List<Book> mBooks = new ArrayList<>();

    //由AIDL文件生成的BookManager
    private final BookManager.Stub mBookManager = new BookManager.Stub() {
        @Override
        public List<Book> getBooks() throws RemoteException {
            synchronized (this) {
                Log.e(TAG, "invoking getBooks() method , now the list is : " + mBooks.toString());
                if (mBooks != null) {
                    return mBooks;
                }
                return new ArrayList<>();
            }
        }

        @Override
        public void addBook(Book book) throws RemoteException {
            synchronized (this) {
                if (mBooks == null) {
                    mBooks = new ArrayList<>();
                }
                if (book == null) {
                    Log.e(TAG, "Book is null in In");
                    book = new Book();
                }
                //尝试修改book的参数，主要是为了观察其到客户端的反馈
                book.setPrice(2333);
                if (!mBooks.contains(book)) {
                    mBooks.add(book);
                }
                //打印mBooks列表，观察客户端传过来的值
                Log.e(TAG, "invoking addBooks() method , now the list is : " + mBooks.toString());
            }
        }
    };

    @Override
    public void onCreate() {
        super.onCreate();
        Book book = new Book();
        book.setName("Android开发艺术探索");
        book.setPrice(28);
        mBooks.add(book);   
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        Log.e(getClass().getSimpleName(), String.format("on bind,intent = %s", intent.toString()));
        return mBookManager;
    }
}
```

客户端
```Java
public class AIDLActivity extends AppCompatActivity {

    //由AIDL文件生成的 Java 类
    private BookManager mBookManager = null;

    //标志当前与服务端连接状况的布尔值，false为未连接，true为连接中
    private boolean mBound = false;

    //包含Book对象的list
    private List<Book> mBooks;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_aidl);
    }

    /**
     * 按钮的点击事件，点击之后调用服务端的addBookIn方法
     *
     * @param view
     */
    public void addBook(View view) {
        //如果与服务端的连接处于未连接状态，则尝试连接
        if (!mBound) {
            attemptToBindService();
            Toast.makeText(this, "当前与服务端处于未连接状态，正在尝试重连，请稍后再试", Toast.LENGTH_SHORT).show();
            return;
        }
        if (mBookManager == null) return;

        Book book = new Book();
        book.setName("APP研发录In");
        book.setPrice(30);
        try {
            mBookManager.addBook(book);
            Log.e(getLocalClassName(), book.toString());
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }

    /**
     * 尝试与服务端建立连接
     */
    private void attemptToBindService() {
        Intent intent = new Intent();
        intent.setAction("com.febers.aidl");
        intent.setPackage("com.febers.aidldemo");
        bindService(intent, mServiceConnection, Context.BIND_AUTO_CREATE);
    }

    @Override
    protected void onStart() {
        super.onStart();
        if (!mBound) {
            attemptToBindService();
        }
    }

    @Override
    protected void onStop() {
        super.onStop();
        if (mBound) {
            unbindService(mServiceConnection);
            mBound = false;
        }
    }

    private ServiceConnection mServiceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            Log.e(getLocalClassName(), "service connected");
            mBookManager = BookManager.Stub.asInterface(service);
            mBound = true;

            if (mBookManager != null) {
                try {
                    mBooks = mBookManager.getBooks();
                    Log.e(getLocalClassName(), mBooks.toString());
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            Log.e(getLocalClassName(), "service disconnected");
            mBound = false;
        }
    };
}
```
当然不要忘记在 Manifest 文件中注册 Service。
控制台显示信息如下：
> //服务端的 log 信息<br>
> 1，on bind,intent = Intent { act=com.lypeer.aidl pkg=com.lypeer.ipcserver }<br>
> 2，invoking getBooks() method , now the list is : [name : Android开发艺术探索 , price : 28]<br>
> 3，invoking addBooks() method , now the list is : [name : Android开发艺术探索 , price : 28, name : APP研发录In , price : 2333]

> //客户端的 log 信息<br>
> 1，service connected<br>
> 2，[name : Android开发艺术探索 , price : 28]<br>
> 3，name : APP研发录In , price : 2333

### Messager
Messager 底层使用了 AIDL 方式。和 AIDL 不同的是，Messager 利用 Handler 形式处理消息的接收发送，因此是线程安全的，这也表示它不支持并发处理；而 AIDL 方式是非线程安全的，支持并发处理，因此使用 AIDL 方式时需要保证代码的线程安全。

#### 概念
Messenger 有两个构造函数，分贝以 Handler 和 Binder 为参数
```Java
private final IMessenger mTarget;
public Messenger(Handler target) {
    mTarget = target.getIMessenger();
}
public Messenger(IBinder target) {
    mTarget = IMessenger.Stub.asInterface(target);    //和前面的 AIDL 很相似吧
}
```

`Handler.getIMessenger`源码：
```Java
final IMessenger getIMessenger() {
    synchronized (mQueue) {
        if (mMessenger != null) {
            return mMessenger;
        }
        mMessenger = new MessengerImpl();
        return mMessenger;
    }
}
```
IMessenger 是 AIDL 生成的跨进程接口，里面定义了一个发送消息的方法`send(android.os.Message msg)`。Handler 中 MessengerImpl 实现了该方法，使用 Handler 将消息发出去：
```Java
private final class MessengerImpl extends IMessenger.Stub {
    public void send(Message msg) {
        msg.sendingUid = Binder.getCallingUid();
        Handler.this.sendMessage(msg);
    }
}
```
所以可以使用`Handler.handlerMessage`接收消息。Messenger 中对`send`的实现如下：
```Java
public void send(Message message) throws RemoteException {
    mTarget.send(message);
}
```
Messenger 中持有一个 IMessenger 的引用，在构造函数中可以通过 Handler 或者 Binder 的形式获得最终的实现，内部`send`方法实际上是调用 IMessenger 实现的`send`方法。

Messenger 是 AIDL 的简化版，把接口都封装好，开发者只需在一个进程创建一个 Handler 传递给 Messenger，Messenger 皆可以把消息跨进程传递到另一个进程，在另一个进程的 Handler 在处理消息就可以。

#### 实现
服务端
```Java
public class MessengerService extends Service {

    private final String TAG = this.getClass().getSimpleName();

    Messenger mMessenger = new Messenger(new Handler() {
        @Override
        public void handleMessage(final Message msg) {
            if (msg != null && msg.arg1 == 9527) {
                if (msg.getData() == null) {
                    return;
                }
                String content = (String) msg.getData().get("MSG_CONTENT");  //接收客户端的消息
                Log.e(TAG, "Message from client: " + content);

                //回复消息给客户端
                Message replyMsg = Message.obtain();
                replyMsg.arg1 = 9528;
                Bundle bundle = new Bundle();
                bundle.putString("MSG_CONTENT", "已收到消息");
                replyMsg.setData(bundle);

                try {
                    msg.replyTo.send(replyMsg);     //回信
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
        }
    });

    @Nullable
    @Override
    public IBinder onBind(final Intent intent) {
        return mMessenger.getBinder();
    }
}

```

客户端代码片段
```Java
public class MessagerActivity extends AppCompatActivity {

    Messenger mClientMessenger = new Messenger(new Handler() {
        @Override
        public void handleMessage(final Message msg) {
            if (msg != null && msg.arg1 == 9528){
                if (msg.getData() == null){
                    return;
                }

                String content = (String) msg.getData().get("MSG_CONTENT");
                Log.e(TAG, "Message from server: " + content);
            }
        }
    });

    //服务端的 Messenger
    private Messenger mServerMessenger;

    private ServiceConnection mMessengerConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(final ComponentName name, final IBinder service) {
            mServerMessenger = new Messenger(service);
        }

        @Override
        public void onServiceDisconnected(final ComponentName name) {
            mServerMessenger = null;
        }
    };
    
    //发送消息
    public void sendMsg() {
        String msgContent = "消息"；

        Message message = Message.obtain();
        message.arg1 = 9527;
        Bundle bundle = new Bundle();
        bundle.putString("MSG_CONTENT", msgContent);
        message.setData(bundle);
        message.replyTo = mClientMessenger;     //指定回信人是客户端定义的

        try {
            mServerMessenger.send(message);
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }
}
```

### ContentProvider
主要用于不同的应用程序之间实现数据共享功能