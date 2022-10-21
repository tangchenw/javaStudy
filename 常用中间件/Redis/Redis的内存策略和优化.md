# Redis的内存策略和优化

## 慢查询

慢查询发送在第三阶段；客户端超时不一定有慢查询，慢查询客户端可能导致客户端慢。

![QQ截图20220630093918](C:\Users\汤琛\Desktop\学习资料\常用中间件\Redis\images\QQ截图20220630093918.png)

慢查询在内存中维护了一个FIFO的固定长度的队列，可以使用slowlog-max-len

slowlog-log-slower-than两个参数来约定慢查询相关信息  

![QQ截图20220630094735](C:\Users\汤琛\Desktop\学习资料\常用中间件\Redis\images\QQ截图20220630094735.png)

## Redis的内存删除策略

redis可以通过expire time来给key设置过期时间  

- [ ] 定期删除
- [ ] 惰性删除
- [ ] 内存淘汰策略

### 定期删除

遍历过期字典表？  <img src="C:\Users\汤琛\Desktop\学习资料\常用中间件\Redis\images\QQ截图20220630100333.png" alt="QQ截图20220630100333" style="zoom:50%;" />

### 惰性删除

被动删除策略， 访问时判断主要删除策略节省CPU资源  

### LRU

**volatile-lru**:从设置了过期时间的数据集中，选择最近最久未使用的数据释放
**allkeys-lru**:从数据集中选择最近最久未使用的数据释放  



### LFU

