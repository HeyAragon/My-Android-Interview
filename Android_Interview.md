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

### 一.广播定义

在Android中，Broadcast是一种广泛运用的在应用程序之间传输信息的机制，Android中我们要发送的广播内容是一个`Intent`，这个`Intent`中可以携带我们要传送的数据。

[]: https://www.jianshu.com/p/ca3d87a4cdf3	"广播"

### 二.广播使用场景

- 同一app具有多个进程的不同组件之间的消息通信
- 不同app之间的组件之间消息通信
- 与`Android`系统再特定情况下的通信（如电话呼入时、网络可用时）

### 三.实现原理

1. **采用的模型**

   `Android`中的广播使用了**观察者模式**：基于消息的发布/订阅事件模型。

   因此，Android将广播的**发送者**和**接受者**解耦，使得系统方便集成，更易扩展。

2. **模型中角色**

   1. 消息订阅者（广播接收者）
   2. 消息发布者（广播发布者）
   3. 消息中心（`AMS`,即`Activity Manager Service`）

3. **示意图&原理**

   ![image](https://github.com/HeyAragon/My-Android-Interview/blob/master/Android_Interview_IMG/broadcast_receiver_model.webp)

   ![image](C:\Users\Administrator\Desktop\Android Interview\Android_Interview_IMG\broadcast_receiver_model.webp)

### 四.广播注册方式

1. **静态注册**

   静态注册，在`AndroidManifest.xml`清单文件中`application`节点下使用`receiver`注册。

   静态注册依附于清单文件，只要APP启动过一次，所静态注册的广播就会生效，无论当前的APP处于停止使用还是正在使用状态。只要相应的广播事件发生，系统就会遍历所有的清单文件，通知相应的广播接收者接收广播，然后调用广播接收者的onReceiver方法。

    以监听手机打电话为例：

   ```xml
     <uses-permission android:name="android.permission.PROCESS_OUTGOING_CALLS"/>
      <receiver android:name=".MyBroadcastReceiver">
           <intent-filter>
                <action android:name="android.intent.action.NEW_OUTGOING_CALL" />
            </intent-filter>
     </receiver>
   ```

2. **动态注册**

   在代码中动态注册。动态注册方式依赖于所注册的组件，当APP关闭后，组件对象都不在了动态注册的代码都不存在了，所动态注册监听的action自然不在生效。

     以监听屏幕点亮与关闭为例子：

   ```java
     public class MainActivity extends Activity {
     
         private MyBroadcastReceiver receiver ;
         @Override
         protected void onCreate(Bundle savedInstanceState) {
             super.onCreate(savedInstanceState);
             setContentView(R.layout.activity_main);
             registerMyReceiver();//在activity创建的时候进行注册监听
         }
     
         private void registerMyReceiver() {
             receiver = new MyBroadcastReceiver();
             IntentFilter filter = new IntentFilter();//创建IntentFilter对象
             filter.addAction(Intent.ACTION_SCREEN_OFF);//IntentFilter对象中添加要接收的关屏广播
             filter.addAction(Intent.ACTION_SCREEN_ON);//添加点亮屏幕广播
             registerReceiver(receiver, filter);
         }
     
         private void unRegisterMyReceiver(){
             if(receiver != null){
                 unregisterReceiver(receiver);//注销广播接收者，使其不起作用
             }
         }
     }
   ```

### 五.广播分类

1. **普通广播（Normal Broadcast）**

   即 开发者自身定义 `intent`的广播（最常用）。发送广播使用如下：

   ```java
   Intent intent = new Intent();
   //对应BroadcastReceiver中intentFilter的action
   intent.setAction(BROADCAST_ACTION);
   //发送广播
   sendBroadcast(intent);
   ```
   > **Note:**若被注册了的广播接收者 中注册时`intentFilter`的`action`与上述匹配，则会接收此广播（即进行回调`onReceive()`）。

   如下`mBroadcastReceiver`则会接收上述广播：

   ```xml
   <receiver 
       //此广播接收者类是mBroadcastReceiver
       android:name=".mBroadcastReceiver" >
       //用于接收网络状态改变时发出的广播
       <intent-filter>
           <action android:name="BROADCAST_ACTION" />
       </intent-filter>
   </receiver>
   ```

2. **系统广播（System Broadcast）**

   - Andorid中内置了多个系统广播：只要涉及到手机的基本操作（如开机、网络状态变化、拍照等等）系统都会发出相应的广播。

   - 每个广播都有特停的`Intent-Filter`(包括具体的Action),Android[系统广播](https://www.jianshu.com/p/ca3d87a4cdf3)部分如下：

     | 系统操作                                       | action                              |
     | ---------------------------------------------- | ----------------------------------- |
     | 关闭或打开飞行模式                             | Intent.ACTION_AIRPLANE_MODE_CHANGED |
     | 关闭或打开飞行模式                             | Intent.ACTION_AIRPLANE_MODE_CHANGED |
     | 充电时或电量发生变化                           | Intent.ACTION_BATTERY_CHANGED       |
     | 电池电量充足（即从电量低变化到饱满时会发出广播 | Intent.ACTION_BATTERY_OKAY          |
     | 系统启动完成后(仅广播一次)                     | Intent.ACTION_BOOT_COMPLETED        |
     | 按下照相时的拍照按键(硬件按键)时               | Intent.ACTION_CAMERA_BUTTON         |
     | 屏幕锁屏                                       | Intent.ACTION_CLOSE_SYSTEM_DIALOGS  |
     | 插入耳机时                                     | Intent.ACTION_HEADSET_PLUG          |
     | 设备当前设置被改变时(界面语言、设备方向等)     | Intent.ACTION_CONFIGURATION_CHANGED |

3. **有序广播(Ordered Broadcast)**

   - **发送出去的广播被广播接收者按照先后顺序接收。**

        > **Note:** 有序是针对广播接收者而言.

      - **广播接收者接收广播的顺序规则（同时面向静态和动态注册的广播接收者）**

        1. 按照`Priority`属性值从大-小排序,取值范围`（-1000-1000）`；
        2. `Priority`属性相同者，动态注册的广播优先；

      - **特点**

        1. 接收广播按顺序接收；
        2. 先接收的广播接收者可以对广播进行截断（调用`abortBroadcast()`），即后接收的广播接收者不再接收到广播；
        3. 先接收的广播接收者可以对广播进行修改，那么后接收的广播接收者将接收到被修改后的广播。

      - **具体使用**

        有序广播的使用过程与普通广播非常类似，差异仅在于广播的发送方式：

        ```java
        sendOrderedBroadcast();
        ```

4. **App应用内广播（Local Broadcast）**

   - **背景**

     `Android`中的广播可以跨App直接通信（`exported`对于有`intent-filter`情况下默认值为`true`）

   - **冲突（可能出现的问题）**

     1. 其他App针对性发出与当前App intent-filter相匹配的广播，由此导致当前App不断接收广播并处理；
     2. 其他App注册与当前App一致的intent-filter用于接收广播，获取广播具体信息；
        即会出现安全性 & 效率性的问题。

   - **解决方案**

     使用App应用内广播（Local Broadcast）

     > 1. App应用内广播可理解为一种局部广播，广播的发送者和接收者都同属于一个App。
     > 2. 相比于全局广播（普通广播），App应用内广播优势体现在：安全性高 & 效率

   - **具体使用1——将全局广播设置成局部广播：**

     1. 注册广播时将exported属性设置为*false*，使得非本App内部发出的此广播不被接收；
     2. 在广播发送和接收时，增设相应权限permission，用于权限验证；
     3. 发送广播时指定该广播接收器所在的包名，此广播将只会发送到此包中的App内与之相匹配的有效广播接收器中。

     > 通过**intent.setPackage(packageName)**指定包名

   - **具体使用2 - 使用封装好的LocalBroadcastManager类**

     使用方式上与全局广播几乎相同，只是注册/取消注册广播接收器和发送广播时将参数的context变成了LocalBroadcastManager的单一实例。

     > **Note：**对于LocalBroadcastManager方式发送的应用内广播，只能通过LocalBroadcastManager动态注册，不能静态注册。

     ```java
     //注册应用内广播接收器
     //步骤1：实例化BroadcastReceiver子类 & IntentFilter mBroadcastReceiver 
     mBroadcastReceiver = new mBroadcastReceiver(); 
     IntentFilter intentFilter = new IntentFilter(); 
     
     //步骤2：实例化LocalBroadcastManager的实例
     localBroadcastManager = LocalBroadcastManager.getInstance(this);
     
     //步骤3：设置接收广播的类型 
     intentFilter.addAction(android.net.conn.CONNECTIVITY_CHANGE);
     
     //步骤4：调用LocalBroadcastManager单一实例的registerReceiver（）方法进行动态注册 
     localBroadcastManager.registerReceiver(mBroadcastReceiver, intentFilter);
     
     //取消注册应用内广播接收器
     localBroadcastManager.unregisterReceiver(mBroadcastReceiver);
     
     //发送应用内广播
     Intent intent = new Intent();
     intent.setAction(BROADCAST_ACTION);
     localBroadcastManager.sendBroadcast(intent);
     ```

5. **粘性广播（Sticky Broadcast）**

   由于在`Android5.0 & API 21`中已经失效，所以不建议使用，在这里也不作过多的总结。

### 六.特别注意

对于不同注册方式的广播接收器回调`OnReceive（Context context，Intent intent）`中的`context`返回值是不一样的：

- 对于静态注册（全局+应用内广播），回调`onReceive(context, intent)`中的`context`返回值是：`ReceiverRestrictedContext`；
- 对于全局广播的动态注册，回调`onReceive(context, intent)`中的`context`返回值是：`Activity Context`；
- 对于应用内广播的动态注册（`LocalBroadcastManager`方式），回调`onReceive(context, intent)`中的`context`返回值是：`Application Context`。
- 对于应用内广播的动态注册（非`LocalBroadcastManager`方式），回调`onReceive(context, intent)`中的context返回值是：`Activity Context`；





