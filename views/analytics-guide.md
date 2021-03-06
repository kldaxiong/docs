# 统计分析开发指南

使用 LeanCloud 统计服务可以帮助开发者在应用上线之后及时查阅各类统计指标，它包含但不限于如下内容：

- 应用趋势（实时数据、版本分布、渠道分布）
- 行为分析（应用使用、推送统计、路径分析、使用时长、使用频率、访问页面）
- 用户分析（用户留存、终端设备、地理位置）
- 推广营销分析（渠道对比）
- 错误分析（错误分析、错误统计）

## 工作原理

![analytics-workflow](images/analytics-workflow.svg)

## SDK 安装与初始化

- [Objective-C](sdk_setup-objc.html)
- [Android](sdk_setup-android.html)
- [Unity](sdk_setup-dotnet.html)
- [Windows](sdk_setup-dotnet.html)

## 开启统计功能

```cs
AVAnalytics.InitAsync(IAVAnalyticsDevice device);
// IAVAnalyticsDevice 是一个接口，需要开发者自己实现针对当前运行平台下的一些硬件和网络等相关的数据
// 下面是一个在 PC 实现这个接口的样例
public class PC : IAVAnalyticsDevice
{
    public string network_access
    {
        get
        {
            return "WIFI";
        }
    }

    public string app_version
    {
        get
        {
            return "0.12.34";
        }
    }

    public string network_carrier
    {
        get
        {
            return "China Unicom";
        }
    }

    public string channel
    {
        get
        {
            return "Market Place";
        }
    }

    public string device_id
    {
        get
        {
            return "98DD09BDFDC24E359E0426219E9FA79A";
        }
    }

    public string device_model
    {
        get
        {
            return "Windows 10";
        }
    }

    public string device_brand
    {
        get
        {
            return "ASUS";
        }
    }

    public string iid
    {
        get
        {
            return null;
        }
    }

    public string language
    {
        get
        {
            return "zh-CN";
        }
    }

    public string mc
    {
        get
        {
            return "02:00:00:00:00:00";
        }
    }

    public string platform
    {
        get
        {
            return "Windows";
        }
    }

    public string os_version
    {
        get
        {
            return "10";
        }
    }

    public string device_resolution
    {
        get
        {
            return "1920x1080";
        }
    }

    public string timezone
    {
        get
        {
            return "8";
        }
    }
}
// 然后在初始化的时候传递给 InitAsync 即可：
AVAnalytics.InitAsync(new PC());
```

上述字段的有格式要求，请遵循如下表格：

