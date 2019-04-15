# JedisSentinelPool and JedisResourcePool
前者是Jedis jar包默认jedis连接池实现，后者是基于Redis proxy的jedis连接池实现，两者都是`redis.clients.util.Pool`的实现
## JedisSentinelPool
- 配置文件中配置Redis sentinel地址，在构造函数中通过sentinel的地址获取master的ip与端口
- 使用获取到的地址初始化Sentinel pool的RedisFactory（PooledObjectFactory）并在makeObject中实现连接redis master
- 代码中的负载均衡可以自己实现，初始化多个JedisSentinelPool bean，它们连接的master是同一个
## JedisResourcePool
- 配置文件中配置的是Redis proxy的地址，在构造函数中使用proxy地址构建RedusResourceFactory
- RedusResourceFactory实现makeObject方法，方法中连接Redis proxy
- 负载均衡由JedisBalancer实现，其内部维护了一个JedisResourcePool list
- 在JedisResourcePool中有一个JceCount用于计算getResource与returnResource超时的次数，默认5次之后就会将proxy节点隔离