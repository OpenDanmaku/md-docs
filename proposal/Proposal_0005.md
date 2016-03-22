# 高效率的仓库同步协议

## 同步功能分析

1. 由于数据库只增查不删改，所以合并仓库只需比对交换键的数组。
2. 键表经过排序后，其SHA1的值可以唯一指代当时的仓库全部数据。
3. 命名这个SHA1为更新，比对更新非常有效率：
  - 如果更新相等，意味着仓库已经完全一致，无需再比对键表和文档。
  - 如果更新在历史更新之中，则只需交换历史更新之后的文档增的量。
  - 其他情形下应先交换键表再同步差异文档，比直接交换文档更有效。

## 同步合并步骤

1. 同步双方握手并商议好传输协议。
2. 提出同步请求的一方（一般是插入了新文档或者被同步）发出请求。
  - m为方法，r为仓库名，c为当前更新，s为会话。
  ```json
  {
    "m":"CONST_SYNC",
    "r":"160-bit raw info-hash string",
    "c":"160-bit raw commit-hash string",
    "s":"session string"
  }
  ```
3. 被请求的一方回应请求：
  - r与s与请求相同。c为本地更新，h为请求方更新是否在被请求方的历史中。
  ```json
  {
    "m":"CONST_SYNC_RESPONSE",
    "r":"160-bit raw info-hash string",
    "c":"160-bit raw commit-hash string",
    "h":true,
    "s":"session string"
  }
  ```
4. 请求方返回，被请求方等待请求方下一条报文：
  - h表示被请求方更新是否在被请求方的历史中。
  ```json
  {
    "m":"CONST_SYNC_RESPONSE_RESPONSE",
    "h":true,
    "s":"session string"
  }
  ```
5. 如双方更新一致，或会话中出现第三次更新不一致（后者双方都应该撤销合并），请求方发送结束消息，双方结束通信。   
  - 我没想清楚4的报文是不是应该和567的捏合在一起……
  ```json
  {
    "m":"CONST_SYNC_CLOSE",
    "s":"session string"
  }
  ```
6. 如双方更新不一致，当任意一方表示对方更新不在本地更新历史中，或会话中第二次出现更新不一致：   
   请求方发送完整键表，接受方返回完整键表，跳转7：    
  - k代表键表。
  ```json
  {
    "m":"CONST_SYNC_KEY",
    "r":"160-bit raw info-hash string",
    "k":[
      "160-bit raw document key string",
      "160-bit raw document key string",
      "160-bit raw document key string"
    ],
    "s":"session string"
  }
  ```
  ```json
  {
    "m":"CONST_SYNC_KEY_RESPONSE",
    "r":"160-bit raw info-hash string",
    "k":[
      "160-bit raw document key string",
      "160-bit raw document key string",
      "160-bit raw document key string"
    ],
    "s":"session string"
  }
  ```
7. 如c不一致，但双方的更新都在双方的更新历史中，或由6跳转而来：   
   请求方发送差异键表，接受方返回差异键表，跳转2：    
  - d代表文档。
  ```json
  {
    "m":"CONST_SYNC_VALUE",
    "r":"160-bit raw info-hash string",
    "d":{
      "160-bit raw document key string":"document value string",
      "160-bit raw document key string":"document value string",
      "160-bit raw document key string":"document value string"
    },
    "s":"session string"
  }
  ```
  ```json
  {
    "m":"CONST_SYNC_VALUE_RESPONSE",
    "r":"160-bit raw info-hash string",
    "d":{
      "160-bit raw document key string":"document value string",
      "160-bit raw document key string":"document value string",
      "160-bit raw document key string":"document value string"
    },
    "s":"session string"
  }

Proposal_0005 by Schezuk on 2015.04.28
