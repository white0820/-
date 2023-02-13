### windows增加端口转发

```shell
netsh interface portproxy add v4tov4 listenaddress=10.128.240.24 listenport=25999 connectaddress=192.168.232.8 connectport=22

netsh interface portproxy add v4tov4 listenaddress=192.168.43.62 listenport=25999 connectaddress=192.168.232.8 connectport=22

netsh interface portproxy show v4tov4

# 同一局域网主机a连接本主机b的虚拟机
1. 在主机b的防火墙中添加入站规则，开放某个tcp端口，如25999
2. 在主机a的xshell上添加回话，ip为局域网中主机b的ip，端口号为主机b防火墙开放的25999端口
3. 在主机b中的windows powershell（管理员）上，输入上面的命令添加端口转发的功能，将本机25999端口接收到的信息发送到虚拟机的22端口，进行ssh连接

```

![image-20221122213018250](C:\Users\86186\AppData\Roaming\Typora\typora-user-images\image-20221122213018250.png)



### 查看ssh服务是否打开

ps -e | grep ssh



### 开启ssh服务

service ssh start



### grpc生成go相关文件命令

*.pb.go中是输入输出数据类型的Golang定义

*_grpc.pb.go中是客户端和服务端的gRPC代理类型和方法的Golang定义

```protobuf
protoc --proto_path=proto --go_out=pb proto/*.proto

protoc --proto_path=proto proto/*.proto --go_out=plugins=grpc:pb

protoc --proto_path=proto proto/*.proto --go-grpc_out=pb


protoc -I ecommerce ecommerce/product_info.proto --go_out=plugins=grpc:<module_dir_path>/ecommerce

protoc --proto_path=. --go_out=. ecommerce/*.proto

--- 
protoc --proto_path=. --go_out=. ./*.proto

protoc --proto_path=. --go-grpc_out=. ./*.proto
---

protoc --proto_path=. --go-grpc_out=require_unimplemented_servers=false:. ./*.proto
```





### docker相关文章

docker学习路线：https://zhuanlan.zhihu.com/p/26221700

基于docker镜像部署go项目：https://blog.csdn.net/MoFengLian/article/details/101035531?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-101035531-blog-90578294.pc_relevant_multi_platform_whitelistv4&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-101035531-blog-90578294.pc_relevant_multi_platform_whitelistv4&utm_relevant_index=2

[Ubuntu]配置go命令永久生效：https://blog.csdn.net/Balmunc/article/details/125558112

linux设置gopath：https://blog.csdn.net/Shuffle_Ts/article/details/112952538

protoc安装：https://blog.csdn.net/weixin_44504392/article/details/122196887

ubuntu安装protoc go插件：https://blog.csdn.net/zxueb/article/details/119468181

ubuntu下go version时go command not found：https://blog.csdn.net/qq_27447001/article/details/101430695?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-101430695-blog-120225876.pc_relevant_aa_2&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-101430695-blog-120225876.pc_relevant_aa_2&utm_relevant_index=1

ubuntu安装make：https://blog.csdn.net/qq_19734597/article/details/103125602

goland远程linux开发：https://www.cnblogs.com/haima/p/16298440.html

<font color="yellow">vim使用技巧</font>https://blog.csdn.net/weixin_37552816/article/details/126815498



### grpc

grpc编译失败：https://blog.csdn.net/HYZX_9987/article/details/124972708

我们定义的proto文件是涉及了RPC服务的 ，而默认是不会生成RPC代码的，因此需要在go_out中给出plugins参数，将其传递给protoc-gen-go插件，即告诉编译器，请支持RPC（这里指定了内置的grpc插件）

go-grpc 代码库及其工具：https://blog.csdn.net/xk_moving/article/details/117660152

深入解析grpc源码：https://zhuanlan.zhihu.com/p/438808354

一文了解protoc的使用：https://juejin.cn/post/6949927882126966820

protoc-gen-grpc-go生成的grpc Server接口中带mustEmbedUnimplemented***方法解决办法：https://blog.csdn.net/Mirale/article/details/122736894

golang的序列化protobuf篇：https://www.cnblogs.com/yinzhengjie2020/p/12741943.html



### go context包

go context包使用：https://blog.csdn.net/qq_36769368/article/details/126121110

chan+select控制goroutine结束：https://www.cnblogs.com/xxswkl/p/14219706.html

go context包：https://www.jb51.net/article/244587.htm



### go相关资源

go restful api：https://tutorialedge.net/software-eng/designing-a-rest-api/



### centos7安装musql8

安装mysql时的问题（结合mysql8 cookbook一起进行安装）：https://blog.csdn.net/lyouhuan/article/details/124868523

用navicat连接mysql数据库时常报的错误：2003、1698、1251：https://blog.csdn.net/m0_67402970/article/details/125383662

安装mysql8：

```shell
rpm -Uvh "https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm"

yum repolist enabled | grep 'mysql.*-community.*'

yum repolist all | grep mysql

yum install yum-utils.noarch -y
yum-config-manager --disable mysql57-community
yum-config-manager --enable mysql80-community

yum repolist all | grep mysql8

yum install -y mysql-community-server
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022

rpm -qa | grep -i 'mysql.*8.*'
```

```shell
systemctl status mysqld
systemctl start mysqld
systemctl stop mysqld
```

```shell
grep 'temporary password' /var/log/mysqld.log
mysql -uroot -p
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Hyy0820..';
GRANT ALL ON *.* TO 'root'@'%';

select host, user, authentication_string, plugin from user; 

ALTER USER 'root'@'localhost' IDENTIFIED BY 'Hyy0820..' PASSWORD EXPIRE NEVER; 
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'Hyy0820..'; 

FLUSH PRIVILEGES; 
```

### go相关文章

- [ ] go数据结构map和set：https://www.jianshu.com/p/5a9a2f8ef69b


- [ ] go语言map：https://mp.weixin.qq.com/s/2CDpE5wfoiNXm1agMAq4wA


- [ ] go切片slice：https://zhuanlan.zhihu.com/p/449851884







## docker容器内安装软件

linux通过apt-get安装yum命令：https://blog.csdn.net/m0_48766085/article/details/125969579

docker中安装ps，使用ps -ef | grep ...

```shell
apt-get update && apt-get install procps
apt-get install vim
apt-get install yum # 安装yum
```









## bm5

```go
	head := &ListNode{}
    numOfVals := 0
    // 算出一共有多少个数
    for _, list := range lists {
		cur := list
        for cur != nil {
            numOfVals++
            cur = cur.Next
        }
	}
    // 将所有数字放入切片中
    nums := make([]int, numOfVals)
    i := 0
    for _, list := range lists {
        cur := list
        for cur != nil {
            nums[i] = cur.Val
            i++
            cur = cur.Next
        }
    }
    for i := 0; i < numOfVals-1; i++ {
        if nums[i] > nums[i+1] {
            temp := nums[i+1]
			nums[i+1] = nums[i]
			nums[i] = temp
        }
    }
```























