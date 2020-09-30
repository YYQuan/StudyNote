#  计划：

1. OKHttp
2. Retrofit
3. Rxjava
4. glide 
5. ButterKnife

# OkHttp



## 一.基础使用方法





### Get

```JAVA
OkHttpClient client = new OkHttpClient();
String run (String url) throws IOException{
	Request request = new Request.Builder()
        .url()
        .build();
    
    //同步
    try(Response response =  client.newCall(request).execute()){
        return response.body().string();
	}
    
    // 异步
    //
    /**
    try{
        client.newCall(request).enqueue(new Callback(){
            @Onverride
            public void onFailure(Call call,IOException e){
                
            }
            
            @Override
            public void onResponse(Call call, Response response)  throws IOException{
                
}
        })
    }
    */
}
```





### POST

```java
public static final MediaType JSON  = MediaType.get("application/json;charset=utf-8");

OkHttpClient client = new  OkHttpClient();
String post(String url,String json) throws IOExcetion{
    RequestBody request = new RequestBody.Builder()
        .url(url)
        .post(body)
        .build();
    
    try(Response response = client.newCall(request).execute()){
        return response.body().string();
}
    
}
```



从这两个用法可以看出来发起网络请求分为了以下的几步：

1. 获取OkHttpClient
2. 构建Request
3. 把Request传入OkHttpClient中构建出 Call
4. call调用execute ()或者enqueue()来完成同步或者异步的调用
   返回Response

总结下 涉及到的对象
OkHttpClient: 就是Call的工厂，
           主要作用就是构建出call（其实Client还管web socket ）
Request:用户的具体请求 
RequestBody:构建Post的Request请求所需的参数
Call: 实际执行网络请求的对象
Response: 网络请求的返回对象

## 二.整体流程图

