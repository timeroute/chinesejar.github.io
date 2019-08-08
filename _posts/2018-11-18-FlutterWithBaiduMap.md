---
layout: page
title: Flutter 显示百度地图 Native 组件
date: 2018-11-18 11:51:15
tags:
- flutter
- Android
---

> Flutter 相对还是很新的一门技术，目前来看市场上好像阿里对 Flutter 的关注度最高，相关的技术讨论也是最多的。今天看了篇 Flutter 与 Native 混合栈开发的文章，觉得 Flutter 好牛逼，当然这只是个 Flutter 新手的错觉～

文章写的很好，简单易懂，先放出处：[在Flutter中嵌入Native组件的正确姿势是...](https://zhuanlan.zhihu.com/p/50078969)，看了文章想亲自动手实践一下。

## 前提

1. 已经有了 Flutter 开发环境
2. 已经注册了百度地图的开发者密钥，并按 [教程](http://lbsyun.baidu.com/index.php?title=androidsdk/guide/create-project/androidstudio) 导入 SDK
3. 创建一个 Flutter Application 工程

## 开始

Flutter 显示 Native 组件最关键的一个类是 AndroidView，它可以嵌入到 Widget 视图里，不过这种操作代价比较大， Flutter 官方文档描述提到尽可能使用 Flutter 的实现方式来避免使用它。

### Native 端

#### 初始化百度地图

首先注意下 `AndroidManifest.xml` 文件 Application 的 name 参数为 `io.flutter.app.FlutterApplication`，所以如果想在 Application 初始化百度地图 SDK 的话，应当继承它来重写 `onCreate` 方法

新建一个名为 `MainApplication.java` 的文件

```java
public class MainApplication extends FlutterApplication {

    @Override
    public void onCreate() {
        super.onCreate();
        // 初始化百度地图 SDK
        SDKInitializer.initialize(this);
        SDKInitializer.setCoordType(CoordType.BD09LL);
    }
}
```

然后将 `AndroidManifest.xml` 的 application name 参数改为 `.MainApplication`

#### 添加 PlatformViewFactory

写一个 `PlatformViewFactory`，`PlatformViewFactory` 的主要任务是，在 create() 方法中创建一个 View 并把它传给 Flutter

```java
public class BMapViewFactory extends PlatformViewFactory {

    private MapView mapView;

    public BMapViewFactory(MessageCodec<Object> createArgsCodec, MapView mapView) {
        super(createArgsCodec);
        this.mapView = mapView;
    }

    @Override
    public PlatformView create(final Context context, int i, Object o) {
        return new PlatformView() {
            @Override
            public View getView() {
                return mapView;
            }

            @Override
            public void dispose() {

            }
        };
    }
}
```

#### 添加 ViewRegistrant

添加 `ViewRegistrant` 类将该插件注册到 APP，`ViewType` 为 `MapView`。参数中新增实例化的百度地图 MapView。

```java
public final class ViewRegistrant {
    public static void registerWith(PluginRegistry registry, MapView mapView) {
        final String key = ViewRegistrant.class.getCanonicalName();

        if (registry.hasPlugin(key)) return;

        PluginRegistry.Registrar registrar = registry.registrarFor(key);
        registrar.platformViewRegistry().registerViewFactory("MapView", new BMapViewFactory(new StandardMessageCodec(), mapView));
    }
}
```

#### 注册 PlatformViewFactory

在 `MainActivity` 中实例化地图并且注册以上添加的 `ViewRegistrant` 类。

```java
public class MainActivity extends FlutterActivity {
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    MapView mapView = new MapView(this);
    ViewRegistrant.registerWith(this, mapView);
  }
}
```

### Flutter 端

简单实现一个只有地图的界面，实例化 `AndroidView`，`viewType` 设置为刚刚创建的 `MapView`，将百度地图显示。

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      title: 'Flutter Demo',
      theme: new ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: new Scaffold(
        appBar: AppBar(title: Text("Map"),),
        body: new Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Expanded(child: new AndroidView(viewType: 'MapView'))
          ],
        ),
      ),
    );
  }
}
```

## 编译

如此，地图正常显示了。

![ui]({{ site.url }}/assets/FlutterWithBaiduMap/ui.png)
