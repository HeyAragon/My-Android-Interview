## 四大组件之Service
### Service

#### Service简介

`Service`是长期运行在后台的应用程序组件，`Service`不是一个单独的进程，它和应用程序在同一个进程中，`Service`也不是一个线程，它和线程没有任何关系，并且`Service`运行在主线程中，所以它不能执行耗时操作。如果直接把耗时任务放在`Service`的`onStartCommand()`中，很容易引起ANR，如果有耗时操作必须开启一个单独的线程来处理。

#### Service两种启动方式

- **启动**

  当应用组件（如Activity）通过调用`StartService()`启动服务时，Service处于启动状态。一旦启动成功，Service即可再后台无限期运行，即使启动服务的组件已被销毁也不受影响。已启动的Service通常是执行单一操作，而且不会将结果返回给调用方。

  - **<u>生命周期</u>：**

    `onCreate()`-->`onStartCommand()`-->`onDestroy()`

  > `onCreate()`只会调用一次，若服务启动之后，再次启用服务则只会调用`onStartCommand()`。

- **绑定**

  当应用组件通过调用`bindService()`绑定到服务时，服务处于“绑定”状态。绑定服务提供了一个客户端-服务器接口，允许组件与服务进行交互、发送请求、获取结果，甚至是利用进程间通信(IPC)跨进程执行这些操作。仅当与另一个组件绑定时，绑定服务才会运行。多个组件可以同时绑定到该服务，但全部取消后，该服务才会被销毁。

  -  **<u>流程</u>：**
     1. 创建`BindService`服务，继承`Service`并在类中创建一个实现`IBinder`接口的实例对象并提供公共方法供客户端调用。
     2. 在`BindService`的`onBind()`方法中返回步骤一中声明的Binder对象。
     3. 在客户端中声明`ServiceConnection`类，并重写其`onServiceConnected()`方法，用于在此方法回调中接收来自`BindService`传递过来的`IBinder`对象，用于与`Service`交互。
     4. 在客户端调用`bindService()`绑定`Service`。
  - **<u>生命周期</u>：**

    `onCreate()-`->`onBind()`-->`（onServiceConnected()非Service中方法）`-->`onUnbind()`-->`onDestroy()`

> 记住这两种服务可以同时以这两种方式运行，也就是说它既可以是启动服务（以无限期运行），也允许绑定。问题只在于是否实现了一组毁掉方法：`onStartCommand()`（允许组件启动服务）和`onBind()`（允许绑定服务）。


### IntentService

#### IntentService简介

`IntentService`是继承于`Service`并处理异步请求的一个类，在`IntentService`内有一个工作线程`HandlerThread`来处理耗时操作，启动`IntentService`方式和启动传统`Service`一样，同时，当任务执行完成之后，`IntentService`会自动停止，而不是我们去手动控制。另外，可以启动I`IntentService`多次，而且每一个耗时操作会以工作队列的方式在`IntentService`的`onHandleIntent()`回调方法中执行，并且每次只会执行一个工作线程，执行完第一个再执行第二个，以此类推。

#### IntentService作用

- 创建默认的工作线程，用于再应用的主线程外执行传递给`onStartCommand()`的所有`Intent`。
- 创建工作队列，用于将`Intent`逐一传递给`onHandleIntent()`实现，无需处理多线程问题。
- 所有请求处理完成后，`IntentService`会自动停止，无需调用`stopSelf()`方法停止`Service`。
- 为`Service`提供`onBind()`默认实现，返回null。
- 为`Service`提供`onStartCommand()`的默认实现，可将 Intent 依次发送到工作队列和 `onHandleIntent()` 实现。

## 四大组件之Broadcast Receiver

### 广播定义

在Android中，Broadcast是一种广泛运用的在应用程序之间传输信息的机制，Android中我们要发送的广播内容是一个`Intent`，这个`Intent`中可以携带我们要传送的数据。

### 广播使用场景

- 同一app具有多个进程的不同组件之间的消息通信
- 不同app之间的组件之间消息通信
- 与`Android`系统再特定情况下的通信（如电话呼入时、网络可用时）

### 实现原理

**采用的模型**

`Android`中的广播使用了**观察者模式**：基于消息的发布/订阅事件模型。

因此，Android将广播的**发送者**和**接受者**解耦，使得系统方便集成，更易扩展。

**模型中角色**

1. 消息订阅者（广播接收者）
2. 消息发布者（广播发布者）
3. 消息中心（`AMS`,即`Activity Manager Service`）

**示意图&原理**

![image](https://github.com/HeyAragon/My-Android-Interview/blob/master/Android_Interview_IMG/broadcast_receiver_model.webp)

### 广播注册方式

- **静态注册**

  静态注册，再

- **动态注册**



	

