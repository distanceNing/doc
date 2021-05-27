### etcd架构图：

<img src="..\picture\34486534722d2748d8cd1172bfe63084.png" alt="img" style="zoom: 33%;" />

租约机制：







#### Watcher:

##### 第一，client 获取事件的机制，etcd 是使用轮询模式还是推送模式呢？两者各有什么优缺点？

​	V2用的是轮询的方式，没有事件发生的时候，依然占用资源。

​	V3采用http/2流式协议，采用server端流式推送事件给watcher。

HTTP/2 协议的多路复用

##### 第二，事件是如何存储的？ 会保留多久？watch 命令中的版本号具有什么作用？

​	所有的事件都保存到了boltdb

##### 第三，当 client 和 server 端出现短暂网络波动等异常因素后，导致事件堆积时，server 端会丢弃事件吗？若你监听的历史版本号 server 端不存在了，你的代码该如何处理？

​	不会 加入victim事件列表中

##### 第四，如果你创建了上万个 watcher 监听 key 变化，当 server 端收到一个写请求后，etcd 是如何根据变化的 key 快速找到监听它的 watcher 呢？