## 1.目前云拨测都支持哪些协议

云拨测目前支持http/https,tcp,ping三种协议进行探测。

## 2.拨测是否支持有登录态的http请求

云拨测目前只支持一次性的http请求，暂不支持诸如带登录态的多步请求方式。

## 3.拨测周期如何选择

云拨测提供了5分钟、15分钟、30分钟四种拨测周期。周期越短，相同时间内拨测的次数就越多，其数据精度就越高，同时也能提供更及时的告警。

## 4.什么是拨测点分组

拨测agent分组主要为了方便拨测点自有搭配组合，将一个或者多个拨测点设定为一个分组。对于每一个开发商，总是存在一个”系统默认组“，这个组是系统自动生成的分组，如果用户不需要自定义，可以直接使用该分组。另外还支持用户自定义拨测点分组。当用户自定义分组后，可以将分组设定为默认分组这样，新建拨测任务时，会默认使用这个分组。

## 5.云拨测数据异常，常见的故障类型有哪些

### **DNS解析失败**:

现象：通常为DNS服务器压力大、客户端和DNS服务器之间的网络不稳定，出现丢包、域名没有生效等。
原因：可能是DNS服务商或者拨测点ISP服务商DNS服务器的故障。
解决方案：建议您在故障日志中找到用于解析的DNS服务器，并联系相关负责人。请选择当地区域网络状况较好的服务器运营商。

### **连接超时**:

现象：拨测agent发送请求后10秒内未能建立连接，说明网络、服务器或后端服务不可用。
原因：可能是后端服务器未能提供正常的服务。
解决方案：建议您检查后台服务软件是否工作正常。

### **响应超时**:

现象：从发送请求到接收完返回数据的时间超过10秒，或未找到可用的连接。
原因：可能服务器繁忙。
解决方案：建议您优化后端服务器性能或者进行扩容。

### **连接失败**:

现象：拨测agent连接后端服务时直接返回错误。
可能原因：拨测agent和后端服务之间的网络不通，或者后端服务未启动。
解决方案：建议检查网络或者后端服务监听的端口是否正常。

### **连接被断开**:

现象：拨测agent和后端服务之间的网络连接被中断。
可能原因：后台服务在重启前，会主动断开所有tcp连接，导致agent到服务器之间网络链路发生问题，丢失socket连接；或者存在IP地址认证，连接数超过最大值等原因导致的后端服务器主动断开连接。
解决方案：建议您检查服务器Web服务程序。

### **Ping超时**:

现象：ping探测时，服务端无响应。
可能原因：如果只有某个拨测点出现该问题，可能是某个地域的某个运营商本身网络存在连通性问题，如果所有拨测点均出现该问题，则很有可能是服务器本身宕机或者失去响应。
解决方案：建议您检查服务器状态或者请[提交工单](http://console.qcloud.com/ticket)进行确认咨询。