![img](https://img-blog.csdn.net/20180824142557322?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5MTUyMjQx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

  



整个流程还是很清晰的

1. 用户构建出Reuqest

2. 构建出call ： 具体的call由 用户传入的Request来决定

3. 执行call ：同步异步执行的方式是不同的。

4. 执行call的拦截器处理

   

## 三.流程详解

根据该流程来分析

1. 用户构建出Request
2. 构建出call ： 具体的call由 用户传入的Request来决定
3. 执行call ：同步异步执行的方式是不同的。
4. 执行call的拦截器处理

### 构建出Reuqest

![image-20200814115035793](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814115035793.png)

没有太特别的就是通过Request的构建者Reuqest.Builder 存储了一个HTTP 请求 所需要的变量： 请求地址、请求头、请求体、请求方式（get/post）等等


### 构建出Call

先看看Call的源码
![image-20200814112947900](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814112947900.png)

可以看出来Call是一个接口，接口中最重要的就是execute 和enqueue函数

另外其 request() 的返回值其实就是OkHttpClient.newCall(Request) 中传入的Request对象

Call的具体实现类是RealCall

![image-20200814113118484](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814113118484.png)

按着流程从Call的入口开始看

```java
Call call =  client.newCall(Request);
```

实际上的调用的就是     ![image-20200814114038101](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814114038101.png)![image-20200814114015394](https://i.loli.net/2020/09/24/YNAWEaLseHp5GJB.png)

这样Call的实现类就已经创建出来了

### 执行Call

Call创建出来之后， 接着来看执行Call的部分。
执行Call分为了同步 excecute 和 异步enqueue 两个部分。

#### 同步执行exceute

源码
![image-20200814120259213](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814120259213.png)

 整段代码的核心在红色框内

client就是OkHttpClient

![image-20200814120639942](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814120639942.png)

源码中可以看出，RealCall的execute的核心就是
用client的dispatcher 来分发自己。
然后通过getResponseWithInterceptorChain() 来得到返回的。

![image-20200814121026117](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814121026117.png)

originalRequest就是用户传入的request。

所以说execute的关键就是
client的dispatcher的execute 和  拦截链的proceed 

先来看 client的Dispatcher 是啥，拦截链的process 放在拦截链中去讲。

Dispatcher是OkHttp3的任务调度核心类，负责管理同步和异步的请求，以及每一个请求的状态。并且其内部维护了一个线程池用于执行相应的请求


接下来看看源码， Dispatcher 内部维护了三个队列

![image-20200814122341878](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814122341878.png)

一个正在跑着的同步队列， 一个正在跑着的异步队列，还有一个异步的等待队列（把未能及时执行的任务保存在其中，等到正在跑的异步队列有空闲的时候再执行）

PS： 这队列的实现方式都是 双向队列 ，支持从队列的两端检索和插入元素。

接着回到 dispatcher.execute(RealCall)；
![image-20200814122744480](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814122744480.png)

原来现实就是把call添加到正在运行的同步队列中



dispatcher.execute() 运行完了之后
会调用dispatcher.finish();

![image-20200814141350977](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814141350977.png)



![image-20200814141227750](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814141227750.png)

dispatcher.finish的核心作用就是将 call 从队列中移除





#### 异步执行enqueue

接着来看异步任务

![image-20200814142315986](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814142315986.png)

主流程和同步类似。 但是少了  拦截链的处理和 dispatcher.process
估计都放在回调当中去做了。

但是使用的不是RealCall了，而是AsyncCall

![image-20200814142718257](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814142718257.png)

AsyncCall是RealCall的内部类。

![image-20200814142854444](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814142854444.png)

AsyncCall继承了NamedRunnable，NamedRunnable是实现了Runnable接口的。
先继续往下看。



dispatcher.enqueue(AsyncCall)里面去做了啥

![image-20200814143452853](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814143452853.png)

enqueue里就是把call 放在准备队列，或者异步队列当中然后执行

现在看看 excutorService().execute(call)是啥

excutoeService()就是获取dispatcher 维护的一个线程池

![image-20200814143626447](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814143626447.png)

然后通过这个线程池去执行AsyncCall这个runnable.

那就需要回到AsyncCall的run函数里看做了什么了。

![image-20200814145441222](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814145441222.png)

发现run没做什么特别的就只执行了execute函数，接下来看 AsyncCall的execute

![image-20200814144334582](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814144334582.png)

看到这里逻辑就很清晰了。异步和同步的逻辑基本是一样的。只是异步放在了线程池里去执行了。




### 拦截器处理

现在来看真正得到返回值的地方了。
同步和异步都会调用到的

![image-20200814145734308](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814145734308.png)

接下来看，getResponseWithInterceptorChain的源码

![image-20200814145931769](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814145931769.png)

可以看出 拦截器是由client和request 来决定的。

接着看执行的 chain.process(request)的代码。

![image-20200814150954769](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814150954769.png)



![image-20200814151043904](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814151043904.png)

 chain.process(request);里的有效操作是调用第一个拦截器的intercept（）

![image-20200814152721160](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814152721160.png)

从标注出可以判断是从第一个拦截器开始调用。

首先调用的拦截器是用户自定义的拦截器，不固定。而最终获取数据的拦截器是固定的，是CallServerInterceptor 

从 interceptorChain 这个名字来看 这应该是一个责任链
先取最后调用的拦截器来看 连接拦截器 CallServerInterceptor

![image-20200814153201782](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814153201782.png)

这里通过http流取到了网络中的返回值。

接着看看其他的拦截器 比如ConnectInterceptor (连接拦截器)

![image-20200814151828997](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814151828997.png)

发现是继续调用了chain的proceed()

通过 chain.proceed()->intercept.intercept()->chain.proceed()....
这个模式看判断这是一个责任链模式。





PS：拦截器的主要类型

![image-20200814153552248](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814153552248.png)



TIPS. 拦截器的生效是在  getResponseWithInterceptorChain里的，所以对于异步执行来说。拦截器的代码是执行在子线程当中的。



## 其他细节



### 等待队列的成员是怎么移动到异步队列当中的

asyncCall  finish的时候会检测，会通过promoteCalls函数尽量的把等待队列的成员放到异步队列中去。



## 总结

new client -> new request -> new call(client,request)->

->同步-> dispatcher.execute->把realCall加入同步任务队列中
           -> 拦截链执行任务 获取数据
           -> 把realCall从同步任务队列中移除

->异步-> dispatcher.enqueue -> 异步任务队列是否满了
->没满-> 把asyncCall加入异步任务队列中
           -> 把任务加入到线程池当中
           -> 拦截链执行任务 获取数据
           -> 移除asyncCall
           -> 检测等待队列， 尽量把等待队列的元素移动到异步队列
->满了->把asyncCall 加入等待队列
          ->等待promoteCalls把asyncCall移到异步任务队列中





![image-20200814114220570](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814114220570.png)





OkHttp的默认线程池：

![image-20200814155440108](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814155440108.png)

SynchronousQueue 是同步队列，
OkHttp当中默认的队列是同步队列





# Retrofit

```java
implementation 'com.squareup.retrofit2:retrofit:2.9.0'
implementation 'com.squareup.retrofit2:converter-gson:2.0.2'
```

## 基础用法

![image-20200814170109965](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814170109965.png)



![image-20200814170155679](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814170155679.png)

![image-20200814170211832](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814170211832.png)



可以看出来 只要吧retrofit的实例创建出来。
然后用retrofix.create.create(x.class),就能直接通过接口来得到
OkHttp的Call,得到call之后就能执行网络请求了

## 完整流程

一次通讯的流程如下：
![image-20200814171405150](https://i.loli.net/2020/09/24/byEGDZIFPANRMhc.png)

![image-20200817095309847](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817095309847.png)



## 源码解析

### Retrofit对象的构建

Retrofit的对象的构建是通过构建者模式 Build来创建。创建的过程中，需要传入各种的工厂来，帮助构建各种适配器来处理返回或者 处理各种平台的适配和返回值的转换等。

各种工厂和适配器先不看，先看主流程。

调用过程中，retrofit通过外观模式 统一的对 子类进行处理拿到子类对象。



### 接口实例

看看retrofit是怎么拿到接口实例的？

![image-20200814173147377](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814173147377.png)



看看具体的实现

![image-20200817103442863](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817103442863.png)

看下 validateServiceInterface(Class<?> service)
从名字来看，是验证传进来的class ,是不是合法的。

![image-20200817105659007](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817105659007.png)

上面的代码可以分成两个部分
1： 检测参数是否合法
2：loadServiceMethod 获取服务函数

对于1的检测是否合法，没啥好说的 
接下看看 获取服务函数的核心：loadServiceMethod函数

![image-20200817110232819](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817110232819.png)

首先是在缓存cache里找对象，  DCL 双重检查锁来保证线程安全的。
并且 cahce的数据结构是用 ConcurrentHashMap 来实现的。COncurrentHashMap是线程安全数据结构。

![image-20200817111346960](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817111346960.png)

回到 retrofit.create(Class)
validateServiceInterface(Class) 就是把 该结构的函数都添加到 cache当中。

接着往下看。
![image-20200817111700630](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817111700630.png)

看名称应该是动态代理
首先这个Proxy 代理类是 java.lang.reflect 下的。 不是retrofit特有的。
![image-20200817113211826](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817113211826.png)

这个是Proxy.newProxyInstance 中的备注。
也就是说 这个类是java通用的得到接口类的实例的函数。

![image-20200817115543072](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817115543072.png)

这个个Proxy的newProxyInstance的参数的说明

loadser 是 类加载器，
interfaces 是要被代理的接口列表
invocationHandler  被代理的类的函数被执行后的回调

回到retrofit.create
![image-20200817120051432](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817120051432.png)

红框内 是被代理的对象的被代理的函数被执行后的回调。
已开始的例子来说

![image-20200817120400603](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817120400603.png)

该函数的返回其实是动态代理Proxy中的InvokeHandler.invoke()的返回。
所以现在的重点 是看 InvacationHandler中的invoke的实现。
![image-20200817122048034](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817122048034.png)

重点就是这三句。 
platForm.isDefaultMethod(method)的意思就是判断有没有被实现。对于retrofit来说 一般是没有被实现的。
所以一般也就是执行了  loadServiceMethod(method).invoke(args);

跟踪loadServiceMethod(method).invoke(args);
![image-20200817122850069](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817122850069.png)

发现真正的对象是通过红框类的这句实现出来的。
实际上调用的也是他的 invoke 方法。

![image-20200817141141051](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817141141051.png)

1是验证参数的合法性
2才是真正的处理



HttpServiceMethod.parseAnnotations()的源码

```java
  /**
   * Inspects the annotations on an interface method to construct a reusable service method that
   * speaks HTTP. This requires potentially-expensive reflection so it is best to build each service
   * method only once and reuse it.
   */
  static <ResponseT, ReturnT> HttpServiceMethod<ResponseT, ReturnT> parseAnnotations(
      Retrofit retrofit, Method method, RequestFactory requestFactory) {
    boolean isKotlinSuspendFunction = requestFactory.isKotlinSuspendFunction;
    boolean continuationWantsResponse = false;
    boolean continuationBodyNullable = false;

    Annotation[] annotations = method.getAnnotations();
    Type adapterType;
    if (isKotlinSuspendFunction) {
      Type[] parameterTypes = method.getGenericParameterTypes();
      Type responseType =
          Utils.getParameterLowerBound(
              0, (ParameterizedType) parameterTypes[parameterTypes.length - 1]);
      if (getRawType(responseType) == Response.class && responseType instanceof ParameterizedType) {
        // Unwrap the actual body type from Response<T>.
        responseType = Utils.getParameterUpperBound(0, (ParameterizedType) responseType);
        continuationWantsResponse = true;
      } else {
        // TODO figure out if type is nullable or not
        // Metadata metadata = method.getDeclaringClass().getAnnotation(Metadata.class)
        // Find the entry for method
        // Determine if return type is nullable or not
      }

      adapterType = new Utils.ParameterizedTypeImpl(null, Call.class, responseType);
      annotations = SkipCallbackExecutorImpl.ensurePresent(annotations);
    } else {
      adapterType = method.getGenericReturnType();
    }

    CallAdapter<ResponseT, ReturnT> callAdapter =
        createCallAdapter(retrofit, method, adapterType, annotations);
    Type responseType = callAdapter.responseType();
    if (responseType == okhttp3.Response.class) {
      throw methodError(
          method,
          "'"
              + getRawType(responseType).getName()
              + "' is not a valid response body type. Did you mean ResponseBody?");
    }
    if (responseType == Response.class) {
      throw methodError(method, "Response must include generic type (e.g., Response<String>)");
    }
    // TODO support Unit for Kotlin?
    if (requestFactory.httpMethod.equals("HEAD") && !Void.class.equals(responseType)) {
      throw methodError(method, "HEAD method must use Void as response type.");
    }

    Converter<ResponseBody, ResponseT> responseConverter =
        createResponseConverter(retrofit, method, responseType);

    okhttp3.Call.Factory callFactory = retrofit.callFactory;
    if (!isKotlinSuspendFunction) {
      return new CallAdapted<>(requestFactory, callFactory, responseConverter, callAdapter);
    } else if (continuationWantsResponse) {
      //noinspection unchecked Kotlin compiler guarantees ReturnT to be Object.
      return (HttpServiceMethod<ResponseT, ReturnT>)
          new SuspendForResponse<>(
              requestFactory,
              callFactory,
              responseConverter,
              (CallAdapter<ResponseT, Call<ResponseT>>) callAdapter);
    } else {
      //noinspection unchecked Kotlin compiler guarantees ReturnT to be Object.
      return (HttpServiceMethod<ResponseT, ReturnT>)
          new SuspendForBody<>(
              requestFactory,
              callFactory,
              responseConverter,
              (CallAdapter<ResponseT, Call<ResponseT>>) callAdapter,
              continuationBodyNullable);
    }
  }

```



HttpServiceMethod.parseAnnotations的注释

![image-20200817141439488](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817141439488.png)



由此能大致判断，这里就是处理Retrofit的注解的 并且声称对应函数的地方了。

通过HttpServiceMethod.parseAnnotations就把下面这个NetService的serviceApi这个接口给实现了。

![image-20200814170109965](E:/tools/Typora/res/andoirdCodeAnaly/image-20200814170109965.png)

简化下 HttpServiceMethod.parseAnnotations()

![image-20200817142716863](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817142716863.png)

发现主要流程就是3个：

1. 创建CallAdapter
2. 创建Converter
3. 根据calladapter 和 converter,以及 请求工厂和call工厂来创建要返回的 ServiceMethod 对象

现在已经得到了ServiceMethod,
CallAdapted 其实就是HttpServiceMethod的子类，CallAdapater的invoke函数的实现就是HttpServiceMethod
然后先不管CallFactory 和ConverFactory这两个点内部的具体行为，先接着主处理。
动态代理执行的是ServiceMethod的invoke()
接下来看ServiceMethod的invoke里做了什么。

ServiceMethod.invoke的源码
![image-20200817143552315](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817143552315.png)

![image-20200817143636073](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817143636073.png)

对于java的正常流程来说，只会执行到CallAdapted的。对于主流程的源码解析来说，只需要看CallAdapter即可

源码如下

![image-20200817144008201](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817144008201.png)



从这可以看出，这是ServiceMethod.parseAnnotation中传入的
![image-20200817144754281](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817144754281.png)

因此 往 HttpServiceMethod.parseAnnotations()中的createCallAdapter里面去看。

![image-20200817144944712](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817144944712.png)

![image-20200817145006446](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817145006446.png)

![image-20200817145310794](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817145310794.png)

![image-20200817145347697](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817145347697.png)

发现就是构建retrofit时传入的callFactory或者 默认的callFactory来完成的。

callAdapterFatories 在用户的callFactory后会追加默认的callFactory。

![image-20200817145752411](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817145752411.png)

另外 有多个callFactory时，只要有一个工厂能够创建出CallAdapater即可

![image-20200817145708821](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817145708821.png)



接下来看默认的CallFactory中是怎么创建出CallAdapter的。

![image-20200817150526590](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817150526590.png)



找到了默认情况下的 CallAdapter的adapt的处理的地方了。



再回到动态代理中
Proxy中的InvocationHandler()的invoke的实现里执行的  loadServiceMethod(method).invoke(args)
在默认情况下就是执行 

![image-20200817151537024](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817151537024.png)

![image-20200817151654908](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817151654908.png)

adapt(call) 的就是从HttpServiceMethod.invoke中得到的。

PS：这里的Call和OKHttp里的是不一样的。
CallAdapater中 无论是返回 call ,还是   executorCallbackCall 主流程都是类似的，先接着看主流程。
以返回call 来继续。

回到主流程

retrofit.create返回的是一个动态代理。netApi.serviceApi执行后，返回的其实是HttpServiceMethod.parseAnnotations().invoke()的返回 OkHttpCall<T>了

![image-20200817154600852](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817154600852.png)



到此接口的实例就已经创建出来了。
虽然前面还有个坑还没有讲。conver 工厂 的创建。conver工厂在解析 okHttp3.execute的按返回的时候会用到，放在那讲。







接下来看OkHttpCall的 同步和异步的执行。

### 接口的执行

![image-20200817155705182](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817155705182.png)

接口的执行在这里。
先分同步和异步。

#### 同步

看OkHttpCall的同步

![image-20200817155802194](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817155802194.png)

可以看到 ，这里最终调用的还是OKHttp的call.execute，且OkHttpCall是通过getRawCall来转化成OkHttp.Call的

![image-20200817155959951](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817155959951.png)

![image-20200817160109358](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817160109358.png)

![image-20200817160230229](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817160230229.png)

![image-20200817160547085](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817160547085.png)



所以OkhttpCall里面有 构建retrofit时的callFactory和requestFactory  callFactory 构建成OKhttp3.call,
requestFactory 构建OkHttp3.reqeust
实际上OkHttpCall就是一个 OkHttp3.Call的代理。

CallFactory 构建OkHttp3.call 比较简单，实际上就是调用OKHttp3.Client.newCall

但Request的构建就麻烦一些。

看reqeustFactory .create构建OkHttp3.Request的源码

![image-20200817173450562](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817173450562.png)



![image-20200817172725553](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817172725553.png)

![image-20200817172839534](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817172839534.png)

构建OkHttp.Request的时候 是借助ParameterHandler来添加参数的

![image-20200817175224245](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817175224245.png)

![image-20200817175317672](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817175317672.png)

ParamterHandler是设置Retrofit.RequestBuilder的相关参数的抽象类。其包括 body ，header等参数。
通过ParamterHandler把接口影响的参数都设置进来之后，Retrofit.RequestBuilder.get()才能完成的构建出OkHttp3.Request。



那现在的疑问就是ParameterHandler是怎么构建出来的

![image-20200817175906798](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817175906798.png)

![image-20200817180005038](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817180005038.png)

看起来和注解的参数有关系。

![image-20200817180326958](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817180326958.png)

Method.getParameterAnnotations:是拿参数的注解的。

例如：

![image-20200817180525956](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817180525956.png)

![image-20200817181025600](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817181025600.png)

这一步就是得到注解的参数的对应值。内部应该是通过反射来获取的。这不是重点。先不看了。
这样就能得到用户传入的参数，然后通过Retrofit.RequestBuild 来构建OkHttp3.Request了。





再看OkHttp3.Call.execute的返回

![image-20200817161730695](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817161730695.png)

![image-20200817161806322](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817161806322.png)

这个Response.success是Retrofit的Response不是OkHttp的。

![image-20200817162401400](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817162401400.png)



![image-20200817162244806](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817162244806.png)



通过源码可以看出
responseConverter.convert就是泛型转换的关键。

而这个responseConverter 就是HttpServiceMethod
中创建出来的

![image-20200817162954689](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817162954689.png)



接着往下看：
![image-20200817163018910](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817163018910.png)

![image-20200817163041303](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817163041303.png)

![image-20200817163130811](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817163130811.png)

这些过程和callFactory很类似。

GsonConverterFactory 是比较常用的 ConvertFactory。

看下 GsonConverterFactory 的

![image-20200817163744115](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817163744115.png)

![image-20200817163842672](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817163842672.png)

以上就是同步的流程
一般情况 ， CallFactory和CoverFactory都是别人实现好了的。



#### 异步



![image-20200817161007068](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817161007068.png)

类似同步， 最终也是通过OkHttp3.call来执行enqueue的。

![image-20200817164232405](E:/tools/Typora/res/andoirdCodeAnaly/image-20200817164232405.png)



异步和同步的返回值的泛型转换是一样的。不需要单独分析。

## 总结

​	retrofit 先通过构造者模式 创建出retrofit对象。并且设置好配置，比如CallFactoty, ConVertFactory 等参数。
CallFactory 可以控制 怎么执行OkHttpCall
ConvertFactory可以控制返回的值
CallFactory 和ConvertFactory 传入多个的时候，只有一个会生效，优先级是先加入的优先。

然后通过retrofig对象的craete函数来创建出接口动态代理。
等调用动态代理的函数时，实际上返回的是retrofit的invokeHandle的invoke函数。
一般默认情况下返回的就是OkHttpCall<T>。
这样就得到接口的实例了。
当接口的execute / enqueue 被执行的时候，实际上是被转化成为了OKhttp3.call去执行了。
然后把从OKHttp3中得到的Response 通过ConverFactory的conver函数来解析。解析出来的就是最终retrofit返回来的数据.

![image-20200817181634119](../../AppData/Roaming/Typora/typora-user-images/image-20200817181634119.png)



PS：对于 CallAdapter的 适配。其实最重要的就是适配把回调函数回到主线的方法。
在默认CallAdapter时，用的是Executor来切换。
而在Rx的Adapter的话就会用别的方式在切换。
但功能都是一样的都只是把OkHttpCall的异步回调切换为主线程中执行。

默认情况下的CallAdapter

![image-20200818115144287](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200818115144.png)

![image-20200818115230125](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200818115230.png)

![image-20200818115327401](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200818115327.png)

这部分就是 把持有主线程handle的Executor来实现主线程的切换。



# RxJava

参考：https://segmentfault.com/a/1190000019243389
作用：简化异步步骤。

## 基础使用

```java
//被观察
Observable<Integer> observable = Obserbable.create(new ObservableOnSubscribe<Integer>()){
	@Override 
    public void subscribe(IbservableEmitter<Integer> emitter throws Exception{
        
        emitter.onNext(1);
        emitter.onNext(2);
        emitter.onComplete();
    }
}
        
Obserber<Integer> obser = new Obserber<Integer>(){
    public void onSubscribe(Disposable d){
        
    }
    
    
    public  void onNext(Integer integer){
        
    }
    
    public void onError(Throwable e){
		
    }
    
    public void onComplete(){
        
    }
    
}
                          
observable.subscribe(observer);
```



整体的流程就是上流 observable  发出数据，
然后下游observer  进行观测。
observable 和observer 通过subscribe 函数联系起来



所以说，Observable 和observer 都是 负责定义的。 而subscirbe函数才是真正有执行动作的地方。 
所以我们从subscribe进去看一看.

## 源码解析

### Observable.subscribe

从Observable.subscribe(Observer) 入手来看源码。

![image-20200827110244651](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200827110245.png)

很容易就看到 , Observable的subscribe函数的核心
就是这两句。

RxJavaPlugins.onSubscribe(this, observer)这句对observer进行了一些处理之后，重新把返回值赋给了observer。

接着看RxJavaPlugin.onSubscribe()中做了什么



![image-20200827111714313](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200827111714.png)

看这个RxJavaPlugins.onSubscribe()的注释

看起来这是hook的工具类的入口
但是要RxJavaPlugins.onSubscribe() 要做处理的话，那么就需要onObservableSubscribe 不为null,

![image-20200827112811044](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200827112811.png)

这里可以看出来  RxJavaPlugins.onObservableSubscribe 是通过 setOnObserbableSubscribe来赋值的。
但是我们一般都不会调用这个。所以可以认为RxJavaPlugins.onSubscribe 就是直接返回了Observer。

回到Observable.subscribe()。

![image-20200827110244651](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200827110245.png)

接着看 subscribeActual

![image-20200827114932199](https://i.loli.net/2020/08/27/8eHXWoRdPZLaIOw.png)

从这 可以看出来， Observable.subscribe 实际上就是调用了 其实现类的subscribeActual（Observer ）

ps： Observable.subscribe不仅仅可以传 Observer,还可以传Consumer

![image-20200827115253953](https://i.loli.net/2020/08/27/tFgwuqD7oKmRePj.png)



![image-20200827115437401](https://i.loli.net/2020/08/27/Arq9u6dhlZRyvP4.png)

从这可以看出 Observable.subscribe(consumer )实际上也是执行了 Observable.subscribe(Observer)



### Observable.subscribeActual(Obserber)

上面以及分析出 Observable.subscribeActual(Observer) 取决于具体的实现类，所以要回到创建Obserbable的实例

```java
//被观察
Observable<Integer> observable = Observable.create(new ObservableOnSubscribe<Integer>()){
	@Override 
    public void subscribe(IbservableEmitter<Integer> emitter throws Exception{
        
        emitter.onNext(1);
        emitter.onNext(2);
        emitter.onComplete();
    }
}
```

所以要来看Observable.create()

![image-20200827120631948](https://i.loli.net/2020/08/27/P3a9FCnBU61WYk2.png)

--> RxJavaPlugins.onAssembly

![image-20200827121109931](https://i.loli.net/2020/08/27/g3s9T8rXGZ6Pnvm.png)

也是一个hook

![image-20200827121143421](https://i.loli.net/2020/08/27/8tjae6Sl1mPKRAi.png)

但是我们一般都没有额外设置 ，所以就是直接返回了source

所以说Observable的真正实现类就是 ObservableCreate

插一句： Observable.just 的具体实现类是 ObservableJust 其他创建Observable实例的方法可能有其他的对应的实现类。

所以，现在要知道Observable.subscribeActual()具体做了啥 ，对于Observable.create 来说 就是看，ObservableCreate.subscribeActual的实现。

接下来来看ObservableCreate.subscribeActual的具体实现。
先从整体的看下ObservableCreate

![image-20200827122828249](https://i.loli.net/2020/08/27/3g1eSrNU6wsBypD.png)



可以看出 ObservableCreate 唯一做了的事情就是 实现了subscribeActual
CreateEmitter 只是辅助ObservableCreate的



专注于主线

![image-20200827142050171](https://i.loli.net/2020/08/27/jpPcIdlrXxAfDiO.png)

源码中：

先把observer包装成对应的emitter
接着调用了下游的 observer.onSubscribe( emitter) 
这个是用户实现的
然后再调用了上游的 source(也就是上游的实例)的subscribe()
也就是用户自定义实现的部分。

source.subscribe(emitter)
所以对于ObservableCreate 这个实例来说， subscribeActual就做了两件事情，
1.执行用户实现的下游的observer的onSubScribe方法
2.执行用户实现的上游的observable的subScribe(emitter)
然后剩下的事情就交给emitter去处理。
对于Observable的emitter是CreateEmitter



因此需要看看CreateEmitter的实现

![image-20200827144319469](https://i.loli.net/2020/08/27/CiGTP6fpFySAcq7.png)

从结构来看，都是很熟悉的函数， 基本上从函数名称都能够知道函数的作用

看几个常用用的函数的实现

![image-20200827145736533](https://i.loli.net/2020/08/27/MBfpHKLscxjUP9z.png)其实就是调用了 下游Observer的对应函数。
subscribeActual 就解析到这里。

## 线程切换

上下游的基础使用只是RXjava的主题，线程切换才是RxJava的亮点和核心。

先来看看RxJava的线程切换的基础用法

```java
observable.subscribeOn(Scheddulers.newThread())
		  .observeOn(AndroidSchedulers.mainThread())
    	  .subscribe( observer );
```



调用了Observable的subscribeOn（Schedulers.newThread()）之后,上游的Observable.subscribe()代码就会切换到子线程中去执行，而 observeOn(AndroidScheduelers.mainThread()), 又会把下游代码切换到 主线程中执行。



接下来就从上游相关的切换开始分析

### 上游线程切换 subScribeOn

看看传入的这个subscribeOn(Schedulers.newThread())  这个scheduler 调度器是怎么起作用的

先看

![image-20200827153312373](https://i.loli.net/2020/08/27/3qoknEif5sWabGA.png)

![image-20200827153351420](https://i.loli.net/2020/08/27/vV1N94GuwftHaXR.png)

和之前的类似，一般我们并没设置onNewThreadHandler。
所以Schedulers.newThread() 就是返回了Schedulers的静态成员NEW_THREAD

![image-20200827153910726](https://i.loli.net/2020/08/27/c8DpW2liLUegk1Y.png)

![image-20200827154040487](https://i.loli.net/2020/08/27/c8DpW2liLUegk1Y.png)

![image-20200827154111653](https://i.loli.net/2020/08/27/54NFjn8eZ67mxf3.png)

![image-20200827154132243](https://i.loli.net/2020/08/27/FCbLJIPKHruMej9.png)

从上面的流程可以看出，Schedulers的静态变量NEW_THREAD 实际上就是
new NewThreadTask（）.call();

![image-20200827154246794](https://i.loli.net/2020/08/27/7rc9l6xqtAvF53w.png)

![image-20200827154310276](https://i.loli.net/2020/08/27/7rc9l6xqtAvF53w.png)

所以Schdulers.newThread（） 实际上就是创建了一个NewThreadScheduler;

![image-20200827154702670](https://i.loli.net/2020/08/27/a4RjQIGhkpD65Tu.png)

这个NewThreadSchduler起什么作用呢？
暂时还不清楚。

先回到RxJava的线程调度
subscribeOn(Schedulers.newThread())

![image-20200827151834602](https://i.loli.net/2020/08/27/8PKSxkyQUNJ3hlE.png)



RxJavaPlugins.onAssembly 不用管，subscribeOn得到的就是
ObservableSubscribeOn()

![image-20200827160441294](https://i.loli.net/2020/08/27/sgEPAuWLbZH35fx.png)

![image-20200827160454038](https://i.loli.net/2020/08/27/zTZ2eU9QIO3bv8j.png)

scheduler 传入到了ObservableSubscribeOn当中 能在哪里其作用呢？
上面的基础分析当中，已经知道。Observable.subcribe(observer)里会先调用obserbable具体实现类的subscribeActual  所以说ObservableSubscribeOn 的subscribeOnActual 会被执行。



回顾下ObservableSubscribeOn 的创建

![image-20200827161654284](https://i.loli.net/2020/08/27/R1SpxA47Cw5DIfz.png)

![image-20200827162335092](https://i.loli.net/2020/08/27/uFUOrZSDPqbwh4f.png)

![image-20200827162342439](https://i.loli.net/2020/08/27/i2ubQs3JevYLWlt.png)

看到上面的源码 ，发现 RxJava的链式调用其实是把每一个对象都层层包装成Observable ,每层都持有者上一层的引用。这样来达到分层处理的效果。也就是使用了装饰器模式。

回到源码分析
ObservableSubscribeOn 的subscribeOnActual 

![image-20200827171702539](https://i.loli.net/2020/08/27/oNJnQS8vwDhdbLy.png)



回顾一下ObservableCreate的subscribeActual(). 是先执行下游的onSubscribe,然后再执行上游Observable.subscribe();
感觉ObservableSubscribeOn 也应该类似。

![image-20200827173404233](https://i.loli.net/2020/08/27/OL4UNdYbyf5zqIl.png)

这部分是比较熟悉的， 也就是执行下游的onSubscribe方法。
但是看不到显示的调用上流的subscribe方法。

猜测可能和parent.setDisposable 有关
但是看名字 像是设置 上游Observable的dispose
所以猜测执行上游的subscribe的地方在

```java
scheduler.schedulerDirect(new SubscribeTask(parent));
```

从内往外看， 先看SubscribeTask
![image-20200827174411601](https://i.loli.net/2020/08/27/X98DJPvQNzO5dLx.png)

看到了SubscribeTask 是一个runnable 。看到了 runnable,那就是终于看到了和线程操作相关的部分了。
SubscribeTask的run方法就是执行了上游的subscribe()

在回顾一下，
![image-20200827174736847](https://i.loli.net/2020/08/27/LuBbF8pkEjN4sgf.png)

![](https://i.loli.net/2020/08/27/E64XHMG53oZfdlb.png)


![image-20200827175159220](https://i.loli.net/2020/08/27/vqFTzregx8AaEnl.png)



![image-20200827174803048](https://i.loli.net/2020/08/27/RY6NOoCsupzEWJG.png)

从上面的源码得知， 这是通过subscribeOn()传入的scheduler分发器来分发SubscribeTask(),SubscribeTask 实际就是一个runnable, 他的run函数中就会执行上游的subscribe();

现在知道了subscribeOn是把上游的subscribe() 封装到runnable当中，然后交给schedule去分发从而实现了切换线程的目的了。
具体内部是怎么实现的呢？

看源码：
![image-20200827180143739](https://i.loli.net/2020/08/27/i7NDAIrkUKdTY1n.png)

![image-20200827180158064](https://i.loli.net/2020/08/27/OdtJD4ScF3gxPbz.png)

![image-20200827180215013](https://i.loli.net/2020/08/27/aEr8UQntXcHzsmg.png)

![image-20200827180252986](https://i.loli.net/2020/08/27/KoFHxQX8nTjM3OI.png)

![image-20200827180307054](https://i.loli.net/2020/08/27/ygv64QGNCISqsDk.png)

![image-20200827181136701](https://i.loli.net/2020/08/27/OZIxoNh5Udema24.png)



从上面这段源码 可以看出， subsribeOn 中传入的 schedule就是重载了createWork函数 ，然后创建出不同的线程池，然后分发到线程当中去。
中间还加了些封装，实现了dispose ,这个接口。提供给外层来控制整个过程。

这个就是subscribeOn 切换 上游 Observable的subscribe的整个过程：

接下来 下游线程切换

### 下游线程切换 observeOn

![image-20200827181600441](https://i.loli.net/2020/08/27/io7nwvXWK4UjY83.png)

熟悉的模式
schedule的创建， 讲上游的时候已经讲过这里不再啰嗦

直奔ObservableObserveOn的subscribeActual函数
![image-20200827181719051](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200827181719051.png)

![image-20200827182305067](https://i.loli.net/2020/08/27/72pqKeI5sJHkX9U.png)



 红框内有点 不一样了，之前都是包装了 上游Observable,但是这里是包装了下游Observer。
那现在就可以猜测，RxJava是重写了下游的一系列函数通过 传入的scheduler 来达到切换线程的

先看onSubscribe()
![image-20200831152432594](https://i.loli.net/2020/08/31/rFgRoYE6s4BDuIM.png)

从这里看出，downStream的onSubscribe是执行在当前线程的 因为它不需要通过schedule .

挑的来看
![image-20200827183046069](https://i.loli.net/2020/08/27/IvsKAtSEZLpkrcd.png)

![image-20200827183115787](https://i.loli.net/2020/08/27/C76BaONVFxMDvTz.png)

![image-20200831150015879](https://i.loli.net/2020/08/31/yBFIKo578gRXqaA.png)

可以看出下游的全部操作都和schedule（）有关系。

![image-20200827183126579](https://i.loli.net/2020/08/27/y1bITkHXwGxCt4p.png)

​	run中的drainFused 看名字就觉得是和背压相关。先不管。



来看 drainNormal的源码

![image-20200831151124068](https://i.loli.net/2020/08/31/soNHcWhzQvAEZXB.png)

![image-20200831151215566](https://i.loli.net/2020/08/31/FIeoasMyL6PTf47.png)



checkTerminated 是检测任务是否结束，然后从queue中得到 待处理的元素然后执行downStream.onNext()

![image-20200831151327575](https://i.loli.net/2020/08/31/dOG1tLCYMkmHsgD.png)

downStream.onComplete()和 downStream.onError() 是在线程任务完成的时候执行。

下游的线程切换的实质，其实也是通过scheduler来执行runable 来处理。



## 总结

来总结一下线程切换：

1. 其实RXjava的整个链式调用就是一种责任链的模式，也就中下层的执行中包含链上层的执行；
2. 下游downStream.onSubscribe()是执行在当前线程的
3. subScribeOn指定的是上游的线程，其实就是用schedule去执行一个runnable
4. observeOn指定的是下游的 onNext ,onComplete ，onError的线程
5. 默认情况下，下游接收时间的线程和上游发送的线程是同一个线程



这第五点是怎么得到的呢？
首先下游除了onSubscribe 之外，其他的方法都是在上游的subscribe里面触发的。
如果上游的subscribe在线程A里被执行，在没有其他线程切换的干预下，那么下游的剩余的操作自然也就是在线程A里被执行。

Ps:
多次指定上游线程，只有第一次指定是生效这个结论是错误的。

来分析一下，下面的流程

```java
Observable.create()
          .subscribeOn( threadA)// ObservableA
          .subscribeOn( threadB)// ObservableB
          
```

一开始是得到Observable  其 subscribe () 是执行在当前线程。
经过subscribeOn( threadA )之后，Observable 就执行在线程A了，并且返回了ObservableA,
接着执行了subscribeOn(threadB),这个时候 ObservableA的subscribe就是执行在thradB当中了。
整体流程就是

ObservableB 在threadB 执行了ObservableA的subscribe()
ObservableA在threadA中执行了Observable的subscribe()
所以用户写在Observable的代码就被执行在了threadA当中了。
但threadB是起了作用的

所以subScribeOn是全部都会起作用的， 但是对于用户的代码是链下最近的那个subscribeOn才起作用





# Glide

参考 ：https://blog.csdn.net/github_33304260/article/details/78116312?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param

优势  可以与activity的生命周期绑定，加载Gif ，还可以配置相应的网络请求框架。



基础用法：

![image-20200831170512909](https://i.loli.net/2020/08/31/flVFp3SosNJQ2jW.png)

## with

![image-20200831170525832](https://i.loli.net/2020/08/31/jGP4LKkA9Ighb7O.png)

![image-20200831170840000](https://i.loli.net/2020/08/31/sUYKR3zjyIuFODe.png)

可以看出 with 函数有很多重载。

但是 目的只有一个  那就是 拿到 RequestManager
怎么拿到RequestManager呢？
得通过RequestManagerRetriever

### RequestManagerRetriever

​	with 怎么拿到RequestManager呢？

​    看with的代码  都是先借助 getRetriever()来处理的

​    

![image-20200918165147409](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200918165147409.png)

从 RequestManagerRetriever 的名字来看， 就是RequestManager的 发现器 Retriever的意思是 寻物犬
![image-20200918165840666](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200918165840666.png)

看ReqeustManagerRetriever 的注释， 意思很明显就是维护RequestManager的管理类。 去创建或者去获取。

看看RequestManagerRetriever的get函数

![image-20200918170059046](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200918170059046.png)

get的重载  基本对应着 Glide.with()的重载。

![image-20200918170229195](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200918170229195.png)

但是RequestManagerRetriever 的重载最终都会调用到get(Context) 这个函数 或者向当前的Activity添加一个fragment

![image-20200918171057125](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200918171057125.png)



![image-20200918171604350](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200918171604350.png)



application的ReuqestManger  实际上是RequestManagerRetriever的一个单例的静态变量。

为啥要向Activity 添加一个 fragment。
Glide 是通过这种手段来监听activity的生命周期的。
这样当图片还未被加载，activity就被销毁的时候，Glide就能直接监听到，并且不再处理该图片。

为什么不用android 提供的registerActivityLifecycleCallbacks接口来监听activity的生命周期呢？

因为registerActivityLifecycleCallbacks接口监听的是全部activity的，不是指定activity的。你得去重新去维护 Glide和关联的activity.很麻烦。并且由于监听的是全部activity，有的activity不一定会用到glide ,这会造成性能的浪费。





总结下：
Glide.with()是为了拿到RequestManager ,
RequestManager是由RequestManagerRetriever来维护的。
为啥要特地弄个RequestManger的维护类呢？

因为RequestManger并不是全局唯一的。
这种管理类做成唯一的不行吗？
可以是可以，但是Glide为了监听activity的生命周期，
在Glide.with()执行在子线程的时候，是会去给activity添加一个fragment的。有了这个fragment就能监听activity的生命周期了。然后就能及时的释放资源。

要点：

1. 通过 Glide.with()  来拿到 RequestManager
2. RequestManager通过RequetManagerRetriever来管理
3. RequestManagerRetriever管理的RequestManager有两种：
   a. 全局的和application 绑定的实例 
   b.  通过创建fragment 依附在当前activity上的RequestManager
4. RequestManagerRetriever  通过依附在activity上的fragment,从而可以监听activity的生命周期



## load

通过Glide.with() ,就可以获取到RequestManager
接下来就是执行RequestManager.load(str);



看下源码

![image-20200918175621382](https://i.loli.net/2020/09/18/rFORmbSVIwLq21C.png)



asDrwable() 是为了构建 RequestBuilder对象  
重点是RequestBuilder对象的laod(string) 函数

接下来看下ReuqestBuildre的load (String)的源码

![image-20200918180806268](https://i.loli.net/2020/09/18/rFORmbSVIwLq21C.png)

![image-20200918180836479](https://i.loli.net/2020/09/18/2afh3mLS9QXgVDM.png)

可以看出 load的核心就是 RequestBuilder 的构造函数了。因为loadGeneric（） 中只是做了些赋值操作。

RequestBuilder的构造函数如下

![image-20200918181144074](https://i.loli.net/2020/09/18/JO1CSm2eRgQYb5f.png)

发现RequestBuilder的构造函数 也只是在 做些赋值操作。
所以可以判定 RequestManager.load() 主要作用就是做了些赋值操作。真正做处理的地方是  into 这个函数。

## into



源码

```java
  /**
   * Sets the {@link ImageView} the resource will be loaded into, cancels any existing loads into
   * the view, and frees any resources Glide may have previously loaded into the view so they may be
   * reused.
   *
   * @see RequestManager#clear(Target)
   * @param view The view to cancel previous loads for and load the new resource into.
   * @return The {@link com.bumptech.glide.request.target.Target} used to wrap the given {@link
   *     ImageView}.
   */
  @NonNull
  public ViewTarget<ImageView, TranscodeType> into(@NonNull ImageView view) {
    Util.assertMainThread();
    Preconditions.checkNotNull(view);

    BaseRequestOptions<?> requestOptions = this;
    if (!requestOptions.isTransformationSet()
        && requestOptions.isTransformationAllowed()
        && view.getScaleType() != null) {
      // Clone in this method so that if we use this RequestBuilder to load into a View and then
      // into a different target, we don't retain the transformation applied based on the previous
      // View's scale type.
      switch (view.getScaleType()) {
        case CENTER_CROP:
          requestOptions = requestOptions.clone().optionalCenterCrop();
          break;
        case CENTER_INSIDE:
          requestOptions = requestOptions.clone().optionalCenterInside();
          break;
        case FIT_CENTER:
        case FIT_START:
        case FIT_END:
          requestOptions = requestOptions.clone().optionalFitCenter();
          break;
        case FIT_XY:
          requestOptions = requestOptions.clone().optionalCenterInside();
          break;
        case CENTER:
        case MATRIX:
        default:
          // Do nothing.
      }
    }

    return into(
        glideContext.buildImageViewTarget(view, transcodeClass),
        /*targetListener=*/ null,
        requestOptions,
        Executors.mainThreadExecutor());
  }

```



关键部分

![image-20200924103217506](https://i.loli.net/2020/09/24/b4toCOAmDMizG5P.png)

可以发现 只是 调用了带其他参数的into函数



看看多参数 的into 的各个参数是怎么把图片资源加载到imageView当中去的.

![image-20200924103949485](https://i.loli.net/2020/09/24/TsEG78ncb6tMBF3.png)

参数分别是Target , targetListener , requestOptions,Executor 
从名称上来看，   RequestOptions 应该是保存了相关的一些配置， executor 应该是指定了一些逻辑需要执行在哪个线程。Glide的改变UI这一步，肯定是在主线程的当中执行的。

Target和TargetListener 是啥的， 从名称上来看，知道了Target就应该知道Target了，而且这里TargetListener传的是null，所以先看Target即可。



### Target

![image-20200924104526713](https://i.loli.net/2020/09/24/t4R7ZBDOAHnaeLY.png)

![image-20200924104154309](https://i.loli.net/2020/09/24/8PfKtFLcIg6oAOU.png)

![image-20200924104208109](https://i.loli.net/2020/09/24/3mERZ8zkW9UDGNw.png)

看出Target是一个回调接口，  
和ImageView关联在一块的回调接口，那自然联想到是onResourceReady之后就把图片资源赋给 ImageView的啦。
那就接着往下看看.

Target是怎么构建的

### buildImageViewTarget-构建Target

![image-20200924104819224](https://i.loli.net/2020/09/24/Kv8X5uW4yscxp6l.png)

![image-20200924104846044](https://i.loli.net/2020/09/24/pHfnYbkdSALRBvF.png)

![image-20200924104938624](https://i.loli.net/2020/09/24/N4PfIj5xpBiHCZK.png)

![image-20200924105107655](https://i.loli.net/2020/09/24/6VCESaBmPOxg2KH.png)

这里看到了 给imageView设置资源的地方了
但是没其他的  还是的看下DrawableImageViewTarget的父类ImageViewTarget。看了下 只是实现了各个接口。

所以Target的创建只是实现了各个回调接口。

因此现在可以回到RequestBuilder的into（）中。



回到into

![image-20200924110617670](https://i.loli.net/2020/09/24/XuUBNFVI2YL4srt.png)

终于看到RequestManager了

看到这里 ，就感觉 RequestManager.trace函数是重点。不过先一步步的往下看。
猜测是  buildRequest 来构建 Request ，然后通过RequestManager的track 去执行，再接着track中会触发Target中的相关回调， 从而完成对ImageView的赋值



看着猜测来看，
先看buildRequest

### buildRequest-构建Request



![image-20200924111546310](https://i.loli.net/2020/09/24/abLBCrd5neIhiJZ.png)



buildRequestRecursive的实际就是
看有没有错误处理，没有的话就直接返回mainRequest,有的话，那么就把mainRequest和errorRequest打包在一起封装成一个新的。再返回

![image-20200924111718568](https://i.loli.net/2020/09/24/oX7WHswemBdPNEF.png)

![image-20200924111750264](https://i.loli.net/2020/09/24/Onh5PFUfZlYb9RK.png)





先抓住主干， 看MainRequest的构建，errorRequest先不管。

#### buildThumbnailRequestRecursive- 构建MainReqeust

buildThumbnailRequestRecursive中分了很多中情况 ，还包含了很多 缩略图的处理等。但是核心只有一句。

![image-20200924120020534](https://i.loli.net/2020/09/24/8DVPYd5xZpOMR6u.png)

![image-20200924120041790](https://i.loli.net/2020/09/24/wYCgEAXRUinfK4t.png)

要用的参数巨多 ，但是 实际上也只是new 了一个 SingleRequest

![image-20200924120452433](https://i.loli.net/2020/09/24/uyoA4idvPnwsbxC.png)





所以buildeRequest的实际就是new了一个singleRequest对象



接着回到 into中

![image-20200924120736051](https://i.loli.net/2020/09/24/QNO4fAnLZi7jPh8.png)

先不管缓存相关的，往下走。

```java
requestManager.clear(target); // 顾名思义
target.setRequest（request）; // 缓存相关

requestManager.trace( target,request);

return  target;
```

那就只剩下requestManger.trace(target, request)了

接着看RequestManager.trace

### RequestManager.trace

![image-20200924121302108](https://i.loli.net/2020/09/24/zarshnZgDVYpmEC.png)



先看TargetTracker.track

#### Targetracker（Target的追踪器）

![image-20200924121542038](https://i.loli.net/2020/09/24/DrbQtT7gzipuSn2.png)

targetTracker就是触发 target 和activity相关的生命周期回调的地方。

![image-20200924121831406](https://i.loli.net/2020/09/24/ePkHciWUdoLO8wm.png)

TargetTracker并没有关联到资源加载的回调， 所以资源加载应该是在 RequestManager.track中的 requestTracker.runRequest

#### RequestTracker.runRequest

![image-20200924122353282](https://i.loli.net/2020/09/24/CpQtB7f19zIAXHU.png)



执行的是Request.begin() , 对于我们的分析中， 也就是SingleRequest.begin()

#### Request.begin()

![image-20200924122955990](https://i.loli.net/2020/09/24/mX6xAKhnskD1cYM.png)

可以看出  begin就是  glide 读取资源的核心 .标红的三个函数 应该就是对应着glide的target对应的三个状态  start， ready ,failure



先来看fail 

##### Target- Failed状态 onLoadFailed

![image-20200924142501369](https://i.loli.net/2020/09/24/vbfuJHwrsmRWQd8.png)

都是些 状态设置和 触发回调
而target的onfail 方法在 setErrorPlaceholder（）中执行

![image-20200924142723315](https://i.loli.net/2020/09/24/xqSlrABmv5Cngh9.png)

以ImageViewTarget 为例子
onLoadFailed

![image-20200924142820036](https://i.loli.net/2020/09/24/F8Q7Ei9e15vDVmt.png)

![image-20200924142833153](https://i.loli.net/2020/09/24/81mKUN9Wcf4YLl5.png)

图片就被设置进去了。

接着看start 状态

##### Target- Start状态

没啥好说的 就只直接调用了target的onLoadStarted()

![image-20200924143301962](https://i.loli.net/2020/09/24/Ntz2Xj6bV8EaiwW.png)

![image-20200924143214898](https://i.loli.net/2020/09/24/G8AZTktDgXqCV7o.png)

##### Target- Ready状态

![image-20200924143524916](https://i.loli.net/2020/09/24/Q4PlANa8GIe7uqJ.png)

这可以可以观察下 glide的数据来源有哪些 -- DataSource 这个枚举类型

![image-20200924143700789](https://i.loli.net/2020/09/24/tv4VYnRcwKhq6mN.png)

总体三类 local， remote  ,cache

ok 接着ready状态往下看。



![image-20200924144105564](https://i.loli.net/2020/09/24/qo5jFiJlkTWf6QH.png)

![image-20200924144508858](https://i.loli.net/2020/09/24/WbSsE8L1tO76eRc.png)

 这里把result 就传过去了。
那么这个result是怎么得来的呢？

回到 两个参数的 onResourceReady

![image-20200924144820838](https://i.loli.net/2020/09/24/AgHvKh4OtXEMS2p.png)

可以看出是通过resource.get来得到的。

![image-20200924144919292](https://i.loli.net/2020/09/24/wPX6lxKeL1hWTBz.png)

get的实现有这么多种， 

挑一个来看，
![image-20200924145310416](https://i.loli.net/2020/09/24/rDcVmtMbuj19PiU.png)

 发现是Resource的构造函数中带进去的



那就要回到 singleRequest.begin中看 resource是怎么来的了.

![image-20200924145455842](https://i.loli.net/2020/09/24/GTZkwUOtK54pBaA.png)

resource是个成员变量

![image-20200924145548961](https://i.loli.net/2020/09/24/aHEXvFUZVRWT8jd.png)

只有一个赋有效值的地方

那就 3个参数的 onResourceReady  


![image-20200924145809835](https://i.loli.net/2020/09/24/3waeSYRtnNUMkzO.png)

卧槽 ，这不是套娃了吗？
一直都是空了呀。

这肯定是有问题的。

所以这个三个参数的onResourceReady 一定是有别的入口.
而是正常来说这个入口应该是由 start状态触发的

追踪一下

三个参数的onResourceReady 只被 两个参数的onResourceReady调用

![image-20200924150344809](https://i.loli.net/2020/09/24/NvClcwADtg8BaR9.png)



![image-20200924150409148](https://i.loli.net/2020/09/24/OJdXjTSvIycEBxt.png)

![image-20200924150433107](https://i.loli.net/2020/09/24/Dq7sC1AuBQMXboV.png)

这就追踪到了， 还是由begin 触发的

![image-20200924150517494](https://i.loli.net/2020/09/24/Oh2qcbC8GgjlyVU.png)

所以说 resource 资源的加载 是在Request的begin中 的onSizeReady中触发的

![image-20200924150732286](https://i.loli.net/2020/09/24/Fb4A7mGkdvaNrMI.png)

onsizeReady的代码很长， 但是核心的就是这句。
调用engine.load();

所以这个engine 就是 读取图片资源的关键了

##### Engine  -  负责开始读取和管理活动、以及内存资源 

![image-20200924150953322](https://i.loli.net/2020/09/24/O13SfRGy6EZmpgb.png)

看注释 ： 负责开始读取和管理活动、以及内存资源 。

大致知道了engine的作用。
再细看下代码



发现个问题 可以观察一下

![image-20200924151406776](https://i.loli.net/2020/09/24/RamleygfE8XtUnM.png)

![image-20200924152207312](https://i.loli.net/2020/09/24/NiOHwL8bc6WefBR.png)

核心就是 waitForExistingOrStartNewJob

![image-20200924153415812](https://i.loli.net/2020/09/24/GvhlZy5wjD39z4N.png)

先看下返回值。



![image-20200924153223710](https://i.loli.net/2020/09/24/GyZzpTwPu14QoAv.png)

看的出来 这个返回值，只是用来cancel操作的



所以核心就是EngineJob的 start

##### EngineJob.start

![image-20200924153525218](https://i.loli.net/2020/09/24/3juKYsi1DqMHeBN.png)

![image-20200924153817169](https://i.loli.net/2020/09/24/ch86NTPjqAF1IoV.png)





所以现在要看的是这个异步任务DecodeJob 做了啥。

##### DevideJob

DevideJob .run

![image-20200924154010525](https://i.loli.net/2020/09/24/FRYWUSH4wqdivrC.png)

核心是 runWrapped()函数

![image-20200924160906021](https://i.loli.net/2020/09/24/RquLPiUo1BAQ4Yy.png)

先看INITIALIZE状态的 起执行了 runGenerators()

![image-20200924161001030](https://i.loli.net/2020/09/24/PbRFtk7K5pB3WGA.png)

runGenerators中重点是红框的两句

![image-20200925141647092](https://i.loli.net/2020/09/25/RruDzcOfx7mWT2C.png)

看看个currentGenerator 是怎么初始化的。

![image-20200925142001194](https://i.loli.net/2020/09/25/FndJ6qewujbhpYR.png)

![image-20200925142021802](https://i.loli.net/2020/09/25/SvBy3znbPG57dZF.png)

其实就是对应了 三种类型 的数据源 



回到startNext函数

![image-20200925142129915](https://i.loli.net/2020/09/25/B61YeynzrHqNjV8.png)

有三种重载。

对于首次的情况，肯定是直接去拿源。SourceGenerator

接着来看SourceGenerator的startNext函数

![image-20200925143008693](https://i.loli.net/2020/09/25/nRvGZ5f1sVqbr7z.png)



![image-20200925143209174](https://i.loli.net/2020/09/25/5tvMhAklRxUwZQJ.png)

所以 判定 资源的加载就是在 这个  loadData.fetcher.loadData 这里处理了。

![image-20200925143402268](https://i.loli.net/2020/09/25/uHBWR94KGpiq13X.png)

![image-20200925143421768](https://i.loli.net/2020/09/25/Zw2pMBV7jyGnaSR.png)

来看看这两个类

![image-20200925143733976](https://i.loli.net/2020/09/25/H3a5MsCx1JROfjG.png)

![image-20200925143932515](https://i.loli.net/2020/09/25/ZWtlXjuLrFHziaN.png)

![image-20200925144040846](https://i.loli.net/2020/09/25/4P5GUvbXyEikDjF.png)

这样大体类的功能就知道了。

真正加载资源的就是DataFetcher这个接口了

这个DataFetcher的实现这么多， 该用哪一个呢？

![image-20200925144244542](https://i.loli.net/2020/09/25/LrESQ9ImtRiq26F.png)

从名称上来看 ， 以例子来说 应该是HtppUrlFetcher 。 但是是怎么对应到HttpUrlFetcher的 大概看了下。感觉很复杂。就先不管，先直接去看主流程。

也就是HttpUrlFetcher

##### DataFetcher  - 真正加载资源的地方



接下来看HttpUrlFetcher 的loadData的源码

![image-20200925145120341](https://i.loli.net/2020/09/25/mpbwTaJtMedFHS8.png)





![image-20200925150602781](https://i.loli.net/2020/09/25/jyUq96JDOLBac1t.png)

![image-20200925150724757](https://i.loli.net/2020/09/25/qlpUAhjfM564xi3.png)

终于看到HTTP的东西了
那现在就拿到了  流。 接着看流是怎么处理的

![image-20200925151352271](https://i.loli.net/2020/09/25/wTeQpCOYcGzLjWs.png)

![image-20200925151411926](https://i.loli.net/2020/09/25/XwkYfL2RJMHo4Nb.png)



流给到了SourceGenerator .onDataReadyInternal来处理

![image-20200925152403480](https://i.loli.net/2020/09/25/2sV9nPulIQMTxgJ.png)

![image-20200925152803892](https://i.loli.net/2020/09/25/jrSm8YvVXB1PgyI.png)

SourceGenerator 中的FetcherReadyCallback  传的是DecodeJob对象

所以调用的是DecodeJob.onDataFetcherReady

![image-20200925153133600](https://i.loli.net/2020/09/25/yZI1hT4VP3Lai2b.png)

![image-20200925153254444](https://i.loli.net/2020/09/25/LIcSxvBo6gdH7qD.png)



先看decode资源

![image-20200925153730315](https://i.loli.net/2020/09/25/n2eCkuyZXAiEYSx.png)

![image-20200925153840347](https://i.loli.net/2020/09/25/Wwv3CjsAmqPVorQ.png)

![image-20200925153938909](https://i.loli.net/2020/09/25/32sbuJdfwvtMeQE.png)

![image-20200925154028613](https://i.loli.net/2020/09/25/6Y9jmWHpD1Gaxed.png)

![image-20200925154234144](https://i.loli.net/2020/09/25/jUhnI6WF3c41SDr.png)

返回了一个LoadPath 实例 

接着往下看



![image-20200925154722445](https://i.loli.net/2020/09/25/rVtvYuilgP3OATX.png)

![image-20200925154832509](https://i.loli.net/2020/09/25/5gnfqxmprhaKsWb.png)

然后经过Registry的处理 和 一系列的调用 （涉及到好几个类，  晕乎乎的 先不管）



![image-20200925155556879](https://i.loli.net/2020/09/25/lFcZGXfKrS9IPWi.png)



![image-20200925155644101](https://i.loli.net/2020/09/25/lFcZGXfKrS9IPWi.png)



这几句先停一下，不往下跟了。
这里可以判断出是 返回的值Resource<Transcode>了  ，先返回去。看看Resource中的这个泛型是怎么转成能设置给imageView的类型的

![image-20200925160436052](https://i.loli.net/2020/09/25/wbr6j8x1s9CNATJ.png)



经过一系列 让我头大的 调用  会回到

![image-20200925161234606](https://i.loli.net/2020/09/25/T7dAJmjefQ5cRO8.png)



![image-20200925161537244](https://i.loli.net/2020/09/25/RpFdHTnQ2jELNvM.png)

![image-20200925161600687](https://i.loli.net/2020/09/25/pPguAN9GY4z3Q8r.png)

![image-20200925161637873](https://i.loli.net/2020/09/25/TWcfM5Vkwb4K1Rm.png)

![image-20200925161857799](https://i.loli.net/2020/09/25/bv2yktcGsEFap8q.png)

而后会执行到这里。







接着回到

![image-20200927095922449](https://i.loli.net/2020/09/27/y6JhSmdwpsfUbCI.png)

这里就是

![image-20200927100505528](https://i.loli.net/2020/09/27/zV6G1ra5WkquEKd.png)



![image-20200927141513622](https://i.loli.net/2020/09/27/feYCD6IGXABP7uL.png)

![image-20200927141551282](https://i.loli.net/2020/09/27/USIW86Jvjapo2LK.png)





#### 总结Into流程

重头再整理一遍主流程

<img src="https://i.loli.net/2020/09/30/hcOJB6bEMSLfmQo.png"  />





主流程

RequestBuilder 构建ViewTarget   然后 构建Request
接着通过RequestManager来执行request
然后就开始异步执行加载任务， 真正执行加载任务是通过EngineJob来执行的
engineJob 是启动DecodeJob这个runnable来启动任务的。
DecodeJob 这个任务有分三个部分 

1. 读取未解析的数据， 
2.  解码未解析的数据 
3.  把解码后的数据转换成需要的格式

读取数据通过 DataFetcher来完成
解码数据通过 ResourceDecode 来完成
转换格式通过ResourceTranscoder 来完成

转换完之后 再通过 一层一层的回调 最终回调到ViewTarget的ready回调用，viewTarget内部维护了用户传入的view,由其完成贴图操作
转换完成 - > DecodeJob.callback --> EngineJob.callback --> 
Request.callback --> viewTarget.callback





大致的主流程是这样。
还有很多其他的要求 没有说到  他的三级缓存，以及各个泛指的指定的意义等都没有说。这看到头疼先看到这里。后面再继续补充



GlideContext 是全局唯一的