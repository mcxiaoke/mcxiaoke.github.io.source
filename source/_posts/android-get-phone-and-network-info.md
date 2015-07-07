Title: Android中检测手机网络类型
Date: 2010-11-03 23:22
Author: mcxiaoke
Category: Android
Tags: Android, Phone, Network
Slug: android-get-phone-and-network-info

Android中检测手机制式和移动网络类型

**[还没写完，有时间再更新，并添加示例代码]**

Android中与电话功能相关的类是 TelephonyManager
，此类中定义了很多常量，以下分类说明  
获取以下信息需要在AndroidManifest.xml中指定权限

一、 数据连接状态  
获取数据连接状态：int getDataState()  
获取数据活动状态：int getDataActivity()  
常用的有这几个：  

```
int DATA_ACTIVITY_IN 数据连接状态：活动，正在接受数据  
int DATA_ACTIVITY_OUT 数据连接状态：活动，正在发送数据  
int DATA_ACTIVITY_INOUT 数据连接状态：活动，正在接受和发送数据  
int DATA_ACTIVITY_NONE 数据连接状态：活动，但无数据发送和接受  
int DATA_CONNECTED 数据连接状态：已连接  
int DATA_CONNECTING 数据连接状态：正在连接  
int DATA_DISCONNECTED 数据连接状态：断开  
int DATA_SUSPENDED 数据连接状态：暂停
```


二、 移动网络类型  
获取网络类型：int getNetworkType()  
常用的有这几个：  

```
int NETWORK_TYPE_CDMA 网络类型为CDMA  
int NETWORK_TYPE_EDGE 网络类型为EDGE  
int NETWORK_TYPE_EVDO_0 网络类型为EVDO0  
int NETWORK_TYPE_EVDO_A 网络类型为EVDOA  
int NETWORK_TYPE_GPRS 网络类型为GPRS  
int NETWORK_TYPE_HSDPA 网络类型为HSDPA  
int NETWORK_TYPE_HSPA 网络类型为HSPA  
int NETWORK_TYPE_HSUPA 网络类型为HSUPA  
int NETWORK_TYPE_UMTS 网络类型为UMTS  
```

在中国，联通的3G为UMTS或HSDPA，移动和联通的2G为GPRS或EGDE，电信的2G为CDMA，电信的3G为EVDO

三、 手机制式类型  
获取手机制式：int getPhoneType()  

```
int PHONE_TYPE_CDMA 手机制式为CDMA，电信  
int PHONE_TYPE_GSM 手机制式为GSM，移动和联通  
int PHONE_TYPE_NONE 手机制式未知
```

四、 SIM卡状态  
获取SIM卡状态：int getSimState()  

```
int SIM_STATE_ABSENT SIM卡未找到  
int SIM_STATE_NETWORK_LOCKED SIM卡网络被锁定，需要Network PIN解锁  
int SIM_STATE_PIN_REQUIRED SIM卡PIN被锁定，需要User PIN解锁  
int SIM_STATE_PUK_REQUIRED SIM卡PUK被锁定，需要User PUK解锁  
int SIM_STATE_READY SIM卡可用  
int SIM_STATE_UNKNOWN SIM卡未知
```

五、其它信息  

```
String getSimCountryIso()  
返回SIM卡提供商的国家代码  
String getNetworkCountryIso()  
返回ISO标准的国家码，即国际长途区号  
String getSimOperator()  
String getNetworkOperator()  
返回MCC+MNC代码 (SIM卡运营商国家代码和运营商网络代码)(IMSI)  
String getSimOperatorName()  
String getNetworkOperatorName()  
返回移动网络运营商的名字(SPN)  
String getSubscriberId()  
返回IMSI，即国际移动用户识别码  
String getDeviceId()  
如果是GSM网络，返回IMEI；如果是CDMA网络，返回MEID  
String getSimSerialNumber()  
返回SIM卡的序列号(IMEI)  
String getLine1Number()  
返回手机号码，对于GSM网络来说即MSISDN  
boolean isNetworkRoaming()  
返回手机是否处于漫游状态
```

解释：  
IMSI是国际移动用户识别码的简称(International Mobile Subscriber
Identity)  
IMSI共有15位，其结构如下：  
MCC+MNC+MIN  
MCC：Mobile Country Code，移动国家码，共3位，中国为460;  
MNC:Mobile NetworkCode，移动网络码，共2位  
在中国，移动的代码为电00和02，联通的代码为01，电信的代码为03  
合起来就是（也是Android手机中APN配置文件中的代码）：  
中国移动：46000 46002  
中国联通：46001  
中国电信：46003  
举例，一个典型的IMSI号码为460030912121001

IMEI是International Mobile Equipment Identity
（国际移动设备标识）的简称  

IMEI由15位数字组成的"电子串号"，它与每台手机一一对应，而且该码是全世界唯一的  
其组成为：  
1. 前6位数(TAC)是"型号核准号码"，一般代表机型  
2. 接着的2位数(FAC)是"最后装配号"，一般代表产地  
3. 之后的6位数(SNR)是"串号"，一般代表生产顺序号  
4. 最后1位数(SP)通常是"0"，为检验码，目前暂备用

