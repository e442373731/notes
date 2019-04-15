# magpie与dubbo
magpie与dubbo类似，都是RPC框架，主要使用magpie其支持java to java与c to java。
## dubbo
- 主要有dubbo、rmi、hessian、http与webservice等协议
- dubbo协议是其缺省协议，单一长连接和NIO异步通讯，使用hessian二进制序列化，适合于小数据量大并发的服务调用，以及服务消费者机器数远大于服务提供者机器数的情况 
- rmi协议采用JDK标准的java.rmi.*实现，采用阻塞式（bio）短连接与JDK标准序列化方式
- hessian协议底层采用http进行阻塞式同步传输，使用表单序列化，实质是提供了http服务
- 其他略，对于java来说主要使用dubbo协议
## magpie
- 主要有magpie_binary、up8583、uphead、magpie_rpc等协议
- magpie_binary等协议是c与java之间的通信协议，应用拿到的数据都是byte[]需要手动转成字符串，序列化方式只能是binary它的作用就是将String自动序列化为byte[]且不会自动反序列化，不同协议的区别是数据格式不一样，可能是json或者uphead报文等，对于C来说屏蔽了报文的组织过程。
- magpie_rpc协议是java与java之间的通信协议，magpie暂时只支持此一种，序列化方式有hessian、kryo、jdk等。
## 协议与序列化方式
- 协议指定了RPC的连接方式通信方式（同步异步）与序列化方式，对于magpie C来说还决定了C端报文的格式（uphead还是8583）
- 而序列化方式可以说是报文的编码方式，如JDK与hessian
## JDK与hessian
- Java序列化会把要序列化的对象类的元数据和业务数据全部序列化为字节流，而且是把整个继承关系上的东西全部序列化了。它序列化出来的字节流是对那个对象结构到内容的完全描述，包含所有的信息，因此效率较低而且字节流比较大。但是由于确实是序列化了所有内容，所以可以说什么都可以传输，因此也更可用和可靠。
- 它的实现机制是着重于数据，附带简单的类型信息的方法。就像Integer a = 1，hessian会序列化成I 1这样的流，I表示int or Integer，1就是数据内容。而对于复杂对象，通过Java的反射机制，hessian把对象所有的属性当成一个Map来序列化，产生类似M className propertyName1 I 1 propertyName S stringValue这样的流，包含了基本的类型描述和数据内容。而在序列化过程中，如果一个对象之前出现过，hessian会直接插入一个R index这样的块来表示一个引用位置，从而省去再次序列化和反序列化的时间。这样做的代价就是hessian需要对不同的类型进行不同的处理（因此hessian直接偷懒不支持short），而且遇到某些特殊对象还要做特殊的处理（比如StackTraceElement）。而且同时因为并没有深入到实现内部去进行序列化，所以在某些场合会发生一定的不一致，比如通过Collections.synchronizedMap得到的map。
- 类似于utf-8与gbk的关系