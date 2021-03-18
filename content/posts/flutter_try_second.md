---
title: "第二次尝试Flutter"
date: 2021-03-11T13:53:48+08:00
draft: false
---

## 前言
学了几天flutter，感觉写起来还是很香的，比原生开发速度快很多，不过以前没接触过安卓开发，思维还在Web开发上，记录一下

## 布局
一般是使用`Column`和`Row`以及`Container`来布局比较多，还有`Expanded`，类似与`flex: 1`这个效果，能把剩余的空间补充满。

在使用的时候还遇到了按钮宽度需要100%的情况，2.0废弃了1.0一些组件，按钮一般使用`ElevatedButton`，但是没法设置宽高。网上查了一下用`SizedBox`来当它的父容器，这样它就会跟着父容器的宽高走了，需要宽度100%的话就是`double.infinity`

## 如何设置width:100%
目前发现有`double.infinity`和`MediaQuery.of(context).size.width`。后续发现的话继续补充

## 路由
路由的话一般定义在`MaterialApp`里面，如下：
```dart
return MaterialApp(
    title: 'title',
    initialRoute: "/home", // 默认的路由
    routes: {
        "/home": (context) => MyHome(), // 每个路由需要默认传参context
        "/login": (context) => MyLogin()
    },
);
```
导航的话常用以下几个
```dart
Navigator.pushNamed(context, "/login"); // 跳转到/login页面
Navigator.pop(context); // 返回上一级，可传第二个参数携带回上一级
Navigator.pushReplacementNamed(context, "/login"); //重定向至/login页面，就没法返回到上一级页面了
```

## 父子组件传参
* 父组件调用子组件
```dart
// 定义一个全局key
GlobalKey<_MyMapState> mapKey = GlobalKey();

// 子组件
class MyMap extends StatefulWidget {
  MyMap({Key key}) : super(key: key);
  @override
  _MyMapState createState() => _MyMapState();
}
class _MyMapState extends State<MyMap> {
    // ...
    void getItem() {
        print('get');
    }
}

// 父组件
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Row(
        children: [
            TextButton(
                child: Text('Click'),
                onPressed: () {
                    // 调用子组件方法
                    mapKey.currentState.getItem();
                }
            ),
            // 在引入子组件时，传入全局key
            MyMap(key: mapKey)
        ]
    );
  }
}
```
* 子组件调用父组件
```dart
// 子组件
// 父组件通过传参传入方法，子组件调用
class MyMap extends StatefulWidget {
  final changeParent;
  MyMap({Key key, this.changeParent}) : super(key: key);
  @override
  _MyMapState createState() => _MyMapState();
}
class _MyMapState extends State<MyMap> {
    // ...
    void getItem() {
        // 调用父组件
        widget.changeParent();
    }
}
```

## 总结
如果是Web开发，可能思想上会有点不一样，多写写就能领悟到了。用Flutter进行快速开发的话还是很香的，毕竟只是为了出成品，速度足够快，也不用管性能优化。