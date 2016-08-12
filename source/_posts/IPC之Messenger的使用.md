title: IPC之Messenger的使用
date: 2016-05-28 09:24:38
tags:
categories:
- Android
---

>IPC之Messenger的使用

## Android系统为什么要提供Messenger?
Messenger是一种轻量级的IPC方案，系统已经为我们封装了Binder的实现细节，也要用到AIDL。通过Messenger传递进程间的数据可以让们更方便的使用。
<!-- more -->
## Messenger的不足：
Messenger 传递的数据格式收到限制，Messenger传递的数据需要Handler进行处理，所以数据必须要放入Message中，Message中的数据只能是what,arg1,arg2,Bundle,以及replyTo,Object在2.2以后只能传递系统提供的parcelable子类，不能传递自定义parcelable子类。主要还是Bundle提供的较多的数据类型。

Messenger为串行处理客户端请求，无法处理大量并发。Messenger用来传递消息，而不是来处理请求，更不能跨进程调用服务器的方法。


## 客户端服务端的实现：

客户端实现：
```java
public class MessengerActivity extends Activity {
	private Messenger mService;
	private Messenger mGetReplyMessenger = new Messenger(new MessengerHander());
	private static class MessengerHander extends Handler{
		@Override
		public void handleMessage(Message msg) {
			switch (msg.what) {
			case MyMessgeConst.MSG_FROM_SERVICE:
			    Log.i("MessengerActivity",msg.getData().getString("reply"));
				break;
			}			
		}
	}

	private ServiceConnection mConnection = new ServiceConnection() {
		@Override
		public void onServiceDisconnected(ComponentName name) {
			// TODO Auto-generated method stub
		}

		@Override
		public void onServiceConnected(ComponentName name, IBinder service) {
			mService = new Messenger(service);//用绑定返回的IBinder对象创建Messenger
			Message msg = Message.obtain(null,MyMessgeConst.MSG_FROM_CLIENT);
			Bundle dataBundle  = new Bundle();
			dataBundle.putString("msg", "hello service ,this is client");
			msg.setData(dataBundle); //通过序列化对象Bundle封装数据

			//客户端发送消息的时候需要把 “接收服务器端回复的Messenger对象” 绑定到Message对象传给服务器
			msg.replyTo=mGetReplyMessenger; //绑定接收服务端返回的Messenger 这个Messenger的创建不需要传入IBinder对象
			try {
				mService.send(msg);
			} catch (RemoteException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	};

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		// TODO Auto-generated method stub
		super.onCreate(savedInstanceState);
		setContentView(R.layout.messenger);
		Intent intent = new Intent(this,MessengerService.class);
		bindService(intent, mConnection, Context.BIND_AUTO_CREATE);//绑定服务端的Service
	}
}

```
服务端实现
```java

public class MessengerService extends Service {

	private static class MessengerHander extends Handler{

		@Override
		public void handleMessage(Message msg) {
			switch (msg.what) {
			case MyMessgeConst.MSG_FROM_CLIENT:
				Log.i("MessengerService", msg.getData().getString("msg"));
				Messenger client = msg.replyTo;
				Message replyMessage = Message.obtain(null,MyMessgeConst.MSG_FROM_SERVICE);
				Bundle bundle = new Bundle();
				bundle.putString("reply", "hello client, i receive your message");
				replyMessage.setData(bundle);
				try {
					client.send(replyMessage);
				} catch (RemoteException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
				break;
			}			
		}

	}
	private static Messenger mMessenger = new Messenger(new MessengerHander());
	//通过创建Handler对象创建一个Messenger对象处理客户端的请求。
	@Override
	public IBinder onBind(Intent intent) {
		return mMessenger.getBinder();//通过onBind的方法返回底层的Binder供客户端使用
	}
}
```

## Messenger构造函数

```java
/**
 * Create a new Messenger pointing to the given Handler.  Any Message
 * objects sent through this Messenger will appear in the Handler as if
 * {@link Handler#sendMessage(Message) Handler.sendMessage(Message)} had
 * been called directly.
 *
 * @param target The Handler that will receive sent messages.
 */
public Messenger(Handler target) {
    mTarget = target.getIMessenger();
}
/**
 * Create a Messenger from a raw IBinder, which had previously been
 * retrieved with {@link #getBinder}.
 *
 * @param target The IBinder this Messenger should communicate with.
 */
public Messenger(IBinder target) {
    mTarget = IMessenger.Stub.asInterface(target);
}
```
Messgenger提供了两个重载的构造函数，其中一个传入一个IBinder对象,如果跨进程通讯，就通过asInterface方法获取获得服务端的proxy对象。
