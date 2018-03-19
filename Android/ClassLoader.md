# Android中ClassLoader


## ClassLoader介绍

> 对于Java程序来说，编写程序就是编写类，运行程序也就是运行类（编译得到的class文件），其中起到关键作用的就是类加载器ClassLoader。

对于Android来说，ClassLoader作用是负责从dex文件中加载所需的类到Dalvik/ART虚拟机

## ClassLoader种类，作用，区别


ClassLoader有三个直接子类BootClassLoader，BaseDexClassLoader，SecureClassLoader（略）；
BaseDexClassLoader的直接子类有直接子类是 PathClassLoader 和 DexClassLoader (Android O添加了 InMemoryDexClassLoader)

## BootClassLoader

BootClassLoader是ClassLoader的内部类，在系统启动时创建，是系统最顶层的ClassLoader，当一个类没有加载器时，默认获取此加载器。

## PathClassLoader

PathClassLoader 在应用启动时创建，一般从 data/app/… 安装目录下加载 apk 文件

## DexClassLoader

DexClassLoader支持从任何目录加载包含.dex的apk，jar文件，多个文件用分隔符分隔；


## ClassLoader双亲委托模型

当类加载器在收到加载类或资源的请求时，通常会先委托给父加载器加载，只有当父加载器找不到指定的类或者资源时，自身才会执行实际的类加载过程。


ClassLoader执行loadClass的流程：

1. 首先判断当前ClassLoader是否加载过此类，如果加载过就直接返回；
2. 如果没有加载过，就委托给Parent加载器；
3. 如果继承链上所有加载器都加载此类失败，那么当前ClassLoader加载此类。

作用：

1. 一些Framework层的类被顶层的ClassLoader加载后，其他地方变不需要再加载。
2. 不同继承路线上的ClassLoader加载的类便不再相同，这样的限制避免了用户自己的代码冒充核心类库的类访问核心类库包可见成员的情况。

## 双亲委托模型在热修复中的应用

热修复就是想办法让系统加载我们修复后的dex，从而达到修复目的。

原理：

1. PathClassLoader负责加载已安装的apk中的class.dex文件
2. DexClassLoader负责加载SD卡中的class.dex文件
3. DexClassLoader 和 PathClassLoader 的基类 BaseDexClassLoader 查找 class 是通过其内部的 DexPathList pathList 来查找的
4. DexPathList 内部有一个 Element[] dexElements 数组，其 findClass() 方法的实现就是遍历该数组，查找 class ，一旦找到需要的类，就直接返回，停止遍历
5. DexClassLoader获得的Element[] dexElements 数组通过反射添加到PathClassLoader获得的Elements 数组的前面，于是先查找到最新的class后便会停止查找，从而达到修复目的。


## 参考

[热修复实现：ClassLoader 方式的实现](https://jaeger.itscoder.com/android/2016/09/20/nuva-source-code-analysis)

[热修复入门：Android 中的 ClassLoader](https://jaeger.itscoder.com/android/2016/08/27/android-classloader.html)