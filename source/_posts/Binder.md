title: IPC之Binder原理初探
date: 2016-05-21 13:07:27
tags:
categories:
- Android
---

>IPC之Binder初探

## Binder是什么
- Binder具有粘结剂的意思，将Client、Server、Service Manager和Binder驱动程序四个组件粘连在一起
- 1 Android中一种虚拟的物理设备，/dev/binder，在Linux中没有这种通信方式。
- 2 Android中的一个类，Android.os.Binder，实现了IBinder接口。
- 3 Android FrameWork中连接ActivityManager,WindowsManager和相应的ManagerService的桥梁。
- 4 Android应用中客户端和服务端进行通信的媒介， 客户端bindService，服务端返回一个Binder对象，客户端通过Binder对象调用服务端的服务和数据，服务包含普通服务和AIDL服务。

<!-- more -->


## Binder和AIDL关系，为何要分析AIDL

在Binder用于Service通信的同时，包含Messenger和AIDL，Messenger的底层就是AIDL，分析Binder之前需要弄懂AIDL工作机制。

## AIDL(Android Interface definition language)

### 何为AIDL

全称参上，AIDL是进程间通信接口定义语言，顾名思义，笔者本章所分析的都是IPC(Inner-process-communication)进程间通信相关内容，AIDL就是其通信接口，AIDL如何构成，其怎么工作，后面分析。它具体是什么，有什么作用需要分析了它的整个面貌之后才能知晓。

### AIDL构成

当我们作为服务端需要提供进程间访问服务时，Android SDK 可以为我们定义的一个aidl接口文件自动生成一个对于的接口类，如我们定义一IBookManger.aidl文件，Book.java，和Book.aidl文件：
```java
package com.example.ipctest.aidl;
import com.example.ipctest.aidl.Book;
interface IBookManager {
  List<Book> getBookList();
  void addBook(in Book book);
}

```
Book.aidl：
```java
package com.example.ipctest.aidl;
parcelable Book;
```
Book.java:
```java
package com.example.ipctest.aidl;
import android.os.Parcel;
import android.os.Parcelable;

public class Book implements Parcelable{
	private int bookId;
	private String bookName;

	public Book(int bookId, String bookName) {
		this.bookId = bookId;
		this.bookName = bookName;
	}

	private Book(Parcel source) {
		bookId = source.readInt();
		bookName = source.readString();
	}

	@Override
	public int describeContents() {
		// TODO Auto-generated method stub
		return 0;
	}

	@Override
	public void writeToParcel(Parcel dest, int flags) {
		dest.writeInt(bookId);
		dest.writeString(bookName);
	}
	public static final Parcelable.Creator<Book> CREATOR = new Parcelable.Creator<Book>() {

		@Override
		public Book createFromParcel(Parcel source) {
			return new Book(source);
		}

		@Override
		public Book[] newArray(int size) {
			// TODO Auto-generated method stub
			return new Book[size];
		}
	};
}
```
其中Book.java实现了Parcelable接口，通过序列化和反序列才能跨进程通信, **对象不能直接跨进程传输，跨进程传输的本质就是序列化和反序列化的过程**。Book.aidl中通过Parcelable 声明 Book类。IBookManger就是aidl接口文件，其中定义的addBook(Book b),和getBookList()方法就是我们客户端需要请求服务端的方法。三个文件定义好之后，SDK就会在gen目录下对应包下生产一个IBookManager.java接口，这个接口有一个内部类abstract class Stub (桩)，Stub继承Binder实现IBookManger.java接口,Stub就是系统为我们生产的一个Binder对象，Stub内部还有一个内部类proxy。

