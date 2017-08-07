---
title: 获取IP地址信息
date: 2017-08-02 16:12:05
tags:
- IP
- API
categories:
- PHP
- IP
---
### 介绍
目前主要的解析方式有两种：通过API，或通过IP数据库。
#### API
**优点**
1、编码简单，通过请求获得数据（通常是json数据）
2、而且不用维护数据库，对本地没有负担。

<!-- more -->
**缺点**
1、有并发限制。每秒只能发几条
2、上限限制。每天请求不超过一个上限。
3、**地址可能面临变更**。

#### 本地数据
**优点**
1、数据存储在本地，只有硬件限制。
2、可以获取任意格式。
3、可以随时取随时用。
**缺点**
1、需要编程解析数据。
2、需要维护数据。
3、需要考虑服务器负载。

**提示** : 前期开发可用API查询,并缓存查询的IP信息,后期根据请求的实际情况使用纯真数据库或GEOIP数据库在本地获取位置信息

### 淘宝IP地址库

提供的服务包括：
1、根据用户提供的IP地址，快速查询出该IP地址所在的地理信息和地理相关的信息，包括国家、省、市和运营商。
2、用户可以根据自己所在的位置和使用的IP地址更新我们的服务内容。


**优势 :**
1、提供国家、省、市、县、运营商全方位信息，信息维度广，格式规范。
2、提供完善的统计分析报表，省准确度超过99.8%，市准确度超过96.8%，数据质量有保障。

**示例 :**

```php
$ip = @file_get_contents("http://ip.taobao.com/service/getIpInfo.php?ip=".$_GET["ip"]);
$ip = json_decode($ip,true);
```

### 新浪IP API

新浪这个应该说是最好的。可以根据请求参数 `format` 自定义数据格式（默认为纯文本格式，`format` 的可选参数`js` ，`json`。下面列举的是 `js` 的格式）。

**示例 :**

```php
$ip = @file_get_contents("http://int.dpool.sina.com.cn/iplookup/iplookup.php?format=js&ip=".$_GET["ip"]);
echo $ip;
```

返回值数据格式：

```plaintext
var remote_ip_info = {“ret”:1,”start”:”59.37.164.179″,”end”:”59.37.165.17″,”country”:”\u4e2d\u56fd”,”province”:”\u5e7f\u4e1c”,”city”:”\u6c5f\u95e8″,”district”:””,”isp”:”\u7535\u4fe1″,”type”:””,”desc”:””};
```

### 126 IP API

126提供的这个功能方面不如新浪但是数据相对来说是比较准确的。

示例 :

```php
$ip = @file_get_contents("http://ip.ws.126.net/ipquery?ip=".$_GET["ip"]);
echo $ip;
```

返回值数据格式：

```plaintext
var lo="北京市", lc="朝阳区"; var localAddress={city:"朝阳区", province:"北京市"};
```

### 腾讯位置服务
　　腾讯支持数据格式 `JSON`, `JSONP`方式返回, 支持 `http` 和 `https` 协议  。

**使用限制**
1、 需申请开发者密钥Key
2、日调用量：`10,000` 次/接口/Key
3、并发限制：`5` 次/秒/接口/Key

超过日调用量和并发数的开发者，可通过以下途径解决：

    对于多频次的相同请求，可通过缓存结果，并定时访问更新的方式，减少对在线服务调用的依赖；
**地址**

```plaintext
http://apis.map.qq.com/ws/location/v1/ip 
```

**请求参数**
<div class="ip-api">

| 参数    | 必填  | 说明                               | 示例                                    |
| ------- | ----- | ---------------------------------- | --------------------------------------- |
| p       | 否    | IP地址，缺省时会使用请求端的IP     | ip=202.106.0.20                         |
| ey      | 是    | 开发密钥（Key）                    | key=OB4BZ-D4W3U-B7VVO-4PJWW-6TKDJ-WPB77 |
| utput   | 否    | 返回格式：支持JSON/JSONP，默认JSON | output=json                             |
| allback | 否    | JSONP方式回调函数                  | callback=function1                      |
</div>
```json
//响应示例：
{
    "status": 0, // 状态码，0为正常,
                 // 310 请求参数信息有误，
                 // 311 Key格式错误,
                 // 306 请求有护持信息请检查字符串,
                 // 110 请求来源未被授权
    "message": "query ok", // 状态说明
    "result": {
        "location": {
            "lng": 116.407526,  // 纬度
            "lat": 39.90403     // 经度
        },
        "ad_info": {
            "nation": "中国",   // 国家
            "province": "",     // 省
            "city": "",         // 市
            "adcode": 110000    // 行政区划代码
        }
    }
}
```

### 百度IP定位

使用以及限制与腾讯基本相同，此处不作详细介绍，请移步官方查看文档：[百度IP定位](http://lbsyun.baidu.com/index.php?title=webapi/ip-api)

### GEOIP

GeoIP IP智能系列产品分类 :
1、 **API接口服务**。 GeoIP根据IP返回的地址信息。
2、 **从本地获取位置信息**。提供网络查询免费开源IP数据库下载。
[直达GitHub仓库PHP版本( 包含 `mmdb` 格式的IP数据库 )](https://github.com/maxmind/GeoIP2-php)

   直达官网链接 [GEOIP](https://dev.maxmind.com/zh-hans/geoip/)

### 纯真IP数据库
下载并安装软件，查找安装目录的文件 `qqwry.dat` ,可转换为数据库文件或通过类库解析使用。
实际IP地址一般为二进制: 01010101.10101010.00110011.11001100。
直达链接 [纯真](https://link.zhihu.com/?target=http%3A//www.cz88.net/)
dat文件解压为txt查看内容
```
0.0.0.0     0.255.255.255    IANA保留地址
1.0.0.0     1.0.0.255        澳大利亚 亚太互联网信息中心
1.0.1.0     1.0.3.255        福建省 电信
1.0.4.0     1.0.7.255        澳大利亚 墨尔本Big Red集团
1.0.8.0     1.0.15.255       广东省电信

```
每行三个字段，start, end, address，从start到end之间的IP都是address这个位置。因为end跟下一个start是连接的，所以只需要记录start: address就好了，可通过查找“小于等于给定IP的最大值”来获取地址。
**参考链接 :**[解析纯真IP地址库](http://www.cnblogs.com/anpengapple/p/5384985.html)

注：如果每一条ip对应一个address ，把所有ip的address写成一个列表？那么服务器得有200G来存放。
