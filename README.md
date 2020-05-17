<p align="center">
<b>X Port Forward Server Api文档</b><br>
<b>您可以依据此文档进行开发,Api接口目前全部为GET请求</b>
</p>

## 请求地址

``` powershell
http://[你的服务器IP]:2333/
```

## 参数

``` powershell
字段名 说明
sign  [必传]签名,签名方法为AuthKey|PortForward的md5加密,如:md5('XPORTFORWARD|PortForward')
action  [必传]操作行为,为string
action_data [选传]操作参数,某些操作行为必传,json格式
```

## 响应

``` powershell
Json格式:
{"code":全局状态码,"message":"全局信息","data":{"Code":模块状态码(某些情况下不会含有),"Message":"模块返回信息(某些情况下不会含有)","Status":"模块状态,分为Success和error,主要判断",...(额外信息)}
提示:全局状态码成功也需要判断模块状态和模块状态码!
```

## 全局状态码

``` powershell
状态码 说明
403 Sign签名有误/未传递模块/模块不存在,可根据全局信息查看(Auth Error/Action cannot be empty/Action Not Found)
500 模块调用失败,可根据全局信息查看
200 模块调用成功
```

## 模块状态码

``` powershell
状态码 说明
0 发送重载配置命令失败
1 新增端口转发,未知的转发类别,暂只支持TCP和UDP
2 新增端口转发参数不全
3 端口转发插入到数据库失败
4 端口转发修改参数不全
5 端口转发未找到,无法修改
6 修改端口转发,未知的转发类别,暂只支持TCP和UDP
7 端口转发修改更新到数据库失败
8 端口转发删除参数不全
9 待删除的端口转发记录不存在
10  从数据库中删除端口转发失败
11  端口转发状态更改参数不全
12  待更改的端口转发记录不存在
13  需要更改到的Status状态无效,仅支持ok(启用),stop(禁用)
14  端口转发状态更改更新到数据库失败
15  端口转发详情获取参数不全
16  待获取的端口转发记录不存在
17  端口转发详情获取成功
18  获取端口转发日志参数不全
19  待获取端口转发日志的记录不存在
20  端口转发日志获取成功
21  流量重置参数不全
22  待流量重置的端口转发记录不存在
23  流量重置更新到数据库失败
24  流量重置成功
25  端口转发重载所需参数不全
26  待重载的端口转发不存在
27  新增端口转发所指定的转发名已经存在
28  新增端口转发所指定的监听端口已经存在
29  修改端口转发所指定的监听端口已经存在
30  服务器信息获取成功
31  重载已删除端口转发所需要的参数不足
32  待重载的已删除端口转发仍然存在于数据库,请先删除
33  查看端口转发状态所需参数不全
34  端口转发状态查询成功
```

## 操作类型详情

### 1.新增端口转发(名称:add_forward)
``` powershell
参数列表:
forwardname 端口转发名,必传
proxyip 对方服务器地址,必传
proxyport 对方服务器端口,必传
listenip  监听地址,必传
listenport  监听端口,必传
maxconnnum  最大连接数,必传,udp传2333
ratelimit 带宽限制,单位为KB,不限制传0,必传
forwardtype 端口转发类别,暂只支持tcp及udp,必传

响应:
错误响应同响应示例
成功响应data只返回Status
```

### 2.修改端口转发(名称:edit_forward)
``` powershell
参数列表:
forwardname 端口转发名,必传
proxyip 对方服务器地址,选传
proxyport 对方服务器端口,选传
listenip  监听地址,选传
listenport  监听端口,选传
maxconnnum  最大连接数,选传,udp传2333
ratelimit 带宽限制,单位为KB,不限制传0,选传
forwardtype 端口转发类别,暂只支持tcp及udp,选传

响应:
错误响应同响应示例
成功响应data只返回Status
```

### 3.删除端口转发(名称:del_forward)
``` powershell
参数列表:
forwardname 端口转发名,必传

响应:
错误响应同响应示例
成功响应data返回Status及Data(Data内为销毁前结算流量统计数据[ForwardName,HourTrafficBandwidth])
```
注意:该操作同时会删除所有的转发流量记录,请先使用get_forward_traffic_logs相关操作获取流量日志进行流量计费

### 4.修改端口转发状态(名称:change_status_forward)
``` powershell
参数列表:
forwardname 端口转发名,必传
status 状态,ok(启用),stop(禁用),必传

响应:
错误响应同响应示例
成功响应data只返回Status
```

### 5.获取端口转发详情(名称:get_forward_infos)
``` powershell
参数列表:
forwardname 端口转发名,必传

响应:
错误响应同响应示例
成功响应data返回Status、ProxyIP、ProxyPort、ListenIP、ListenPort、MaxConnNum、RateLimit、ForwardType、UsedBandwidth(单位为KB)
```

### 6.获取端口转发流量日志(名称:get_forward_traffic_logs)
``` powershell
参数列表:
forwardname 端口转发名,必传
un_report_log 是否只传递等待上报的端口转发流量日志,选传,默认false,传递bool

响应:
错误响应同响应示例
成功响应data返回Info,Info为traffic_logs表记录(所有字段,HourUsedBandwidth单位为KB)
```

### 7.重置端口转发已用流量(名称:reset_forward_bandwidth)
``` powershell
参数列表:
forwardname 端口转发名,必传

响应:
错误、成功响应同响应示例
```

### 8.重载端口转发(名称:reload_forward)
``` powershell
参数列表:
forwardname 端口转发名,必传

响应:
错误响应同响应示例
成功响应data只返回Status
```

### 9.转发服务器信息(名称:server_info)
``` powershell
参数列表:
无

响应:
错误响应同响应示例
成功响应data返回SoftVersion(X Port Forward软件版本)、ServerTime(服务器当前时间)、Author(X Port Forward作者,硬编码为Flyqie)
```

### 10.重载已删除端口转发(名称:forward_del_sync)
此操作存在是为了在删除端口转发后自动重载失败的情况下进行修复,一般无需关注
``` powershell
参数列表:
forwardname 端口转发名,必传

响应:
错误响应同响应示例
成功响应data只返回Status
```

### 11.获取端口转发状态(名称:forward_proxy_status)
``` powershell
参数列表:
forwardname 端口转发名,必传

响应:
错误响应同响应示例
成功响应data返回ForwardStatus(当前转发状态,分为Pstopped(未在转发队列中找到)、stopped(转发已停止)、running(正在进行转发))
```

## 流量上报

``` powershell
上报为Json,PHP请使用php://input获取.
示例(HourUsedBandwidth单位为KB):
{"TrafficLogs":{"23":{"ForwardName":"testhttp","HourUsedBandwidth":0,"Time":"2020-05-15T23:06:33+08:00"},"24":{"ForwardName":"testudp3","HourUsedBandwidth":0,"Time":"2020-05-15T23:06:33+08:00"},"25":{"ForwardName":"testhttp","HourUsedBandwidth":87975.4531,"Time":"2020-05-15T23:09:04+08:00"}},"UsedBandwidth":{"testhttp":7982141.5314,"testudp3":0}}

响应返回:
json格式:
成功示例{"success":true}
错误示例{"success":false,"message":"错误信息"}
```
