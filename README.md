# deviceid-x-gorgon
iOS抖音风控加密算法的来龙去脉


# iOS抖音风控加密算法的来龙去脉(一)

抖音通信协议的加密算法是目前最完善的了，一些关键函数都被VM混淆过 ，比如设备注册、视频信息等常用接口，只能通过动态调试跟踪去理解其过程。

我们先来分析一下常用的设备注册是如何生成的，这是请求抖音接口的第一步，如果没有它，请求抖音的任何接口都不会返回数据的。

#### 1.抖音的设备注册接口

```
https://log.snssdk.com/service/2/device_register/?
```
method：POST

body：设备信息加密数据

url参数：设备信息参数  

#### 2.设备信息参数生成
device_id的生成是根据我们提交给抖音里的参数进行计算的，所以我们要随机生成一些参数。

##### 重点参数:
> carrier
display_name字段：这个字段不是utf-8编码，是GBK编码，要做编码转换

> Idfa、VendorID字段：标准UUID算法生成即可

> Openudid：随机生成的

跟踪调试过程省略……

通过动态调试最后定位到sub_101E7830设备加密函数

sub函数的参数是传入一个字典

```
{
    fingerprint = "";
    header =     {
        access = WIFI;
        aid = 1128;
        "app_language" = zh;
        "app_name" = aweme;
        "app_region" = CN;
        "app_version" = "8.7.1";
        carrier = "\U4e2d\U56fd\U79fb\U52a8";
        channel = AppStore;
        custom =         {
            "app_language" = zh;
            "app_region" = CN;
            "build_number" = 87100;
            "earphone_status" = on;
        };
        "device_id" = ;
        "device_model" = "iPhone X";
        "display_name" = "\U6296\U97f3\U77ed\U89c6\U9891";
        idfa = "E3D93D3-U747-R394-E2033-HF383J3984JE";
        "install_id" = ;
        "is_jailbroken" = 0;
        "is_upgrade_user" = 1;
        language = zh;
        mc = "00:00:00:00:00:00";
        "mcc_mnc" = "";
        openudid = 9234923948d9392934dkk3939935d93939r3a3s3;
        os = iOS;
        "os_version" = "12.1";
        package = "com.ss.iphone.ugc.Aweme";
        region = CN;
        resolution = "1024*768";
        "sdk_version" = 0011;
        timezone = 1;
        "tz_name" = "Asia/Shanghai";
        "tz_offset" = 99000;
        "user_agent" = "Aweme 8.7.1 rv:87100 (iPhone; iPhone OS 12.1; zh_CN) Cronet";
        "vendor_id" = "6J3DJD34-3DE4-R3KD-DS33-739394839384";
    };
    "magic_tag" = "ss_app_log";
}

```

我们所要替换的就是json里的vendor_id，openudid，idfa然后进行加密

加密过程的算法实际上是AES，首先使用标准Gzip压缩参数然后调AES进行加密。Android和iOS加密方法相同。

##### 流程图如下：
![流程图](https://github.com/shenydowa/deviceid-x-gorgon/blob/master/1.png)


完成上述步骤就可以提交设备注册请求了，成功请求结果如下：

```
{
	"server_time": 1557474647,
	"device_id": 61853858364,
	"install_id": 99375638378,
	"device_id_str": "61853858364",
	"install_id_str": "99375638378",
	"new_user": 1
}
```


iOS抖音8系列版本针对设备注册的最新风控多了一个log算法。

#### 3.问答交流
如果有什么不懂的可以联系我交流

邮箱：
shenydowa@outlook.com


测试地址：
https://www.showdoc.cc/527312186916410?page_id=3113306537235933



#### 4.免责声明

请勿使用本服务于商用或大量抓取
若因使用本服务与抖音官方造成不必要的纠纷，本人盖不负责，纯粹技术爱好，若有侵权行为，可以与本人沟通。

#### 5.其他
除了device_register算法还有mas,as,cp,X-gorgon算法，我会在下一篇讲解。
[iOS抖音风控加密算法的来龙去脉之X-gorgon算法mas、as、cp算法(二)]()


