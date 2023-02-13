# redis 准备

## java 客户端

1. jedis实例是线程不安全的
2. lettuce基于netty实现。支持同步、异步和响应式编程，并且线程安全。支持redis的哨兵模式、集群模式和管道模式
3. redisson是一个基于redis实现的分布式、可伸缩的java数据结构集合。





## centos开放6379端口

查询端口是否开放。若有服务开启，除了要在防火墙处查看该端口是否开放之外，还要查询是否服务正在监听该端口。

在windows上使用cmd，输入telnet ip port测试端口是否开放

```shell
telnet 192.168.232.8 22

telnet 192.168.232.8 6379
```

---

centos防火墙，查看端口开放情况

centos7防火墙firewall相关命令：https://www.cnblogs.com/marso/archive/2018/01/06/8214927.html

```shell
# 查看防火墙状态
firewall-cmd --state
# 查看防火墙所有开放的端口
firewall-cmd --zone=public --list-ports
# 开放6379端口
firewall-cmd --zone=public --add-port=6379/tcp --permanent
# 配置立即生效
firewall-cmd --reload

# 查看防火墙运行情况
systemctl status firewalld
# 关闭防火墙（临时关闭防火墙，系统重新启动后，会再次开启防火墙）
systemctl stop firewalld
# 开启防火墙
systemctl start firewalld

# 禁用防火墙服务（永久关闭防火墙服务，想要开启需要手动开启）
systemctl disable firewalld.service
systemctl enable firewalld.service  # 开启防火墙服务
```



<font color="lightgreen">**🌶️防火墙添加端口转发规则**</font>

```shell
# firewall启动NAT端口转发
firewall-cmd --permanent --zone=public --add-masquerade
# 查看防火墙规则
firewall-cmd --list-all
# 将主机803端口请求转发到容器上的80端口
firewall-cmd --add-forward-port=port=6379:proto=tcp:toaddr=172.17.0.2:toport=6379 --permanent
```



---

查看端口的监听情况

```shell
netstat -lnpt

# centos7默认没有netstat命令，需要安装net-tools工具
yum install -y net-tools
```



## docker redis容器

```shell
# 创建redis容器，名字为myredis
docker run --restart=always --log-opt max-size=100m --log-opt max-file=2 -p 6379:6379 --name myredis -v /home/redis/myredis/myredis.conf:/etc/redis/redis.conf -v /home/redis/myredis/data:/data -d redis:6.0.8 redis-server /etc/redis/redis.conf  --appendonly yes

# 进入redis容器
docker exec -it myredis /bin/bash

# 创建redis容器，名字为yourredis
docker run --restart=always --log-opt max-size=100m --log-opt max-file=2 -p 6380:6379 --name yourredis -v /home/redis/myredis/myredis.conf:/etc/redis/redis.conf -v /home/redis/myredis/data2:/data -d redis:6.0.8 redis-server /etc/redis/redis.conf  --appendonly yes
```







## 资料

redis.conf配置文件详解：https://blog.csdn.net/sinat_25207295/article/details/117925174

docker创建redis容器：https://blog.csdn.net/weixin_45821811/article/details/116211724

ubuntu中apt使用以及centos中yum的使用：http://t.zoukankan.com/lgh344902118-p-7519012.html

## 问题

1. 我用java写的连接redis的程序始终运行不起来，于是我创建两个redis容器，用其中一个容器去访问另一个容器的redis服务。发现服务连接成功，但是添加数据时，发现是保护模式，于是我将保护模式关闭，在redis.conf里面将protected-mode设置为no。再次启动服务发现成功。

2. redis客户端中关闭服务的命令是shutdown

3. redis.conf配置中有一项是bind 127.0.0.1，要想进行远程连接redis数据库，需要将这一选项注释掉。

4. redis远程连接的命令是

   ```shell
   redis-cli -h 172.17.0.2 -p 6379
   ```

![image-20221204015312958](C:\Users\86186\AppData\Roaming\Typora\typora-user-images\image-20221204015312958.png)





# go Redis实战

go redis使用：https://www.cnblogs.com/itbsl/p/14198111.html



# 面试问题

<img src="C:\Users\86186\AppData\Roaming\Typora\typora-user-images\image-20221207040813475.png" alt="image-20221207040813475" style="zoom: 80%;" />

## 1. redis基础

