## 服务命名

Mesos-DNS为DC\/OS上运行的Mesos任务定义了DNS顶级域`.mesos`，并通过在此Mesos域中查找A和可选的SRV记录来发现任务和服务。

### A记录

A记录将主机名与IP地址关联在一起。当DC\/OS上的应用启动服务实例时，Mesos-DNS以`<task>.<service>.mesos`的格式生成主机名的A记录，其值为以下选项之一：

运行的服务实例的Agent节点的IP地址

服务实例的网络容器的IP地址（由Mesos容器化器提供）

例如，DC\/OS上的其它服务可以发现由Marathon启动的名为`nirvana`的服务的IP地址，并使用`nirvana.marathon.mesos`的来查找：

```
root:localhost ~$dig nirvana.marathon.mesos

; <<>> DiG 9.9.4-RedHat-9.9.4-29.el7_2.4 <<>> nirvana.marathon.mesos
;; global options: +cmd;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 28116
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:;nirvana.marathon.mesos. IN A

;; ANSWER SECTION:nirvana.marathon.mesos. 60 IN A 192.168.1.73

;; Query time: 3 msec
;; SERVER: 198.51.100.1#53(198.51.100.1)
;; WHEN: 六 11月 26 20:19:05 CST 2016
;; MSG SIZE rcvd: 56
```

此处，`192.168.1.73`为Agent节点的IP地址，如果为`nirvana`容器指定了IP，则会显示容器的IP地址。另外，除了上面显示的`<task>.<service>.mesos`记录，Mesos-DNS还生成一条包含运行任务的Agent节点IP地址的A记录：`<task>.<service>.slave.mesos`。

### SRV记录

SRV记录指定服务的主机名和端口。

对于由名为`myservice`的服务启动的名为`mytask`的任务，Mesos-DNS将生成SRV记录`_mytask._protocol.myservice.mesos`，其中协议为`udp`或`tcp`。如果是tcp服务，则为：`_mytask._tcp.myservice.mesos`。

Mesos-DNS支持使用任务的DiscoveryInfo生成SRV记录。下表展示了生成SRV记录的规则：

| **Service** | ** Container IP Known** | **DiscoveryInfo Provided** | ** Target Host** | **Target Port** | **A Record Target IP** |
| --- | --- | --- | --- | --- | --- |
| \_mytask.\_protocol.myservice.mesos | No | No | mytask.myservice.slave.mesos | Host Port | Agent IP |
| \_mytask.\_protocol.myservice.mesos | Yes | No | mytask.myservice.slave.mesos | Host Port | Agent IP |
| \_mytask.\_protocol.myservice.mesos | No | Yes | mytask.myservice.mesos | DiscoveryInfo Port | Agent IP |
| \_mytask.\_protocol.myservice.mesos | Yes | Yes | mytask.myservice.mesos | DiscoveryInfo Port | Container IP |
| mytask.protocol.myservice.slave.mesos | N\/A | N\/A | mytask.myservice.slave.mesos | Host Port | Agent IP |

### 其它记录

Mesos-DNS生成几个特殊记录：

**leading master：**

A记录：leader.mesos

SRV记录：\_leader.\_tcp.mesos 和 \_leader.\_udp.mesos

**all service schedulers: **

A记录：myservice.mesos

SRV记录：\_myservice.\_tcp.myservice.mesos

**每个DC\/OS master：**

A记录：master.mesos

SRV记录：\_master.\_tcp.mesos 和 \_master.\_udp.mesos

**每个DC\/OS agent:**

A记录：slave.mesos

SRV记录：\_slave.\_tcp.mesos

注意：要查询Master Leader节点，始终使用`leader.mesos`而不是`master.mesos`。

新Master Leader节点的选举与Mesos-DNS更新Master Leader记录之间存在延迟。Mesos-DNS支持对Mesos域的SOA和NS记录的请求。在Mesos域中对其他类型记录的DNS请求将返回NXDOMAIN。 Mesos-DNS不支持用于反向查找所需的PTR记录。

Mesos-DNS为自己生成一条A记录，列出了所有IP地址。这些A记录的主机名是ns1.mesos。

### 查找服务的DNS名称

在Master节点上，通过下述命令可以列出DC\/OS集群上运行的所有应用：

```
curl http://master.mesos:8123/v1/enumerate
```

