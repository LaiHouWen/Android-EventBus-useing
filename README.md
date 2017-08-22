# Android-EventBus-useing

Android EventBus 事件总线的介绍及使用

EventBus 是一个耦合度很低的框架。是一个消息总线，以观察者模式实现，用于简化程序的组件、线程通信，可以轻易切换线程、开辟线程。
EventBus3.0跟先前版本的区别在于加入了annotation @Subscribe，取代了以前约定命名的方式。

在Android studio 工具编写代码，在工程的build.gradle添加 对EventBus的引用

compile 'org.greenrobot:eventbus:3.0.0' 


1.使用EventBus3.0 的步骤

public class MessageEvent {
    public final String message;

    public MessageEvent(String message) {
        this.message = message;
    }
}

2.准备订阅者

// This method will be called when a MessageEvent is posted
    @Subscribe
    public void onMessageEvent(MessageEvent event){
        Toast.makeText(getActivity(), event.message, Toast.LENGTH_SHORT).show();
    }
或
@Subscribe(threadMode = ThreadMode.MAIN)  
public void onMessageEvent(MessageEvent event) {/* Do something */};

3.发送事件

 EventBus.getDefault().post(new MessageEvent("Hello everyone!"));
 
4.ThreadMode线程通信

EventBus可以很简单的实现线程间的切换，包括后台线程、UI线程、异步线程

ThreadMode.POSTING
  //默认调用方式，在调用post方法的线程执行，避免了线程切换，性能开销最少    
    // Called in the same thread (default)
    @Subscribe(threadMode = ThreadMode.POSTING) // ThreadMode is optional here
    public void onMessage(MessageEvent event) {
        log(event.message);
    }

ThreadMode.MAIN

ThreadMode.BACKGROUND
 // 如果调用post方法的线程不是主线程，则直接在该线程执行
 // 如果是主线程，则切换到后台单例线程，多个方法公用同个后台线程，按顺序执行，避免耗时操作

ThreadMode.ASYNC
  //开辟新独立线程，用来执行耗时操作，例如网络访问
  //EventBus内部使用了线程池，但是要尽量避免大量长时间运行的异步线程，限制并发线程数量
  //可以通过EventBusBuilder修改，默认使用Executors.newCachedThreadPool()


5.配置EventBusBuilder
EventBus提供了很多配置，一般的情况下我们可以不用配置.但是，如果你有一些其他要求,比如控制日志在开发的时候输出,发布的时候不输出，
在开发的时候错误崩溃，而发布的时候不崩溃...等情况。

    EventBus eventBus = new EventBus();
    //下面这一条的效果是完全一样的
    EventBus eventBus = EventBus.builder().build();
    //修改默认实现的配置，记住，必须在第一次EventBus.getDefault()之前配置，且只能设置一次。建议在application.onCreate()调用
    EventBus.builder().throwSubscriberException(BuildConfig.DEBUG).installDefaultEventBus();


6.StickyEvent
StickyEvent在内存中保存最新的消息，取消原有消息，执行最新消息，只有在注册后才会执行，如果没有注册，消息会一直保留来内存中

    //在注册之前发送消息
    EventBus.getDefault().postSticky(new MessageEvent("Hello everyone!"));

//限制，新界面启动了
   @Override
    public void onStart() {
        super.onStart();
        EventBus.getDefault().register(this);
    }
    //在onStart调用register后，执行消息
    @Subscribe(sticky = true, threadMode = ThreadMode.MAIN)
    public void onEvent(MessageEvent event) {
        // UI updates must run on MainThread
        textField.setText(event.message);
    }

    @Override
    public void onStop() {
        EventBus.getDefault().unregister(this);
        super.onStop();
    }

priority事件优先级

  //priority越大，级别越高
    @Subscribe(priority = 1);
    public void onEvent(MessageEvent event) {
    …
    }
    
    // 中止事件传递，后续事件不在调用,注意，只能在传递事件的时候调用
    @Subscribe
    public void onEvent(MessageEvent event){
        …
        EventBus.getDefault().cancelEventDelivery(event) ;
    }



7.混淆代码

-keepattributes *Annotation*

-keepclassmembers class ** {
    @org.greenrobot.eventbus.Subscribe <methods>;
}
    
-keep enum org.greenrobot.eventbus.ThreadMode { *; }

# Only required if you use AsyncExecutor

-keepclassmembers class * extends org.greenrobot.eventbus.util.ThrowableFailureEvent {

    <init>(java.lang.Throwable);
    
}