字段名称|类型|格式|长度|是否必须|说明|示例
--|--|--|--|--|--|--
device_id|string|UUID 或 GUID|<span class="text-nowrap">12-64</span>|是|设备唯一 ID|类似于 "98DD09BDFDC24E359E0426219E9FA79A" 或 "820aab78-afc9-48ba-832b-55b9ee9995d3"
network_access|string|无要求|0-64|否|设备联网方式|"WiFi"<br/>"4g"<br/>"wired"
app_version|string|无要求|0-64|否|应用版本|"1.0.0"<br/>"1.0.1-alpha.11"<br/>"2.0.1-beta.1"
network_carrier|string|无要求|0-64|否|通信服务商|"中国移动"<br/>"中国联通"<br/>"中国电信"<br/>"CMCC"<br/>"China Unicom"
channel|string|无要求|0-64|否|应用分发渠道|"App Store"<br/>"Google Play"<br/>"豌豆荚"<br/>"小米商店"
device_brand|string|无要求|0-64|否|设备生产厂商|"Apple"<br/>"小米"<br/>"魅族"<br/>"一加"
device_model|string|无要求|0-64|否|设备型号|"iPhone6,2"<br/>"XiaoMi 5x"<br/>"Sumsang Note8"
iid|string|UUID 或 GUID 或 hex|0-64|否|设备当前的 installation id|"da6c791d-c698-471f-a48d-ead3bd6f5d13"
language|string|`[a-zA-Z-]{2,32}`，建议阅读 [Wiki](https://en.wikipedia.org/wiki/Language_localisation)|0-64|否|当前设备的语言和地区|"zh-cn"<br/>"zh-tw"<br/>"en-us"
mc|string|MAC 地址|0-64|否|设备的网卡地址|"02:00:00:2D:00:1E"
platform|string|`[a-zA-Z0-9_.-]+`|0-64|否|操作系统名称|"iOS"<br/>"Android"<br/>"Windows"<br/>"Linux"
os_version|string|无要求|0-64|否|操作系统版本|"iOS 11.3"<br/>"8.1"<br/>"NT 6.X"
device_resolution|string|`[0-9]{1,10} x [0-9]{1,10}`|0-64|否|设备屏幕分辨率|"1920x1080"<br/>"960*640"
timezone|int|-12 到 14 之间的整数|-|否|设备所在的相对于 GMT 的时区|8 代表北京时间 <br/> -8 代表 Pacific (PST) 太平洋时间 GMT-8


## 应用打开

一般情况下应用被打开的方式有两种：

- 用户主动打开
- 设备接收到应用的推送消息，用户点击消息而打开应用。

因此 SDK 提供了统计这两种打开方式的接口：

```cs
// 记录本次应用的打开是用户主动打开
AVAnalytics.Current.TrackAppOpened();

// 记录本次应用的打开是来自于推送消息的点击
AVAnalytics.Current.TrackAppOpenedWithPush();
```

## 页面访问

用户访问一个页面是一个持续性的过程，我们关注的是用户在什么时间开始进入，在这个页面停留了多长时间，最后在什么时间结束这三个主要的参数，因此我们提供的接口如下：

```cs
var pageId = AVAnalytics.Current.BeginPage("ChatPage");
// pageId 是本地生成的唯一的 id，它是为了保证这一次记录是一次新的打开
// 有可能用户同时打开了两个聊天页面，而在之后记录关闭页面的时候，需要给予一个 ID 来一一对应
// 关闭页面代码如下：
AVAnalytics.Current.EndPage(pageId);

// 调用这两个方法的时间间隔就是访问页面的时长，SDK 会自动进行计算
```

## 自定义事件

用户点击了某个按钮或者使用了某一个触屏手势可以使用如下代码来记录：

```cs
// 记录登录按钮被点击
AVAnalytics.Current.TrackEvent("LogIn_Button_Clicked");
// 记录用户使用了手指滑动的手势
AVAnalytics.Current.TrackEvent("Gesture_Flick");
```

### 多标签事件
有一些事件并不是只有一个名字，它也可能有一些其他标签或者自定义属性，例如电商应用需要了解消费者下单之后是否还浏览了页面上的其他广告：

```cs
var orderEventId = AVAnalytics.Current.BeginEvent("Order", "OrderAction", new Dictionary<string, object>()
{
    { "clicked_ads","left" },//用户点击了左边栏的广告
    { "scorll_down",true},//用户是否拉到了页面底部
});
```

### 事件累计
有的时候一次购物用户可能会分批次下几次订单，因此我们也可以记录多次同名事件：

```cs
var orderEventId1 = AVAnalytics.Current.BeginEvent("Order");
var orderEventId2 = AVAnalytics.Current.BeginEvent("Order");
var orderEventId3 = AVAnalytics.Current.BeginEvent("Order");

...
AVAnalytics.Current.EndEvent(orderEventId1);
AVAnalytics.Current.EndEvent(orderEventId2);
AVAnalytics.Current.EndEvent(orderEventId3);

// 分别下单 3次，然后可以调用结束，这样就可以统计每一次下单的时间消耗
```

## 统计数据的发送
默认情况下只要调用下面的接口，SDK 会根据 [数据发送策略](#设置数据发送策略) 进行发送，无需开发者关心里面的发送逻辑：

```cs
// 结束这一次统计
AVAnalytics.Current.CloseSession();
```

## 设置数据发送策略

你可以进入应用的 {% if node=='qcloud' %}**分析** > **统计设置**<span class="text-muted">（左下角）</span> > **数据发送策略**{% else %}[分析 > 统计设置<span class="text-muted">（左下角）</span> > 数据发送策略](/stat.html?appid={{appid}}&os=android#/statconfig/trans_strategoy){% endif %} 在线更改 SDK 端的数据报告发送策略。在没有取到在线配置的发送策略的情况下，会使用默认的发送策略。

以下均为在线配置中的可选策略。

### 启动时发送
【推荐使用】应用程序每次会在启动时向服务器发送一次消息，在应用使用过程中产生的所有消息（包括自定义事件和本次使用时长）都会在下次启动时候发送。如果应用程序启动时处在不联网状态，那么消息将会缓存在本地，下次再尝试发送。

发送策略默认为启动时发送。

### 批量发送

批量发送，默认当消息数量达到 30 条时发送一次。

### 按最小间隔发送

间隔一段时间发送，每隔一段时间一次性发送到服务器。

## 数据时效性

在控制台的 **分析** 页面中，有些报告可以展示实时数据，有些报告则依靠内部离线数据进行分析，因此有时你会看不到当天的数据。

如果当前页面中存在 **日期选择** 选项（通常在页面右上角），你可以以此判断当前的统计结果是否有延迟。如果 **结束日期** 显示为<u>当天日期</u>或在其下拉菜单中有「今日」的选项，则为实时数据。如下图所示，菜单中无「今日」，所以可以判断这是离线数据，统计结果要延迟一天。

<img src="images/analytics_datepicker_for_offline_data.png" alt="" width="231" height="256">
