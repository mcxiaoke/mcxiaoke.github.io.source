title: Python标准库笔记之datetime
author: mcxiaoke
categories:
  - Python
tags:
  - Python,datetime
date: 2015-07-15 14:43:14
---

#### datetime

```python

# -*- coding: UTF-8 -*-
__author__ = 'mcxiaoke'

from datetime import date, timedelta, datetime
import time

# 常量 datetime.MINYEAR=1, datetime.MAXYEAR=9999
'''
几个类方法
datetime.date(year,month,day)
datetime.time(hour,minute,second)
datetime.datetime(year,month,day,hour,minute,second)
'''

# date对象
# timedelta对象表示两个时间或日期的差值
delta = timedelta(hours=-5)
# out: datetime.timedelta(-1, 68400)
print delta.total_seconds()
# out: -18000.0
print date.today()  # datetime.date(2015, 7, 4)
print date.fromtimestamp(time.time())  # 2015-07-04

today = date.today()
print today  # 2015-07-04
today == date.fromtimestamp(time.time())
print today  # 2015-07-04
birthday = date(today.year, 10, 01)
print birthday  # 2015-10-01
time_to_birthday = abs(birthday - today)
print time_to_birthday  # 89 days, 0:00:00

# datetime对象，类方法
# 构造函数 datetime(year, month, day[, hour[, minute[, second[, microsecond[, tzinfo]]]]])
# 返回当地时间
datetime.today()  # datetime.datetime(2015, 7, 4, 16, 32, 29, 748628)
# 返回当地时间
datetime.now()  # datetime.datetime(2015, 7, 4, 16, 32, 53, 701843)
# 返回UTC时间
datetime.utcnow()  # datetime.datetime(2015, 7, 4, 8, 34, 14, 433219)
# 根据时间戳构建当地时间
datetime.fromtimestamp(time.time())  # datetime.datetime(2015, 7, 4, 16, 34, 53, 402597)
# 根据时间戳构建UTC时间
datetime.utcfromtimestamp(time.time())  # datetime.datetime(2015, 7, 4, 8, 35, 36, 268517)
# 根据格式和日期字符串构造对象，用法同time模块的strptime()
# datetime.strptime(date_string, format)

# datetime对象，实例方法
now = datetime.now()
print now.date()  # 2015-07-04
print now.time()  # 16:45:29.077727
print now.utcoffset()  # None
print now.timetuple()
# out: time.struct_time(tm_year=2015, tm_mon=7, tm_mday=4,
# tm_hour=16, tm_min=46, tm_sec=29, tm_wday=5, tm_yday=185, tm_isdst=-1)
# 类似的还有 now.utctimetuple()
print now.__str__()  # 2015-07-04 16:48:02.930730
print now.ctime()  # 'Sat Jul  4 16:44:03 2015'
print now.strftime('%Y-%m-%d %H:%M:%S')  # 2015-07-04 16:49:25

# time对象
t = now.time()
print t.isoformat()  # 返回ISO8601格式 16:52:19.700394
print t.utcoffset()  # 返回时区偏移 None



```

#### time