系统生成的IBookManger.java
```java
/*
 * This file is auto-generated.  DO NOT MODIFY.
 * Original file: E:\\workspace_eclipse\\IPCtest\\src\\com\\example\\ipctest\\aidl\\IBookManager.aidl
 */
package com.example.ipctest.aidl;
public interface IBookManager extends android.os.IInterface
{
  /** Local-side IPC implementation stub class. */
  public static abstract class Stub extends android.os.Binder implements com.example.ipctest.aidl.IBookManager
  {
        private static final java.lang.String DESCRIPTOR = "com.example.ipctest.aidl.IBookManager";
          /** Construct the stub at attach it to the interface. */
        public Stub()
        {
            this.attachInterface(this, DESCRIPTOR);
        }
        /**
        * Cast an IBinder object into an com.example.ipctest.aidl.IBookManager interface,
        * generating a proxy if needed.
        */     
        public static com.example.ipctest.aidl.IBookManager asInterface(android.os.IBinder obj)
        {
            if ((obj==null)) {
              return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin!=null)&&(iin instanceof com.example.ipctest.aidl.IBookManager))) {
              return ((com.example.ipctest.aidl.IBookManager)iin);
            }
            return new com.example.ipctest.aidl.IBookManager.Stub.Proxy(obj);
        }

        @Override public android.os.IBinder asBinder()
          {
            return this;
          }

        @Override public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException
        {
            switch (code)
              {
                case INTERFACE_TRANSACTION:
                  {
                    reply.writeString(DESCRIPTOR);
                    return true;
                  }
                case TRANSACTION_getBookList:
                  {
                    data.enforceInterface(DESCRIPTOR);
                    java.util.List<com.example.ipctest.aidl.Book> _result = this.getBookList();
                    reply.writeNoException();
                    reply.writeTypedList(_result);
                    return true;
                  }
                case TRANSACTION_addBook:
                  {
                    data.enforceInterface(DESCRIPTOR);
                    com.example.ipctest.aidl.Book _arg0;
                    if ((0!=data.readInt())) {
                    _arg0 = com.example.ipctest.aidl.Book.CREATOR.createFromParcel(data);
                  }else {
                    _arg0 = null;
                  }
                    this.addBook(_arg0);
                    reply.writeNoException();
                    return true;
                  }
                }
              return super.onTransact(code, data, reply, flags);
            }

        private static class Proxy implements com.example.ipctest.aidl.IBookManager
          {
              private android.os.IBinder mRemote;

              Proxy(android.os.IBinder remote)
                {
                  mRemote = remote;
                }

              @Override public android.os.IBinder asBinder()
              {
                return mRemote;
              }

              public java.lang.String getInterfaceDescriptor()
              {
                return DESCRIPTOR;

              }

              @Override public java.util.List<com.example.ipctest.aidl.Book> getBookList() throws android.os.RemoteException
              {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                java.util.List<com.example.ipctest.aidl.Book> _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    mRemote.transact(Stub.TRANSACTION_getBookList, _data, _reply, 0);
                    _reply.readException();
                    _result = _reply.createTypedArrayList(com.example.ipctest.aidl.Book.CREATOR);
                }finally {
                  _reply.recycle();
                  _data.recycle();
                }
                return _result;
              }
              @Override public void addBook(com.example.ipctest.aidl.Book book) throws android.os.RemoteException
              {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                try {
                  _data.writeInterfaceToken(DESCRIPTOR);
                  if ((book!=null)) {
                    _data.writeInt(1);
                    book.writeToParcel(_data, 0);
                  }else {
                    _data.writeInt(0);
                  }
                  mRemote.transact(Stub.TRANSACTION_addBook, _data, _reply, 0);
                  _reply.readException();
                }finally {
                  _reply.recycle();
                  _data.recycle();
                }
              }
          }
          static final int TRANSACTION_getBookList = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
          static final int TRANSACTION_addBook = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
    }
    public java.util.List<com.example.ipctest.aidl.Book> getBookList() throws android.os.RemoteException;
    public void addBook(com.example.ipctest.aidl.Book book) throws android.os.RemoteException;
}

```
以上代码由SDK在gen目录生成，SDK生成的代码在eclipse中居然没有格式，真不能忍，改了格式后如上。IBookManager继承至IInterface接口，其本身还是一个接口，然后又有内部类，内部类还有内部类，关系稍微有点儿复杂，为了便于叙述，笔者强行画了UML类图：
![](http://7xrw2w.com1.z0.glb.clouddn.com/Binder.png)

- asBinder：
  参考类图和UML，IInterface接口只有一个方法asBinder()。proxy的外部类Stub才是一个真正的Binder类，Stub类的asBinder实现就是返回其this,及返回当前的Binder对象，proxy不是一个Binder的子类，其asBinder也是返回当前Binder对象。

- DESCRIPTOR
  Binder的标识，即包名。

- asInterface(IBinder obj)
  Stub中实现，其返回值有三种，null,IBookManager,其子类proxy代理类。首先`android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);`请求本地的服务，如果没有跨进程通信，客户端服务端在同一个进程，返回的是Stub本身，如果客户端服务端不在同一个进程，则返回系统封装后的Stub.Proxy类。这个方法主要将服务端Binder对象转化成客户端所需要的AIDL接口类型对象。一般是将具体的proxy转换成IXXXManger接口。

- proxy:getBookList(),addBook(Book)
  这两个方法运行在客户端，当客户端调用此方法，则创建Parcle对象：参数的输入靠data,输出靠对象reply和返回值对象List，然后通过(Binder.transact) mRemote.transact方法发起RPC(远程过程调用)请求，同时线程挂起，等待服务端的onTransact封装数据回调，然后继续执行从reply中取出RPC过程返回结果。

- onTransact();
  这个方法实现在Stub中，运行在服务端的Binder线程池中。其作用是对code参数进行判断，利用Stub中定义的静态int类型的Code确定请求的方法，然后执行目标方法，数据通过Parcle类进行序列化操作，执行完毕后，向reply写入返回值。

- 梳理：看完几个关键的方法，其实发现还好，不是特别复杂，现在来梳理一遍：
  首先客户端通过asInterface获取服务端返回的IXXXManger接口，再通过IXXXManger发起远程请求，也就是调用两个方法getBookList()，addBook(Book)，这两个方法调用proxy构造时创建时依赖的IBinder接口中的transact方法发送远程过程调用RPC，同时挂起当前线程，服务端调用运行在Binder线程池中的onTransact()方法，根据code处理相关的请求。在reply中写入结果，返回Binder对象，客户端proxy在从reply中获取结果(如果有返回值)。
  注意问题：
  1.客户端发出请求后会被挂起，直到返回。如果服务端是耗时操作，所以不能在UI线程发起请求。
  2.服务端运行在Binder线程池中，用同步实现。
流程图：
![](http://7xrw2w.com1.z0.glb.clouddn.com/Binder_transact.png)
AIDL文件并不是实现Binder通信的必须品，这个文件的提供只是给我们一个工具生成Binder及proxy对象。
自己实现：在服务端就是创建一个继承IInterface接口的IBookManager接口，创建两个业务抽象方法：addBook(),getBookList(),然后创建两个用于标识这两个方法的静态字段。同时创建一个BookManagerImpl实现类实现此接口并继承Binder。在其内部创建一个Proxy内部类运行在客户端发送远程过程请求transact(),在其ManagerImpl中实现onTransact()处理请求。如下：
IBookManager.java：
```java
/**
 * 手动实现 Binder对象 不依靠AIDL文件 客户端接口*
 * @author zhangyi
 */
public interface IBookManager extends IInterface{
	static final String DESCRIPTOR = "com.example.ipctest.IBookManager";
	static final int TRANSACTION_getBookList = IBinder.FIRST_CALL_TRANSACTION+0;
	static final int TRANSACTION_addBook = IBinder.FIRST_CALL_TRANSACTION+1;
	public List<Book> getBookList() throws RemoteException;
	public void addBook(Book book) throws RemoteException;
}
```
BookManagerImpl:
```java
public class BookManagerImpl extends Binder implements IBookManager{

	public BookManagerImpl() {
		this.attachInterface(this, DESCRIPTOR);
	}

	//服务端对象转换成客户端对象所需的AIDL接口类型的对象，这种转换区分进程，如果位于同一个进程，返回服务端Stub本身，如果不同进程，返回Stub.proxy对象
	public static IBookManager asInterface(IBinder obj) {
		if(obj==null){
			return null;
		}
		IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
		if( iin!=null &&(iin instanceof IBookManager)) {
			return (IBookManager) iin;
		}
		return new BookManagerImpl.Proxy(obj);

	}

	@Override
	protected boolean onTransact(int code, Parcel data, Parcel reply, int flags)
			throws RemoteException {
		switch (code) {
		case INTERFACE_TRANSACTION:{			
			reply.writeString(DESCRIPTOR);
			return true;
		}
		case TRANSACTION_addBook:{
			data.enforceInterface(DESCRIPTOR);
			Book arg0;
			if(data.readInt()!=0){
				arg0 = Book.CREATOR.createFromParcel(data);
			}else {
				arg0 = null;
			}
			this.addBook(arg0);
			reply.writeNoException();
			return true;
		}
		case TRANSACTION_getBookList:{
			data.enforceInterface(DESCRIPTOR);
			List<Book> result = this.getBookList();
			return true;
		}
		}
		return super.onTransact(code, data, reply, flags);
	}

	private static class Proxy implements IBookManager{

		private IBinder mRemoteBinder;
		public Proxy(IBinder binder) {
			mRemoteBinder = binder;
		}

		@Override
		public IBinder asBinder() {
			// TODO Auto-generated method stub
			return mRemoteBinder;
		}

		public java.lang.String getInterfaceDescriptor() {
			return DESCRIPTOR;
		}

		@Override
		public List<Book> getBookList() throws RemoteException {
			Parcel data = Parcel.obtain();
			Parcel replyParcel = Parcel.obtain();

			List<Book> result;
			try {
				data.writeInterfaceToken(DESCRIPTOR);
				mRemoteBinder.transact(TRANSACTION_getBookList, data, replyParcel, 0);//挂起
				replyParcel.readException();
				result = replyParcel.createTypedArrayList(Book.CREATOR);
			}finally{
				replyParcel.recycle();
				data.recycle();
			}
				return result;
		}

		@Override
		public void addBook(Book book) throws RemoteException {
			Parcel data = Parcel.obtain();
			Parcel reply = Parcel.obtain();
			try {

				data.writeInterfaceToken(DESCRIPTOR);
				if(book!=null){
					data.writeInt(1);
					book.writeToParcel(data, 1); //序列化属性
				}
				else {
					data.writeInt(0);
				}
				mRemoteBinder.transact(TRANSACTION_addBook, data, reply, 0);
				reply.readException();
			}finally{
				reply.recycle();
				data.recycle();
			}

		}

	}
	@Override
	public IBinder asBinder() {
		return this;
	}
	@Override
	public List<Book> getBookList() throws RemoteException {
		// TODO 服务器具体实现 如从数据库中获取
		return null;
	}
	@Override
	public void addBook(Book book) throws RemoteException {
	}
}

```

## Binder如何维护连接
### Binder中linkToDeath 和 unLinkToDeath();
看源码，Binder对象有两个重要方法，linkToDeath(),和unLinkToDeath();Binder运行在服务端异常终止，客户端远程调用就会失败，Binder中提供了这两个配对的方法，通过LinkToDeath()给Binder设置了一个死亡代理，当Binder死亡的时候，就会收到通知。就可以重新发送请求恢复连接，提供服务。同时unLinkToDeath()就是结束代理。
```java
/**
   * Local implementation is a no-op.
   */
  public void linkToDeath(DeathRecipient recipient, int flags) {
  }

  /**
   * Local implementation is a no-op.
   */
  public boolean unlinkToDeath(DeathRecipient recipient, int flags) {
      return true;
  }

```
DeathRecipient
```java
  /**
     * Interface for receiving a callback when the process hosting an IBinder
     * has gone away.
     *
     * @see #linkToDeath
     */
    public interface DeathRecipient {
        public void binderDied();
    }
```
### 如何设置死亡代理

根据源码可以看出，接口DeathRecipient只有一个方法binderDied()，设置代理LinkToDeath就要声明一个DeathRecipient接口，实现binderDied方法。
服务端
```java
  DeathRecipient mDeathRecipient  = new IBinder.DeathRecipient(){

    @Override
    public void binderDied(){
      if(mBookManager == null){
        return;
      }
      mBookManager.asBinder().unlinkToDeath(mDeathRecipient,0);
      mBookManager == null;
      //TODO 重新绑定远程服务


    }
  }
```
客户端
```java
  mService = IMessageBoxManager.Stub.asInterface(binder);
  binder.linkToDeath(mDeathRecipient,0)
```
标志位0直接设置即可。isBinderAlive()可以直接判断Binder是否死亡。

## 笔者思考的问题：

> 当我们使用Binder机制通讯时，客户端代码和服务端代码怎么写？客户端如何调用服务端的方法。其调用过程需要注意些什么？

客户端代码和服务端代码如下：
### 使用AIDL
服务端
- 1.创建一个AIDL文件，将暴露给客户端的接口(及所支持的服务方法)在这个文件中声明
- 2.创建一个Service 并注册,指定process属性，代表不同进程。
- 3 创建IBookManager.Stub对象，实现具体业务逻辑。通过onBind()将Binder返回给客户端。

IBookManager.aidl
```java
package com.example.messengerdemo.aidl;
import com.example.messengerdemo.aidl.Book;

interface IBookManager {
	List<Book> getBookList();
	void addBook(in Book book);
}
```
Book.aidl
```java
package com.example.messengerdemo.aidl;

parcelable Book;

```
服务端实现：
```java
public class BookManagerService extends Service {
	private static CopyOnWriteArrayList<Book> mBooklistArrayList = new CopyOnWriteArrayList<Book>();//并发读写

	private static Binder mBinder = new IBookManager.Stub() {

		@Override
		public List<Book> getBookList() throws RemoteException {
			return mBooklistArrayList;
		}

		@Override
		public void addBook(Book book) throws RemoteException {
			mBooklistArrayList.add(book);
		}
	};
 	@Override
	public IBinder onBind(Intent intent) {
		return mBinder; //通过onBind方法返回服务端的创建的Binder对象
	}
	@Override
	public void onCreate() {
		super.onCreate();
		mBooklistArrayList.add(new Book(1, "1"));
		mBooklistArrayList.add(new Book(2, "2"));
	}
}
```


客户端实现
1.创建ServiceConnection，并绑定服务端。
2.绑定成功后，在ServiceConnection的onServiceConnected方法中调用aidl生成的IBookManager.Stub.asInterface方法返回其服务端返回的Binder对象
3.通过Binder对象发起远程过程调用，并阻塞等待返回。
```java
public class MainActivity extends Activity {
	private static final String TAG = "BookMangerActivity";
	//通过ServiceConnection和服务端进行通信
	private ServiceConnection mConnection = new ServiceConnection() {
		@Override
		public void onServiceDisconnected(ComponentName name) {

		}
		@Override
		public void onServiceConnected(ComponentName name, IBinder service) {
			IBookManager bookManager = IBookManager.Stub.asInterface(service);//客户端代理对象
			try {
				//获取列表方法测试 应该开启子线程，不能在UI线程直接调用耗时任务。
				List<Book> books = bookManager.getBookList();
				Log.i("BookMangerActivity", books.getClass()+"");
				getComponentName();
				Log.i("BookMangerActivity","qurey books"+books.toString());
				//添加方法测试
				Book book = new Book(3, "test");
				bookManager.addBook(book);
				List<Book> newBooks = bookManager.getBookList();
				Log.i("BookMangerActivity","qurey books"+newBooks.toString());			
			} catch (RemoteException e) {
				e.printStackTrace();
			}
		}
	};

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);

		//通过Intent确定绑定服务的两端
		Intent intent = new Intent(this,BookManagerService.class);
		bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
	}

	@Override
	protected void onDestroy() {
		unbindService(mConnection);
		super.onDestroy();
	}
}
```
以上就可以完成客户端调用服务端，并可以传递参数，和得到返回结果。


## 如何做到客户端和服务端的观察者模式

如何实现客户端向服务端注册(及关注)，同时服务端可以自动给客户端提供推送消息。同时客户端可以取消这种关注，类似于微信公众号的机制。
如果在同一进程中，实现就比较简单，只需要普通的监听接口。客户端需要实现服务端定义的监听接口IOnNewBookArrivedListener，注册的时候将实现的监听接口添加到服务端管理的监听集合中，推送消息时，服务端调用监听器的回调方法，将NewMessege作为参数传入,供客户端使用。
但是跨进程通信，首先不能使用普通接口，普通接口无法跨进程通信。而需要提供AIDL接口，结合AIDL使用的时候，就要考虑到监听器如何回调？当前运行的线程是否是UI线程或Binder线程池？而且还要考虑如何跨进程注册监听？及取消注册监听？
其主要步骤：
1.创建一个监听接口定义aidl文件：IOnNewBookArrivedListener.aidl
2.接口文件中定义onNewBookArrived的回调方法。
3.服务端调用回调方法。
IOnNewBookArrivedListener.aidl
```java
package com.example.messengerdemo.aidl;
import com.example.messengerdemo.aidl.Book;

interface IOnNewBookArrivedListener{
	void onNewBookArrived(in Book book);
}
```
同时SDK也会为其生成一个Binder的实现，用于跨进程通信，生成的IOnNewBookArrivedListener.java参考 **AIDL的构成** 中IBookManger.java。

>客户端服务端的实现:

### 一种不恰当的实现方式，只能注册并收到通知，不能取消注册
服务端
1.实现暴露给客户端接口的Stub内部类中的业务逻辑
2.通过onBind方法将Binder返回给客户端
3.开启线程，定时刷新调用已注册的回调监听对象。
4.实现Binder监听接口中注册监听，取消注册方法。
```java
public class BookManagerService extends Service {
	private static CopyOnWriteArrayList<Book> mBooklistArrayList = new CopyOnWriteArrayList<Book>();//并发读写
	private static CopyOnWriteArrayList<IOnNewBookArrivedListener> mListenerList = new CopyOnWriteArrayList<>(); //维护监听集合
	private AtomicBoolean mIsServiceDestoryed = new AtomicBoolean(false);
  //创建服务端Binder对象
	private static Binder mBinder = new IBookManager.Stub() {
    //其运行在服务端Binder线程池中
		@Override
		public List<Book> getBookList() throws RemoteException {
			return mBooklistArrayList;
		}

		@Override
		public void addBook(Book book) throws RemoteException {
			mBooklistArrayList.add(book);
		}
    //注册监听
		@Override
		public void registerListener(
				IOnNewBookArrivedListener iOnIOnNewBookArrivedListener)
				throws RemoteException {
			if(!mListenerList.contains(iOnIOnNewBookArrivedListener)){
				mListenerList.add(iOnIOnNewBookArrivedListener);
			}else {
				Log.i("BookManagerService", "already register!");
			}
		}
    //取消注册监听
		@Override
		public void unregisterListener(
				IOnNewBookArrivedListener iOnNewBookArrivedListener)
				throws RemoteException {
			if(mListenerList.contains(iOnNewBookArrivedListener)){
				mListenerList.remove(iOnNewBookArrivedListener);
			}else {
				Log.i("BookManagerService", "not found , can not register!");
			}
		}
	};
  //通过onBind方法返回Bind对象
 	@Override
	public IBinder onBind(Intent intent) {
		return mBinder;
	}
	@Override
	public void onCreate() {
		super.onCreate();
		mBooklistArrayList.add(new Book(1, "1"));
		mBooklistArrayList.add(new Book(2, "2"));
		new Thread(new ServiceWorker()).start();//开启推送线程
	}

  //这个方法调用的是客户端创建Binder监听器，运行在客户端的Binder线程池中。是一个耗时任务，所以这里不应该在服务端的UI线程中调用
	private void onNewBookArrived(Book book) throws RemoteException{
		mBooklistArrayList.add(book);1
		for (int i = 0; i < mListenerList.size(); i++) {
			Log.i("BookManagerService", "监听器回调");
			mListenerList.get(i).onNewBookArrived(book);//传入NewMessege参数回调
		}
	}

	private class ServiceWorker implements Runnable{
		@Override
		public void run() {
			while(!mIsServiceDestoryed.get()){
				try {
					Thread.sleep(5000);
				} catch (Exception e) {
				}
			}
			int bookId = mBooklistArrayList.size()+1;
			Book book = new Book(bookId, "new Book#"+bookId);
			try {
				onNewBookArrived(book); //调用回调监听
			} catch (RemoteException e) {
				e.printStackTrace();
			}
		}
	}
}
```
1.实现监听回调接口Binder,获取数据后，可向UI线程发送消息。
2.绑定服务，创建ServiceConnection，并实现onServiceConnected方法调用asInterface获取服务端返回的binder对象。
3.通过Binder对象发起远程调用。
4.注册回调监听。
5.销毁时取消回调监听。
客户端：
```java
public class MainActivity extends Activity {
	private static final String TAG = "BookMangerActivity";
	private static final int MESSAGE_NEW_BOOK_ARRIVED = 1;
	private IBookManager mRemoteBookManager;
	private Handler handler = new Handler(){

		@Override
		public void handleMessage(Message msg) {

			switch (msg.what) {
			case MESSAGE_NEW_BOOK_ARRIVED:
				Log.i(TAG, "received new Book"+msg.obj);
				break;
			default:
				break;
			}

		}
	};
	//通过ServiceConnection和服务端进行通信
	private ServiceConnection mConnection = new ServiceConnection() {
    //以下两个方法运行在UI线程，调用proxy对象的业务方法发起远程过程调用后会被挂起，耗时操作，所以这里可能会导致ANR
		@Override
		public void onServiceDisconnected(ComponentName name) {

		}
		@Override
		public void onServiceConnected(ComponentName name, IBinder service) {
		    mRemoteBookManager = IBookManager.Stub.asInterface(service);//绑定成功后将服务返回的Binder转换成AIDL接口，实际返回proxy对象，非Binder
			try {
				//获取列表方法测试
				List<Book> books = mRemoteBookManager.getBookList();
				Log.i("BookMangerActivity", books.getClass()+"");
				getComponentName();
				Log.i("BookMangerActivity","qurey books"+books.toString());

				//添加方法测试
				Book book = new Book(3, "test");
				mRemoteBookManager.addBook(book);
				List<Book> newBooks = mRemoteBookManager.getBookList();
				Log.i("BookMangerActivity","qurey books"+newBooks.toString());		

				mRemoteBookManager.registerListener(listener);//注册监听器
			} catch (RemoteException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}				
		}
	};

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		//通过Intent确定绑定服务的两端
		Intent intent = new Intent(this,BookManagerService.class);
		bindService(intent, mConnection, Context.BIND_AUTO_CREATE);//绑定服务，
	}

	//在客户端创建一个Binder对象，用于服务端的调用，Binder总是出现于被调用的一端。此类运行在客户端的Binder线程池中，也就是服务端调用的时候，是运行在客户端的线程池中
	private IOnNewBookArrivedListener listener = new IOnNewBookArrivedListener.Stub() {
		//Binder线程池中需要并发处理，需要向UI线程发消息。
		@Override
		public void onNewBookArrived(Book book) throws RemoteException {
			handler.obtainMessage(MESSAGE_NEW_BOOK_ARRIVED, book).sendToTarget();
		}
	};

	@Override
	protected void onDestroy() {
		if(mRemoteBookManager!=null && mRemoteBookManager.asBinder().isBinderAlive()){
			try {
          //mRemoteBookManger 是在UI线程中获取的proxy对象，其传入的参数Listener是不能直接跨进程的。listener在客户端这里是一个对象，如何保证在服务端也是同一个对象呢？
          //如果不能保证在服务端是同一个对象，就不能完成取消注册的功能。
          mRemoteBookManager.unregisterListener(listener);
			} catch (RemoteException e) {
				e.printStackTrace();
			}
		}
		unbindService(mConnection);
		super.onDestroy();
	}
}
```
总结：不恰当的方式有以下几点：
- 1.从UI线程不能调用耗时任务，防止导致ANR的角度来说：
  - 客户端ServiceConnection 中的onServiceConnected 和 onServiceDisconnected 方法是运行在UI线程，不能调用asInterface返回的IBookManger接口对象(proxy对象)的addBook和getBookList方法，proxy对象要发起远程请求transact()，这是耗时的。
  - 服务端onNewBookArrived方法调用的是客户端创建的Binder监听器IOnNewBookArrivedListener对象的onNewBookArrived方法，运行在客户端的Binder线程池中，耗时操作。所以服务端的onNewBookArrived方法不能在UI线程调用。

- 2.从对象不能直接跨进程传输的角度来说：
  - 客户端在onServiceConnected中调用proxy对象的registerListener方法注册监听器时，传入的listener参数和在onDestroy方法中调用的unregisterListener方法传入的listener参数对于客户端来说是一个对象，但是对于服务端来说是不同的对象，对于服务端来说对象跨进程传输的反序列化后的新的对象。所以在unregisterListener时候并不能找到已经注册的对象，而打印not found , listener unregister，不能完成取消注册的功能。

- 3.异步任务
  - 服务端的方法本来就运行在线程池中可以执行大量的耗时操作，所以不要在服务端的线程池里另外开启线程执行异步任务。

所以之前的实现很多地方都是不恰当的。

### 正确实现：

如何解决不恰当方式中的第2点?
对于服务端来说，registerListener传入的listener在服务端是不同的对象，构造Listener的时候是通过客户端IBookManger.Stub()生成的Binder对象。这个Binder对于服务器来说是同一个Binder,why?
取消注册需要用到RemoteCallbackList<>，它是一个泛型，专门用于删除跨进程的listner接口。
服务端改进：
```java
...
private RemoteCallbackList<IOnNewBookArrivedListener> mListenerList = new RemoteCallbackList<IOnNewBookArrivedListener>();
private AtomicBoolean mIsServiceDestoryed = new AtomicBoolean(false);
private Binder mBinder = new IBookManager.Stub() {
  ...
  @Override
  public void registerListener(
      IOnNewBookArrivedListener iOnIOnNewBookArrivedListener)
      throws RemoteException {
    mListenerList.register(iOnIOnNewBookArrivedListener);

  }

  @Override
  public void unregisterListener(
      IOnNewBookArrivedListener iOnNewBookArrivedListener)
      throws RemoteException {
    mListenerList.unregister(iOnNewBookArrivedListener);
  }
};
...
private void onNewBookArrived(Book book) throws RemoteException {
  mBooklistArrayList.add(book);
  int N = mListenerList.beginBroadcast();
  for (int i = 0; i < N; i++) {

    IOnNewBookArrivedListener listener = mListenerList
        .getBroadcastItem(i);
    if (listener != null) {
      listener.onNewBookArrived(book);
    }
  }
  mListenerList.finishBroadcast();//必须和begin配对使用
}
```

为了程序的健壮性，在客户端中我们需要在Binder意外死亡的时候做处理，除了之前设置DeathRecipient监听，重连服务以外还有另外一种方式，在onServiceDisConnected中重连远程服务，区别如下：
- onServiceDisConnected在UI线程中回调
- binderDied在客户端Binder线程中回调，不可以访问UI线程。

## 服务权限验证的几种方式

### 自定义权限及使用
```xml
<permission android:name="com.example.aidldemo.permission.ACCESS_BOOK_SERVICE"
        android:protectionLevel="normal"
        android:label="@string/app_name"        />

<uses-permission   
       android:name="com.example.aidldemo.permission.ACCESS_BOOK_SERVICE"  />

```
声明的含义如下:
android:label：权限名字，显示给用户的，值可是一个 string 数据，例如这里的“自定义权限”。
android:description：比 label 更长的对权限的描述。值是通过 resource 文件中获取的，不能直接写 string 值，例如这里的”@string/test”。
android:name：权限名字，如果其他 app 引用该权限需要填写这个名字。
android:protectionLevel：权限级别，分为 4 个级别：
- normal：低风险权限，在安装的时候，系统会自动授予权限给 application。
- dangerous：高风险权限，系统不会自动授予权限给 app，在用到的时候，会给用户提示。
- signature：签名权限，在其他 app 引用声明的权限的时候，需要保证两个 app 的签名一致。这样系统就会自动授予权限给第三方 app，而不提示给用户。
- signatureOrSystem：这个权限是引用该权限的 app 需要有和系统同样的签名才能授予的权限，一般不推荐

### onBind中验证
在onBind中验证，验证不通过就返回null，从而不返回服务端的Binder对象。 如下采用permission方式验证
```java
@Override
	public IBinder onBind(Intent intent) {
		//权限验证
		int check = checkCallingOrSelfPermission("com.example.aidldemo.permission.ACCESS_BOOK_SERVICE");
		if(check==PackageManager.PERMISSION_DENIED){
			return null;
		}
		return mBinder;
	}

```
### onTransact()中验证
在服务端的onTransact中验证，验证不通过就返回false,这样服务端不会终止AIDL中的方法而达到保护服务端的效果，例如用UID和PID来做验证。
UID PID:
PID：为Process Identifier
PID就是各进程的身份标识,程序一运行系统就会自动分配给进程一个独一无二的PID。进程中止后PID被系统回收，可能会被继续分配给新运行的程序，但是在android系统中一般不会把已经kill掉的进程ID重新分配给新的进程，新产生进程的进程号，一般比产生之前所有的进程号都要大。
UID：一般理解为User Identifier
UID在linux中就是用户的ID，表明时哪个用户运行了这个程序，主要用于权限的管理。而在android 中又有所不同，因为android为单用户系统，这时UID 便被赋予了新的使命，数据共享，为了实现数据共享，android为每个应用几乎都分配了不同的UID，不像传统的linux，每个用户相同就为之分配相同的UID。（当然这也就表明了一个问题，android只能时单用户系统，在设计之初就被他们的工程师给阉割了多用户），使之成了数据共享的工具。
同时验证permission和包名方式：
```java
@Override
    public boolean onTransact(int code, Parcel data, Parcel reply, int flags)
      throws RemoteException {
    //首先验证远程调用服务的方法中必须含有自定义权限
    int check = checkCallingOrSelfPermission("com.example.aidldemo.permission.ACCESS_BOOK_SERVICE");
    if(check == PackageManager.PERMISSION_DENIED){
      return false;
    }
    String packageName = null;
    String [] packages =  getPackageManager().getPackagesForUid(getCallingUid());
    if(packages!=null && packages.length>0){
      packageName = packages[0];
    }
    //其次 应用的报名必须是com.example.aidldemo开头
    if(!packageName.startsWith("com.example.aidldemo")){
      return false;
    }
    return super.onTransact(code, data, reply, flags);
  }

```

### Service 指定android permission属性
```xml
<service
  android:name="com.example.aidldemo.BookManagerService"
    android:process=":remote"
    android:permission="com.example.aidldemo.permission.ACCESS_BOOK_SERVICE"
></service>
```


## Binder连接池优化方案
一个应用，功能模块众多，假设其中两个分别为加密解密服务和计算服务，如果为每一个模块写一个Service，并创建一个对应的AIDL，这样就会就比较浪费资源，一个Service的开销不小，十几个，或者几十个Service的开销，移动操作系系统也许不能负担，所以引入了连接池的概念。

解决方案：会将多个AIDL写入一个Service中。Service提供一个匹配各种不同服务的Binder接口，即连接池，通过连接池查询各种业务相应模块的binder，再通过Binder将请求统一发到一个远程Service中执行。

1.创建相应模块的aidl接口文件并实现其中业务逻辑方法。
服务端：
ICompute.aidl
```java
package com.example.binderpooldemo.aidl;
interface ICompute{
	int add(int a,int b);
}
```

ISecurityCenter.aidl
```java
package com.example.binderpooldemo.aidl;
interface ISecurityCenter{
	String encrypt(String content);
	String decrypt(String password);
}
```
实现：
```java
public class SecurityCenterImpl extends ISecurityCenter.Stub{
	@Override
	public String encrypt(String content) throws RemoteException {
		// TODO Auto-generated method stub
		return null;
	}
	@Override
	public String decrypt(String password) throws RemoteException {
		// TODO Auto-generated method stub
		return null;
	}
}
public class ComputeImpl extends ICompute.Stub {
	@Override
	public int add(int a, int b) throws RemoteException {
		// TODO Auto-generated method stub
		return 0;
	}
}
```
2.创建连接池接口文件，提供queryBinder方法；IBinderPool.aidl
```java
package com.example.binderpooldemo.aidl;
interface IBinderPool{
	IBinder queryBinder(int binderCode);
}
```

3.创建一个连接池BinderPool.java，在内部实现IBinderPool.aidl接口：
```java
public class BinderPool {
	private static final String TAG = "BinderPool";
	public static final int BINDER_NOTE = -1;
	public static final int BINDER_COMPUTE = 0;
	public static final int BINDER_SECURITY_CENTER = 1;
	private Context mContext;
	private IBinderPool mBinderPool;
	private static volatile BinderPool sIntance;
	private CountDownLatch mConnectBinderPoolCountDownLatch;

	private BinderPool(Context context) {
		mContext = context;
		connectBinderPoolService();
	}

	public static BinderPool getInstance(Context context){
		if(sIntance==null){
			synchronized (BinderPool.class) {
				if(sIntance==null){
					sIntance = new BinderPool(context);
				}
			}
		}
		return sIntance;
	}

	public IBinder queryBinder(int binderCode) {
		IBinder iBinder = null;
		if (mBinderPool != null) {
			try {
				iBinder = mBinderPool.queryBinder(binderCode);
			} catch (RemoteException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		return iBinder;
	}

	private synchronized void connectBinderPoolService() {
		mConnectBinderPoolCountDownLatch = new CountDownLatch(1);
		Intent serviceIntent = new Intent(mContext, BinderPoolService.class);
		mContext.bindService(serviceIntent, connection,
				Context.BIND_AUTO_CREATE);
		try {
			mConnectBinderPoolCountDownLatch.wait();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}

	private ServiceConnection connection = new ServiceConnection() {
		@Override
		public void onServiceDisconnected(ComponentName name) {

		}
		@Override
		public void onServiceConnected(ComponentName name, IBinder service) {
			mBinderPool = IBinderPool.Stub.asInterface(service);
			try {
				mBinderPool.asBinder().linkToDeath(mBinderPoolDeathRecipient,0);
			} catch (RemoteException e) {
				e.printStackTrace();
			}
			mConnectBinderPoolCountDownLatch.countDown();
		}
	};

	private IBinder.DeathRecipient mBinderPoolDeathRecipient = new IBinder.DeathRecipient() {
		@Override
		public void binderDied() {
			mBinderPool.asBinder().unlinkToDeath(mBinderPoolDeathRecipient, 0);
			mBinderPool = null;
			connectBinderPoolService();
		}
	};
	public static class BinderPoorImpl extends IBinderPool.Stub {
		@Override
		public IBinder queryBinder(int binderCode) throws RemoteException {
			IBinder binder = null;
			switch (binderCode) {
			case BINDER_SECURITY_CENTER:
				binder = new SecurityCenterImpl();
				break;
			case BINDER_COMPUTE:
				binder = new ComputeImpl();
				break;
			default:
				break;
			}
			return binder;
		}
	}
}
```
连接池通过ServiceConnection获取IBinderPool接口的mBinderPoor对象，并提供绑定远程服务，设置死亡代理，并提供重连机制。客户端通过其mBinderPoor中实现的queryBinder方法返回跟业务相关的Binder进行各自的操作。
服务端：
BinderPoolService.java服务类实现：
```java
public class BinderPoolService extends Service {
	private Binder mBinderPool = new BinderPool.BinderPoorImpl();
	@Override
	public IBinder onBind(Intent intent) {
		return mBinderPool;
	}
	@Override
	public void onCreate() {
		super.onCreate();
	}
}
```
服务类中实例化连接池接口对象，并将Binder通过onBind方法返回，供客户端绑定。
客户端：
```java
public class MainActivity extends Activity {

	private ISecurityCenter mSecurityCenter;
	private ICompute mCompute;
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
	}
	//这个方法需要在线程中执行
	private void doWork(){
		BinderPool binderPool = BinderPool.getInstance(MainActivity.this);
		IBinder securityBinder = binderPool.queryBinder(BinderPool.BINDER_SECURITY_CENTER);
		mSecurityCenter = (ISecurityCenter)SecurityCenterImpl.asInterface(securityBinder);

		IBinder computeBinder = binderPool.queryBinder(BinderPool.BINDER_COMPUTE);
		mCompute = (ICompute)ComputeImpl.asInterface(computeBinder);
		try {
			mSecurityCenter.encrypt("abc");
		} catch (RemoteException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

	}
}
```
客户端调用远程服务doWork()不能在主线程中调用，BinderPool初始化的时候需要绑定远程服务，包括后面queryBinder方法都是调用远程服务的queryBinder方法，运行在服务的Binder线程池中。同时connectBinderPoolService是同步的方法。
如果业务扩展，添加新的Binder服务，则只需创建aidl接口文件，在IBinderPool的queryBinder中添加新的binder返回就行了。不需要改动IBinderPoolService。
