# Memory cache与google LoadingCache
在工作中使用到的内存缓存，一开始自己通过hashMap或者ConcurrentHashMap实现，后来使用google的guava jar中有LoadingCache。
## 1. 构建方式
- Guava Cache有两种创建方式：CacheLoader、callable callback
- CacheLoader方式统一指定了如果get不到结果进行缓存的方法。
- callable可以在get指定key的时候指定缓存方法
```java
	public void testcallableCache()throws Exception{
        Cache<String, String> cache = CacheBuilder.newBuilder().maximumSize(1000).build();  
        String resultVal = cache.get("jerry", new Callable<String>() {  
            public String call() {  
                String strProValue="hello "+"jerry"+"!";                
                return strProValue;
            }  
        });  
        System.out.println("jerry value : " + resultVal);
        
        resultVal = cache.get("peida", new Callable<String>() {  
            public String call() {  
                String strProValue="hello "+"peida"+"!";                
                return strProValue;
            }  
        });  
        System.out.println("peida value : " + resultVal);  
    }
```
- 通过builder构建代码如下
```
	CacheBuilder.newBuilder()
					.maximumSize(maxSize)
					.refreshAfterWrite(expireTime, TimeUnit.SECONDS)
					.concurrencyLevel(16)
					.removalListener(new RemovalListener<String, HotacctInfo>() {
						@Override
						public void onRemoval(RemovalNotification<String, HotacctInfo> notification) {
							String acctCd = notification.getKey();
							logger.info("AcctCd = {} has expire, refresh when next access.", acctCd);
						}
					}).build(new CacheLoader<String, HotacctInfo>() {
						@Override
						public HotacctInfo load(String acctCd) throws Exception {
							HotacctInfo acctInfo = hotacctInfoRedisService.getRedisAcctInfo(acctCd);
							// maybe null
							return acctInfo;
						}
					});
```
## 2. 缓存刷新方式
- refreshAfterWrite：与expireAfterWrite类似，但是结果不同
- expireAfterWrite：缓存在给定的时间内超时
- expireAfterAccess：缓存在给定的时间内没有被读写则回收
- expireAfterWrite在多个线程同时访问一个过期key的时候，只有一个线程会去执行load其他线程阻塞。而refreshAfterWrite在这种情况下一个线程去load其他线程则返回旧的数据。
## 3. 注意点
- load方法返回null的话，调用get方法会显示的抛出异常。