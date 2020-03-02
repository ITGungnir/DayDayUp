## ARouter源码解析

最近看了阿里巴巴的路由框架`ARouter`的源码，这里记录一下心得。文章整体篇幅较长，建议先马再看。

`ARouter`的源码可以在[GitHub - alibaba/ARouter](https://github.com/alibaba/ARouter/tree/master)下载，下载完成后可能会出现构建出错的问题，记得把`settings.gradle`文件中的`include ':arouter-idea-plugin`这一行注掉就可以了。

下载完成后打开项目，可以看到项目中共包含7个模块：
* `arouter-annotation`模块：注解模块，包含框架中用到的注解，以及一些枚举类和路由封装实体类；
* `arouter-compiler`模块：注解处理模块，用来处理项目中的注解，并生成路由表文件；
* `arouter-gradle-plugin`模块：`Gradle`插件模块，用来扫描项目中的字节码文件，并可以动态的向文件中注入代码；
* `arouter-api`模块：`API`模块，包含了`ARouter`暴露给外界的调用入口；
* `app`、`module-java`、`module-kotlin`模块：`DEMO`模块，包括`Java`模块和`Kotlin`模块，用来测试框架的功能和性能。

### arouter-annotation 模块
`ARouter`为我们提供了三种注解：
* `@Route`注解：用于标识一个路由节点，其中可以指定路由的路径、分组、名称、优先级等；
* `@Autowired`注解：用于将路由参数自动注入到路由节点中，其中可以指定参数的名称、是否必须、描述等；
* `@Interceptor`注解：用于标识一个路由拦截器，其中可以指定优先级、名称等。

`ARouter`中支持的参数类型可以从`TypeKind`类中看到：
```java
public enum TypeKind {
    // Base type
    BOOLEAN, BYTE, SHORT, INT, LONG, CHAR, FLOAT, DOUBLE,
    // Other type
    STRING, SERIALIZABLE, PARCELABLE, OBJECT;
}
```

`ARouter`中支持的路由节点类型可以从`RouteType`类中看到：
```java
public enum RouteType {
    ACTIVITY(0, "android.app.Activity"),
    SERVICE(1, "android.app.Service"),
    PROVIDER(2, "com.alibaba.android.arouter.facade.template.IProvider"),
    CONTENT_PROVIDER(-1, "android.app.ContentProvider"),
    BOARDCAST(-1, ""),
    METHOD(-1, ""),
    FRAGMENT(-1, "android.app.Fragment"),
    UNKNOWN(-1, "Unknown route type");
}
```
可以看到，`ARouter`中可以将`Activity`、`Service`、`ContentProvider`、`Fragment`以及框架内提供的`IProvider`等组件作为路由节点。

`arouter-annotation`模块中还提供了一个路由信息封装类`RouteMeta`，其中封装的信息如下：
```java
public class RouteMeta {
    private RouteType type;         // Type of route
    private Element rawType;        // Raw type of route
    private Class<?> destination;   // Destination
    private String path;            // Path of route
    private String group;           // Group of route
    private int priority = -1;      // The smaller the number, the higher the priority
    private int extra;              // Extra data
    private Map<String, Integer> paramsType;  // Param type
    private String name;
}
```

### arouter-compiler 模块

### arouter-gradle-plugin 模块

### arouter-api 模块

### 一些细节
* 路由文档是怎样生成的？
* 调试模式有什么用？开启后可以做什么？
* `ARouter`是如何实现`DeepLink`的？
* `ARouter`是如何支持`InstantRun`的？
