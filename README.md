# TamicAppMonitoring
Android App 无痕迹全埋点方案

本次基于的埋点框架： https://github.com/Tamicer/SkyMonitoring


# 背景

目前统计已经是一个产品常见的需求，尤其在业务模式探索的前期，埋点功能更是必不可少的功能，下面将介绍最简单的app全埋点方案！



# 什么是数据埋点

数据埋点是一般项目采用统计UV，PV，Action，Time等一系列的数据信息，对特定用户行为或事件进行捕获、处理和发送的相关技术及其实施过程。



# 为什么要数据埋点

产品或运营分析人员，基于埋点数据分析需要，对用户行为的每一个事件进行埋点布置，并通过SDK上报埋点的数据结果，进行分析，并进一步优化产品或指导运营。

# 数据埋点包括哪些

这里有我之前写的一篇文章[App优质精准的用户行为统计和日志打捞方案](http://blog.csdn.net/sk719887916/article/details/50931485)

地址：http://blog.csdn.net/sk719887916/article/details/50931485


# 数据埋点采集模式

##自动埋点

App通过代理，调用Sdk相关API，进行的将数据埋点上报的模式.

## 无痕埋点

无需通过专门提供代理类，直接由sdk提供相关接口，或者通过编译工具，预编译替换代码等，直接由sdk全部负责采集上报

## 可视化埋点

可视化埋点指 前端或者app端基于dom 元素和控件所精准自动埋点的上报的方案。


## 对比分析：

### 自动埋点：

 缺点：
 1 开发人员工作量大，需对业务提供唯一的ID，来区分每一个业务，无论是否提供sdk代理，业务开发人员至少需要多次调用sdk相关API.

2  业务人员和产品沟通成本提高，需要对具体业务制定相关的业务标识，以便于产品分析和统计

优点：

 产品运营工作量少，对照业务映射表，就能分析出还原相关业务场景， 数据比较精细，无需大量的加工和处理。
 
### 无痕埋点
  
缺点：

1 sdk开发人员需提供一套无痕埋点技术成品，包括能正确获取PV，UV，ACtion,TIme等多项统计指标。前期技术投入大。

2 数据量大，需后端落地进行大量处理，并由产品进行自我还原业务员场景。 无论采用智能系统平台，还是通过原生的数据库查询数据，都是一种大量的分析精力。

优点：

1 开发人员工作量小，无需对业务标识进行唯一区分，由sdk自动进行生成，ID规则由sdk和产品进行约定。减少业务人员的沟通成本和使用步骤。

2 数据量全面，覆盖面广，产品可按需进行分析。做到毫无遗漏。

3 支持动态页面和局部动效的统计。

## 可视化埋点


**优点：**

1 相对数据量而言
相比较于无埋点相而言对较低，但是这个可视化元素的识别技术是客户端或者前端所要实现的，唯一id生成也无需客户端去自定义规则，这套生成规则由相关产品在自动化工具的情况下生成配置表，下发到客户端，再由客户端按坑就班到相关界面去实现。

2 数据量相对精确

**缺点：**

1 可视化工具的平台的搭建，静态页面的元素识别都需要额外开发。
2 动态效果可能会遗漏。


#实现方案：

埋点需求可参考我之前的文章：

[App优质精准的用户行为统计和日志打捞方案](http://blog.csdn.net/sk719887916/article/details/50931485)

[App打造自定义的统计SDK](https://www.jianshu.com/p/cd83e81b78aa)

自动埋点实际上也是，提供一个base类，由业务类继承base类，在base里面做相关统计api调用，
可参考我的**github:https://github.com/Tamicer/SkyMonitoring**



## 核心实现： ##

以android作为列子：


提供自动遍历元素 并能扑捉点击的控件的activity, 并能在生命周期统计pv的打开和关闭，调用我开源的`SkyMonitoring`的对应的api.


复写`dispatchTouchEvent(MotionEvent ev)` 事件函数，确定被点击的view的相关位置，并生成唯一的ID，企业级app都是从服务器下发对应的ID，对应页面去调用埋点sdk Api，实现事件行为`TcStatInterface.initEvent(path.viewTree)`;。

这个path就是view的路径，页面的深度路径，包括打开和关闭sdk在SkyMonitoring中已能自动获取。

本次demo是id生成规则是按照 :包名+ Activity+ Viewgroup+ Layout+ view + View index + viewID实现的。

业务直接去继承`TamicActivity`即可，就能去实现所有可视化view的埋点功能。


App项目集成使用，初始化url和相关统计配置字典，这个字典可以从服务器下发下来，我本次只是通过简单的本地文件做实践。

```
    public class StatAppliation extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        // you app id
        int appId = 21212;
        // assets
        String fileName = "my_statconfig.json";
        String url = "https://github.com/Tamicer/TamicAppMonitoring";
        // init statSdk
        TcStatInterface.initialize(this, appId, "you app chanel", fileName);
        TcStatInterface.setUrl(url);
        TcStatInterface.setUploadPolicy(TcStatInterface.UploadPolicy.UPLOAD_POLICY_DEVELOPMENT, TcStatInterface.UPLOAD_TIME_ONE);
     }
    }

```

可视化也可以通过aop插桩实现，但是实现起来对代码的入侵性太高，这里不做介绍。

Aop 插桩对碎片化fragment支持比较好。对这块的介绍可看我以前在公众号推送的一篇文章
：[AOP编程之AspectJ实战实现数据无痕埋点](https://mp.weixin.qq.com/s/neH9JXL5AYzjaAaxF-ZF-g)

可参考：
https://www.baidu.com/link?url=FniQOFyj1pd6O5Fz6viRMN3ZgexIKAk7SQ08EgpBU9cHHMszPlm2jRXJ21mkomtY&wd=&eqid=ffc87acf0005fd18000000045a5d98dd

#  项目地址：
github:https://github.com/Tamicer/TamicAppMonitoring

>Tamic 原创 http://blog.csdn.net/sk719887916/article/details/79074556

第一时间获取我的技术文章请关注微信公众号！

![开发者技术前线](http://upload-images.jianshu.io/upload_images/2022038-a7b567ef3a0b0d1f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

