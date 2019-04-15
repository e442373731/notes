# Redis proxy注意点
之前在使用公司redis proxy时踩了一些坑，记录下
## 1. 超时时间设置
- idle-timeout是jedis pool空闲资源的超时时间，在超时后会将所有空闲的连接清空并新建min数量的连接，具体实现在`GenericObjectPool<T>`
- pool-timeout是jedis pool新建jedis连接的时候等待超时的时间
- jedis-timeout是jedis在执行命令等待server端响应的超时时间
## 2. pipeline相关
- 如果在pipeline中的指令只有set类，则`pipeline.sync()`执行之后执行失败的指令无法返回失败结果或者异常，一般如下处理
```java
// do lot set
responses.add(pipeline.setex(key, sec, value));
pipeline.sync();
for(Response<String> response : responses) {
	// 如有异常将被抛出
	response.get();
}
```
- finally中执行pipeline.close()
## 启动相关
- 在启动proxy进程时不指定-o参数