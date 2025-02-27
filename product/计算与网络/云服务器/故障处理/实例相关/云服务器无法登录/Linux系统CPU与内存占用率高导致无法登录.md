本文档介绍 Linux 云服务器因 CPU 或内存占用率高导致登录卡顿等问题的排查方法和解决方案

## 可能原因

CPU 或内存使用率过高，容易引起服务响应速度变慢、服务器登录卡顿等问题。而引起 CPU 或内存使用率过高的原因可能由硬件因素、系统进程、业务进程或者木马病毒等因素导致。您可以使用 [云监控](https://cloud.tencent.com/document/product/248/13466)，创建 CPU 或内存使用率阈值告警，当 CPU 或内存使用率超过阈值时，将及时通知到您。

## 定位工具

**Top**：Linux 系统下常用的监控工具，用于实时获取进程级别的 CPU 或内存使用情况。以下图 top 命令的输出信息为例。
![](https://mc.qcloudimg.com/static/img/8aab6354efba19443ffe88f3ace00794/image.png)
Top 命令的输出信息主要分为两部分，上半部分显示 CPU 和内存资源的总体使用情况：
- 第一行：系统当前时间，当前登录用户个数以及系统负载。
- 第二行：系统总进程数、运行中进程数、休眠、睡眠和僵尸进程数量。
- 第三行：CPU 当前使用情况。
- 第四行：内存当前使用情况。
- 第五行：Swap 空间当前使用情况。

下半部分以进程为维度显示资源的占用情况：
- PID：进程 ID。
- USER：进程所有者。
- PR：进程优先级 NI：NICE 值，NICE 值越小，优先级越高。
- VIRT：使用的虚拟内存大小，单位 KB。
- RES：当前使用的内存大小，单位 KB。
- SHR：使用的共享内存的大小，单位 KB。
- S：进程状态。
- %CPU：更新时间间隔内进程所使用的 CPU 时间的百分比。
- %MEM：更新时间间隔内进程所使用的内存的百分比。
- TIME+：进程使用的 CPU 时间，精确到 0.01s。
- COMMAND：进程名称。

## 故障处理

### 登录云服务器

根据实际需求，选择不同的登录方式登录云服务器。
- 通过第三方软件远程登录 Linux 云服务器。
<dx-alert infotype="notice" title="">
 Linux 云服务器处于 CPU 高负荷状态时，可能出现无法登录状态。
</dx-alert>
- [使用 VNC 登录 Linux 实例](https://cloud.tencent.com/document/product/213/35701)。
<dx-alert infotype="notice" title="">
 Linux 云服务器处于 CPU 高负荷状态时，控制台可以正常登录。
</dx-alert>



### 查看进程占用情况

执行以下命令，查看系统负载，并根据 `%CPU` 列与 `%MEM` 列，确定占用较多资源的进程。
```shellsession
top
```

### 分析进程
根据任务管理器中的进程，分析与排查问题，以采取对应解决方案。
- 如果是业务进程占用了大量 CPU 或内存资源，建议分析业务程序是否有优化空间，进行优化或者 [升级服务器配置](https://cloud.tencent.com/document/product/213/2178)。
- 如果是异常进程占用了大量 CPU 或内存资源，则实例可能中毒，您可以自行终止进程或者使用安全软件进行查杀，必要时考虑备份数据，重装系统。
- 如果是腾讯云组件进程占用了大量 CPU 或内存资源，请通过 [在线支持](https://cloud.tencent.com/online-service?from=doc_213) 联系我们进行进一步定位处理。
常见的腾讯云组件有：
 - sap00x：安全组件进程
 - Barad_agent：监控组件进程
 - secu-tcs-agent：安全组件进程


### 终止进程

1. 根据分析的占用资源的进程情况，记录需要终止的进程 PID。
2. 输入 `k`。
3. 输入需要终止进程的 PID ，按 **Enter**。如下图所示：
此处以终止 PID 为23的进程为例。
![](https://main.qcloudimg.com/raw/38a98b3fc36b09c4e3f99765d3cf5691.png)
<dx-alert infotype="notice" title="">
若按 **Enter** 后出现 `kill PID 23 with signal [15]:`，则继续按 **Enter** 保持默认设定即可。
</dx-alert>
4. 操作成功后，界面会出现` Send pid 23 signal [15/sigterm] ` 的提示信息，按 **Enter** 确认即可。

## 其它相关故障
### CPU 空闲但高负载情况处理

#### 问题描述

Load average 是 CPU 负载的评估，其值越高，说明其任务队列越长，处于等待执行的任务越多。
通过 top 观察，类似如下图所示，CPU 很空闲，但是 load average 却非常高。
![](//mc.qcloudimg.com/static/img/4ddf663a68ee602d8cf8075d88edccf6/image.png)
 
#### 处理办法
 
执行以下命令，查看进程状态，并检查是否存在 D 状态进程。如下图所示：
```
ps -axjf
```
![](//mc.qcloudimg.com/static/img/32420d3fe022b57d85120c941705dbf6/image.png)
<dx-alert infotype="explain" title="">
D 状态指不可中断的睡眠状态。该状态进程无法被杀死，也无法自行退出。
</dx-alert>
若出现较多 D 状态进程，可通过恢复该进程依赖资源或重启系统进行解决。

### Kswapd0 进程占用 CPU 较高处理

#### 问题描述

Linux 系统通过分页机制管理内存的同时，将磁盘的一部分划出来作为虚拟内存。而 kswapd0 是 Linux 系统虚拟内存管理中负责换页的进程。当系统内存不足时，kswapd0 会频繁的进行换页操作。换页操作非常消耗 CPU 资源，导致该进程持续占用高 CPU 资源。
 
#### 处理办法

1. 执行以下命令，找到 kswapd0 进程。
```shellsession
top
```
2. 观察 kswapd0 进程状态。
若持续处于非睡眠状态，且运行时间较长并持续占用较高 CPU 资源，请执行 [步骤3](#kswapd0_step3)，查看内存的占用情况。
3. [](id:kswapd0_step3)执行 `vmstat` ，`free`，`ps` 等指令，查询系统内进程的内存占用情况。
根据内存占用情况，重启系统或终止不需要且安全的进程。如果 si，so 的值也比较高，则表示系统存在频繁的换页操作，当前系统的物理内存已经不能满足您的需要，请考虑升级系统内存。


