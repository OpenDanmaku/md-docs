# Magnet-Danmaku

----
## 什么是Magnet-Danmaku？
简称MD。
是在P2P网络的分布式公共NOSQL服务。
可以用于为BT/Magnet资源提供弹幕服务。

----
## 数据结构
包括三个层次：

1. URI
2. BLOB
3. CONTENT

----
## URI

### 结构
(Peer + )Path

### Peer
Protocol://Domain:Port，或Protocol://PeerHash
PeerHash = 160 bit Peer ID

### Path
/Repository/Commit/Blobhash/
Repository = 160 bit Info Hash
Commit = Hash of Blobs' hashtable
Blob = Hash of Blob

### 范例
http://localhost:1024/0123456789ABCDEF0123/0123456789ABCDEF0123/0123456789ABCDEF0123/

## BLOB
```{
	"s":签名,
	"t":时间戳,
	"f":数据格式(区分c是数据/元数据/超链接),
	"c":签名过的内容
}```
### CONTENT
```{
	"n":文件名,
	"t":时间点,
	"f":数据格式,
	"l":长度,
	"c":内容(如弹幕文本)
}```

----
## 功能组成
主要功能为：

1. DHT组网
2. 用户认证
3. 数据库同步
4. 资源匹配
5. 弹幕客户端

----
## DHT组网

### 目标
使用Kademlia协议，将在同一DHT网络中(如BT-DHT和EMULE-DHT)，关注同一Repository的Peer形成一个连通图，使更新信息得以推送到每个Peer。

### 最好的情况
能找到数学证明，证明MD-Overlay在已有的Kad网络上实现连通图。

### 最坏的情况
静态文件不需要形成连通图，且Kademlia理论上做不到，MD-Overlay孤岛化。(低可能性)

### 备用办法
Gnutella：半分布式，存在超级节点。洪泛广播，确保人人收到。
超级节点：有公网地址的客户端自动升级为超级节点，超级节点始终保持互相同步。

----
## 用户认证
从上至下/从下至上和单一认证/多重认证的组合。

1. 天赋人权：
	Proof-of-work：可能矿机霸占
2. 亚当夏娃：
	证书树（邀请和升级）：可能导致族诛
3. 联合国：
	多证书机构：能否多重签名？
4. 选举与信任：
	我信任朋友，减半信任朋友的朋友……
	问：朋友圈和积分的方式，数学上可行否？
5. 其他：
	用户认证是获取权利的过程，可以和现实世界类比。
	凡是生活中如何攫取权力，都可以借用过来。

----
## 数据库同步

### Peer
与现有的Kademlia或Gnutella网络没有本质区别。
应为如何协商具体传输数据的方式设计一个规范。

### Path
数据库的每个Repository挂在根目录"/"下。
每更新一个Blob，Repository下形成一个Commit。
每个Commit下有多个Blobs。

### 行为

1. 条件：本地存在新Commit，或第一次与其他Peer连接并协商好协议。
2. 行为：向其他Peer发送同步请求，附带自己的Repository和Commit哈希值。
3. 握手：对方确认，并返回自己Repository和Commit哈希值。
4. 确认：发送一致或不一致消息。在后一情况需要交换Blob的哈希表。
5. 比对：类似2-4步，双方比对彼此的Blob哈希表。
6. 请求：向对方请求/Repository/Blobhash，对方返回Blob内容。
7. 结束：双方重新比较自己的Commit。一致则结束，不一致则重复5-6步。不允许只存储Blobhash不存储Blob内容。

----
## 资源匹配
弹幕池与原文件的infohash应该是一一对应的，如果令其相等有困难，那么令其差为常数也可以。

Content级别的数据不但可以封装弹幕文本，图像，脚本等，
也可以封装表示『跨弹幕池引用』的元数据，以构建两个弹幕池的关联。

引用关系的生成方式有：

1. 视频内容识别：直方图/SITH/SURF/缩略图Hash/……
2. 音轨内容识别
3. 文件名智能识别
4. 人工提交

----
##弹幕客户端

通过从后端提供的接口(比如本地的RESTful)
弹幕客户端可以获取P2P网络上的弹幕数据。
弹幕客户端包括但不限于脚本，网页，浏览器插件，可执行程序，APP。

----
## FIN
> Written with [StackEdit](https://stackedit.io/).