```python

# -*- coding: UTF-8 -*-
__author__ = 'mcxiaoke'

import time

# 相关模块 datetime,calendar, locale

## 时区缩写表示
'''
GMT 格林威治标准时间 Greenwich Mean Time
UTC 协调世界时 Coordinated Universal Time
CST 可同时代表以下四个：
Central Standard Time (USA) UT-6:00
Central Standard Time (Australia) UT+9:30
China Standard Time UT+8:00
Cuba Standard Time UT-4:00
'''

# Unix时间起点是1970年1月1号0点，终点是2038年
time.gmtime(0)
# out: time.struct_time(tm_year=1970, tm_mon=1, tm_mday=1, tm_hour=0,
# tm_min=0, tm_sec=0, tm_wday=3, tm_yday=1, tm_isdst=0)
# UTC=GMT 两种叫法而已
# 返回当前的时间戳
time.time()  # out: 1435994390.769246
# 返回当前UTC/GMT时间，结果为 struct_time
time.gmtime()
# out: time.struct_time(tm_year=2015, tm_mon=7, tm_mday=4, tm_hour=7,
# tm_min=20, tm_sec=35, tm_wday=5, tm_yday=185, tm_isdst=0)
# 返回当前时区的时间，结果为 struct_time
time.localtime()
# out: time.struct_time(tm_year=2015, tm_mon=7, tm_mday=4, tm_hour=15,
# tm_min=21, tm_sec=30, tm_wday=5, tm_yday=185, tm_isdst=0)

time.mktime(time.localtime())  # 转换成时间戳
time.asctime(time.localtime())  # 转换成字符串
# out: 'Sat Jul  4 15:24:18 2015'
time.asctime()  # 默认使用 localtime()
time.ctime()  # 字符串表示的当地时间
# ctime(secs) = asctime(localtime(secs))
# ctime.sleep(seconds)  # 挂起当前线程一段时间，单位为秒

# 时间字符串格式化
# time.strftime(format[,t]) 格式化struct_time，默认是当前时间
# 转换成unicode编码 strftime(<myformat>).decode(locale.getlocale()
print "当前时间戳", time.time()
time_text = time.strftime(
    '%%A    (%A)    #当地星期几缩写\n '
    '%%a    (%a)    #当地星期几全称\n '
    '%%B    (%B)    #当地月份全称\n '
    '%%b    (%b)    #当地月份缩写\n '
    '%%c    (%c)    #当地日期和时间表示\n '
    '%%d    (%d)    #代表日期天数的数字[01,31]\n '
    '%%H    (%H)    #24小时制的小时数[00,23]\n '
    '%%I    (%I)    #12小时制的小时数[01,12]\n '
    '%%j    (%j)    #一年中的第几天[001,366]\n '
    '%%m    (%m)    #代表日期月份的数字[01,12]\n '
    '%%M    (%M)    #代表时间分钟的数字[00,59]\n '
    '%%p    (%p)    #当地表示上午/下午的缩写，如AM/PM\n '
    '%%S    (%S)    #代表时间秒数的数字[00,61]\n '
    '%%U    (%U)    #一年中的第几周[00,53](星期日作为一周的开始)\n '
    '%%w    (%w)    #代表日期星期几的数字[0(星期一),6]\n '
    '%%W    (%W)    #一年中的第几周[00,53](星期一作为一周的开始)\n '
    '%%x    (%x)    #当地的日期表示\n '
    '%%X    (%X)    #当地的时间表示\n '
    '%%y    (%y)    #二位数的年份表示\n '
    '%%Y    (%Y)    #四位数的年份表示\n '
    '%%Z    (%Z)    #当前时区表示\n ')
print "当前时间结构", time.localtime()
print "时间格式举例", time.strftime("%Y-%m-%d %H:%M:%S")
print time_text
# 输出的表示
'''
当前时间戳 1435997058.36
当前时间结构 time.struct_time(tm_year=2015, tm_mon=7, tm_mday=4,
tm_hour=16, tm_min=4, tm_sec=18, tm_wday=5, tm_yday=185, tm_isdst=0)
时间格式举例 2015-07-04 16:04:18
%A    (Saturday)    #当地星期几缩写
 %a    (Sat)    #当地星期几全称
 %B    (July)    #当地月份全称
 %b    (Jul)    #当地月份缩写
 %c    (Sat Jul  4 16:04:18 2015)    #当地日期和时间表示
 %d    (04)    #代表日期天数的数字[01,31]
 %H    (16)    #24小时制的小时数[00,23]
 %I    (04)    #12小时制的小时数[01,12]
 %j    (185)    #一年中的第几天[001,366]
 %m    (07)    #代表日期月份的数字[01,12]
 %M    (04)    #代表时间分钟的数字[00,59]
 %p    (PM)    #当地表示上午/下午的缩写，如AM/PM
 %S    (18)    #代表时间秒数的数字[00,61]
 %U    (26)    #一年中的第几周[00,53](星期日作为一周的开始)
 %w    (6)    #代表日期星期几的数字[0(星期一),6]
 %W    (26)    #一年中的第几周[00,53](星期一作为一周的开始)
 %x    (07/04/15)    #当地的日期表示
 %X    (16:04:18)    #当地的时间表示
 %y    (15)    #二位数的年份表示
 %Y    (2015)    #四位数的年份表示
 %Z    (CST)    #当前时区表示
'''

# 按格式解析时间，返回值为 struct_time
time.strptime('2005-09-01 9:45:20', '%Y-%m-%d %H:%M:%S')
# out: time.struct_time(tm_year=2005, tm_mon=9, tm_mday=1,
# tm_hour=9, tm_min=45, tm_sec=20, tm_wday=3, tm_yday=244, tm_isdst=-1)

# struct_time结构说明
'''
0   tm_year (for example, 1993)
1   tm_mon  取值范围 [1, 12]
2   tm_mday 取值范围 [1, 31]
3   tm_hour 取值范围 [0, 23]
4   tm_min  取值范围 [0, 59]
5   tm_sec  取值范围 [0, 61]
6   tm_wday 取值范围 [0, 6] # 星期一=0
7   tm_yday 取值范围 [1, 366]
8   tm_isdst 取值范围 0, 1 or -1
'''



```