1. [redis和memcached的区别](https://www.php.cn/faq/415539.html)（补充：优缺点和各自场景？）
2. [redis数据类型和常用场景](https://blog.csdn.net/hudeyong926/article/details/99540705)
3. [redis基础](https://mp.weixin.qq.com/s/aOiadiWG2nNaZowmoDQPMQ)（）
4. redis module模块



基础：

1. string类型底层数据结构sds的优点

## 2. redis高可用

主从模式可以大幅提升redis服务的可用性。

故障隔离与恢复：不管主节点还是从节点宕机，其他节点依然可以保证服务的正常运行并且可以手动切换主从。

读写隔离：主节点负责写服务，从节点负责读服务，分摊流量压力，均衡流量的负载。

提供高可用保障：主从模式是高可用的最基础版本，是哨兵模式和cluster模式的前置条件。

但是手动切换主从节点的修复时间太长，需要自动感知故障，实现故障自动转移。

## *** golang面试题

初级golang后端工程师面试题https://juejin.cn/post/7198168315080425531

数据库*

网络*

消息中间件 memcache、kafka



# go基础知识

## 结构体与函数传参

参考：Go语言结构体：https://blog.csdn.net/qq_27870421/article/details/118489821



1. 结构体是值类型
2. Go语言中的函数传参方式全部都是值传递，不存在引用传递
3. 将结构体作为函数参数进行传递时，是进行值传递，也即传递的是该结构体的拷贝。在方法传递时希望传递结构体地址（比如需要修改结构体内部参数的值时），可以使用结构体指针完成。
4. 可以使用new(T)函数创建结构体指针

```go
type People struct {
    Name string
    Age  int
}

peo := new(People)
// 因为结构体本质是值类型，所以创建结构体指针时已经开辟了内存空间
fmt.Println(peo==nil) // 输出：false

// 如果不使用new(T)函数，可以直接声明结构体指针并赋值
var peo2 *People
peo2 = &People{"hyy", 22}
```

### 切片与函数传参

切片事实是一个结构体，底层是用数组实现的

```go
// 切片的结构
type slice struct {
	array unsafe.Pointer // array的值是底层数组的地址
	len   int
	cap   int
}

```

把切片作为函数参数进行传递时，传递的切片的值，但是在函数内对切片进行修改操作时，会影响到函数外的原始切片，因为虽然函数内的切片时原始切片的复制，但是array的值不变，指向的是原始切片的底层数组，因此当修改函数内部的切片的值时，原始切片也会改变。下面是一个例子：

```go
func main() {
    var arr = []int{1, 2, 3, 4, 5}
    fmt.Printf("arr pointer: %p\n", &arr)
    test(arr)
    fmt.Printf("arr: %v\n", arr)
    
    s1 := []int{1, 2, 3, 4, 5}
    a1 := [5]int{1, 2, 3, 4, 5}
    fmt.Println(unsafe,Sizeof(s1)) //24 = 3 * 8
    fmt.Println(unsafe.Sizeof(a1)) // 40 = 5 * 8
}


func test(data []int) {
    fmt.Printf("data pointer: %p\n", &data)
    data[0] = 100
}
```

上面例子的运行结果能的到两个结果：

1. arr pointer:和data pointer:的输出是不同的。因为函数在将切片作为参数时是值传递。重新创建了一个新的切片结构体，因此二者的地址不相同
2. 打印的结果是arr: [100 2 3 4 5]。这就证明了对函数传递的参数切片的修改会影响到原切片。
3. s1这个切片占24字节，印证了切片的底层结构是由三个成员属性组成的结构体。而a1这个数组占40字节。

### 指针

指针的零值是nil，如果只声明而不初始化，那么这个变量即为零值nil。

对一个声明但是没有初始化的变量，不能直接对其进行赋值。只声明的变量，没有为其分配地址空间。

通过new，可以初始化一个地址。初始化的地址指向的值是该类型的零值。













## three dots的用处

```go
1.为函数定义多个可选参数
func FuncName(args ...int){
    //TODO:代码逻辑
}
上面的函数声明表示FuncName可以接受任意数量的int参数

2. 将切片拆散（扩展）
slice := make([]int, 6)
FuncName(slice...) //将切片slice拆散成6个int型作为函数的参数

3. 在数组声明时，如果元素指定，那么可以不必显式声明数组长度，可以根据元素个数推断，如:
arr := [...]int{1, 2, 3}
```

## 树

深度：树的根节点到任意叶子节点路径上节点的数量。

最大深度：所有叶子节点的深度的最大值。

叶子节点：没有子节点的节点。



### golang 面试题

golang接口https://www.cnblogs.com/heartchord/p/5258781.html

golang init函数https://www.bilibili.com/read/cv12167787/

循环语句与循环控制语句https://blog.csdn.net/momoda118/article/details/121201795

for循环不支持以逗号为间隔的多个赋值语句，必须使用平行赋值的方式来初始化多个变量 

golang切片https://www.cnblogs.com/ourongxin/p/15881332.html

golang类型转换和类型断言http://t.csdn.cn/aB87k

golang基本数据类型转换方式http://t.csdn.cn/d9BgL

go bool类型https://www.weixueyuan.net/a/466.html

go switch类型http://edu.jb51.net/go/go-decision-making-switch-statement.html

make 的作用是初始化内置的数据结构，new 的作用是根据传入的类型分配一片内存空间并返回指向这片内存空间的指针

new和make    (slice, channel, map)   https://blog.csdn.net/qq_39397165/article/details/116957200

切片 https://www.cnblogs.com/lvnux/p/12907356.html

指针接收器和值接收器 http://t.csdn.cn/jMwVk

<font color="red">gomock</font> https://zhuanlan.zhihu.com/p/404350737

接口赋值和接口查询https://blog.csdn.net/lengyuezuixue/article/details/78602245



# go相关项目

gomybatis框架https://zhuxiujia.github.io/gomybatis.io/#/

https://zhuanlan.zhihu.com/p/506415782

https://www.zhihu.com/question/369863905

# go找工作面试相关问题

简历，面试相关流程以及技巧https://studygolang.com/articles/33540，https://studygolang.com/articles/20682

简历项目书写http://t.csdn.cn/CYB41，https://juejin.cn/post/7166516163604643854







