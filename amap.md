## 分析
首先在我们使用高德sdk的时候我们需要3个东西
* 应用的包名
* 编译apk签名的sha1值
* 高德控制台生成的key

在我们使用sdk的时候它会获取我们的这3样东西，然后跟服务器做对比，如果在服务器找不到就会返回key鉴权失败，
所以我们只需要在获取这3样东西的时候,把我们预先在高德后台添加的那3样写死在jar包里面，这样它跟服务器对比永远都是对的，从而达到破解的目的

## 开始破解
在我们使用高德定位sdk的时候需要在注册一个services，像这样

``` java
<service android:name="com.amap.api.location.APSService"></service>
```
那么我打算从这个`APSService`入手

我们先来用Jadx导入jar包反编译看下目录结构：

![](/screenshot/QQ截图20170525161133.png)

看到这几个类没有混淆，轻松的找到了`APSService`

打开APSService,我们可以看到在`onCreate()`使用代理模式创建了实现`APSServiceBase` 的`Service`

![](/screenshot/QQ截图20170525161647.png)

`APSServiceBase`既然是个接口，那么我们来找下它的实现类是谁

![](/screenshot/QQ截图20170525162318.png)

我们右键这个接口名点击`Find Usage`,可以看到f.class实现了它

![](/screenshot/QQ截图20170525162508.png)

我们打开这个类看下，发现启动这个service的时候传入了3个值，这3个值是不是就是高德需要的那3个参数呢？

![](/screenshot/QQ截图20170525162604.png)


既然是`intent.getStringExtra("a");`，那么肯定会调用`putExtra()`，我们来全局搜下


![](/screenshot/QQ截图20170525162907.png)

这个3不正是f.class取出来的这3个参数吗？我们双击打开看下

![](/screenshot/QQ截图20170525163020.png)

发现有个眼熟东西`getAPIKEY()`，初步可以断定就是我们`AndroidManifest`里填写的key，我们再来看看`b`传过去的是什么

![](/screenshot/QQ截图20170525163417.png)

可以看到`getPackageName();`获取了包名。

也就是说现在`a`传过去的是key，`b`传过去的是包名，那么毫无疑问`d`应该就是签名的sha1值了

那么既然找到这个地方了，我们就直接在里面写死

## 修改jar

使用 Android Studio 新建一个 Android Library 的 Module ，包名与 jar 包要修改的类包名相同,新建一个类，与要修改的类名相同

然后复制里面的代码到我们新建的类中

>复制后可能会报很多错，所以我们得对着Jadx一点一点的修改这些错

修复这些错后，填上我们在高德后台创建的数据

![](/screenshot/QQ截图20170525173227.png)

ok，现在我们来打包生成jar包

点击Build-Make Project

![](/screenshot/QQ截图20170525173828.png)

build成功之后我们找到这个我们生成的`classes.jar`

![](/screenshot/QQ截图20170525174034.png)


然后我们用压缩软件打开`classes.jar`和`AMap_Location_V3.4.0_20170427.jar`

找到生成的class文件替换`AMap_Location_V3.4.0_20170427.jar`里面的的class

ok,这样就我们大功告成了，以后再也不用申请key了!

>每个版本的代码以及破解方式不一样，这里只是提供思路，仅供参考！