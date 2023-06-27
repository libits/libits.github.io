---
title: Hyperledger fabric学习
date: 2021-04-02 11:35:00 +0800
categories: [区块链]
tags: [Hyperledger Fabric]
pin: true
author: 小李学编程

toc: true
comments: true
typora-root-url: ../../libits.github.io
math: false
mermaid: true


---

#  Hyperledger Fabric学习

##  1. Fabric核心模块

|    模块名称     |                     功能                     |
| :-------------: | :------------------------------------------: |
|     `peer`      | 主节点模块, 负责存储区块链数据, 运行维护链码 |
|    `orderer`    |              交易打包, 排序模块              |
|   `cryptogen`   |              组织和证书生成模块              |
|  `configtxgen`  |              区块和交易生成模块              |
| `configtxlator` |              区块和交易解析模块              |

##  2. cryptogen

<font color="red">cryptogen模块主要用来生成组织结构和账号相关的文件</font>

1. cryptogen 配置文件

```yaml
OrdererOrgs:					# 排序节点的组织定义
  - Name: Orderer				# orderer节点的名称
 	Domain: example.com			# orderer节点的根域名 
 	Specs:
	    - Hostname: orderer		# orderer节点的主机名
PeerOrgs:						# peer节点的组织定义
  - Name: Org1					# 组织1的名称	1	1
	Domain: org1.example.com	# 组织1的根域名
 	EnableNodeOUs: true			# 是否支持node.js
 	Template:					
	    Count: 2				# 组织1中的节点(peer)数目
	Users:
 	    Count: 1				# 组织1中的用户数目
  - Name: Org2
    Domain: org2.example.com
    EnableNodeOUs: true
    Template:
        Count: 2
    Users:
        Count: 1
```

##  2. configtxgen

configtxgen 模块的功能一共有两个:

- 生成 orderer 节点的初始化文件
- 生成 channel 的初始化文件

1. configtxgen模块命令

```shell
$ configtxgen --help
Usage of ./configtxgen:
  # 指定所属的组织
  -asOrg string
        Performs the config generation as a particular organization (by name), only 
        including values in the write set that org (likely) has privilege to set
  # 指定创建的channel的名字, 如果没指定系统会提供一个默认的名字.
  -channelID string
        The channel ID to use in the configtx
  # 执行命令要加载的配置文件的路径, 不指定会在当前目录下查找
  -configPath string
        The path containing the configuration to use (if set)
  # 打印指定区块文件中的配置内容，string：查看的区块文件的名字
  -inspectBlock string
        Prints the configuration contained in the block at the specified path
  # 打印创建通道的交易的配置文件
  -inspectChannelCreateTx string
        Prints the configuration contained in the transaction at the specified path
  # 更新channel的配置信息
  -outputAnchorPeersUpdate string
        Creates an config update to update an anchor peer (works only with the default 
        channel creation, and only for the first update)
  # 输出区块文件的路径
  -outputBlock string
        The path to write the genesis block to (if set)
  # 标示输出创始区块文件
  -outputCreateChannelTx string
        The path to write a channel creation configtx to (if set)
  #  将组织的定义打印为JSON(这对在组织中手动添加一个通道很有用)。
  -printOrg string
        Prints the definition of an organization as JSON. (useful for adding an org to
        a channel manually)
  # 指定配置文件中的节点
  -profile string
        The profile from configtx.yaml to use for generation. (default
        "SampleInsecureSolo")
  # 显示版本信息
  -version
        Show version information
```

2. configtxgen模块配置文件

```yaml
Profiles:
	# 组织定义标识符，可自定义，命令中的 -profile 参数对应该标识符， 二者要保持一致
    ItcastOrgsOrdererGenesis:
        Capabilities:
            <<: *ChannelCapabilities	# 引用下面为 ChannelCapabilities 的属性
        Orderer:						# 配置属性，系统关键字，不能修改
            <<: *OrdererDefaults		# 引用下面为 OrdererDefaults 的属性
            Organizations:
                - *OrdererOrg			# 引用下面为 OrdererOrg 的属性
            Capabilities:
                <<: *OrdererCapabilities # 引用下面为 OrdererCapabilities 的属性
        Consortiums:					# 定义了系统中包含的组织
            SampleConsortium:
                Organizations:			# 系统中包含的组织
                    - *OrgGo				# 引用了下文包含的配置
                    - *OrgJava
    # 通道定义标识符，可自定义
    TwoOrgsChannel:	
        Consortium: SampleConsortium
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *OrgGo
                - *OrgJava
            Capabilities:
                <<: *ApplicationCapabilities
                
# 所有的值使用默认的true即可， 不要修改                
Capabilities:
    Global: &ChannelCapabilities
        V1_1: true
    Orderer: &OrdererCapabilities
        V1_1: true
    Application: &ApplicationCapabilities
        V1_2: true
        
# 组织节点相关配置信息
Organizations:
	# orderer节点配置信息
    - &OrdererOrg
        Name: OrdererOrg	# orderer节点名称
        ID: OrdererMSP		# orderer节点编号
        MSPDir: ./crypto-config/ordererOrganizations/itcast.com/msp	# msp文件路径
	#orderer节点中包含的组织，如果有有多个需要配置多个
    - &OrgGo
        Name: OrgGoMSP		# 组织名称
        ID: OrgGoMSP		# 组织编号
        # 组织msp文件路径
        MSPDir: ./crypto-config/peerOrganizations/go.itcast.com/msp
        AnchorPeers:		# 组织的访问域名和端口
            - Host: peer0.go.itcast.com
              Port: 7051
    - &OrgJava
        Name: OrgJavaMSP
        ID: OrgJavaMSP
        MSPDir: ./crypto-config/peerOrganizations/java.itcast.com/msp
        AnchorPeers:
            - Host: peer0.java.itcast.com
              Port: 7051
              
# orderer节点的配置信息
Orderer: &OrdererDefaults
    # orderer节点共识算法，有效值："solo" 和 "kafka"
    OrdererType: solo
    Addresses:
        - ubuntu.itcast.com:7050	# orderer节点监听的地址
    BatchTimeout: 2s
    BatchSize:
        MaxMessageCount: 10
        AbsoluteMaxBytes: 99 MB
        PreferredMaxBytes: 512 KB
	# kafka相关配置
    Kafka:
        Brokers:
            - 127.0.0.1:9092
    Organizations:
    
Application: &ApplicationDefaults
    Organizations:
```

##  3. peer chaincode 命令

> peer channel的子命令可以通过 `peer channel --help`进行查看. 这里介绍一个这些子命令可以共用的一些参数:
>
> - `--cafile `:  当前orderer节点pem格式的tls证书文件, <font color="red">要使用绝对路径</font>.
>
>   `crypto-config/ordererOrganizations/itcast.com/orderers/ubuntu.itcast.com/msp/tlscacerts/tlsca.itcast.com-cert.pem`
>
> - `-o, --orderer`: orderer节点的地址
>
> - `--tls`: 通信时是否使用tls加密

- **create** - 创建通道

  >命令: `peer channel create [flags]`, 可用参数为:
  >
  >- ` -c, --channelID`: 要创建的通道的ID, 必须小写, 在250个字符以内
  >- `-f, --file`: 由configtxgen 生成的通道文件, 用于提交给orderer
  >- `-t, --timeout`: 创建通道的超时时长

  ```shell
  $ peer channel create -o ubuntu.itcast.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA
  ```

- **join** - 将peer加入到通道中

  > 命令: `peer channel join[flags]`, 可用参数为:
  >
  > - `-b, --blockpath`: genesis创始块文件

  ```shell
  $ peer channel join -b mychannel.block
  ```

- **list** - 列出peer加入的通道

  ```shell
  $ peer channel list
  ```

- **update** - 更新

  > 命令: `peer channel update [flags]`, 可用参数为:
  >
  > - ` -c, --channelID`: 要创建的通道的ID, 必须小写, 在250个字符以内
  > - `-f, --file`: 由configtxgen 生成的组织锚节点文件, 用于提交给orderer

  ```shell
  $ peer channel update -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/Org1MSPanchors.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
  ```

##  4. peer 默认监听端口

下面是Hyperledger中相关监听的服务端口（默认）

- 7050: REST 服务端口
- 7051：peer gRPC 服务监听端口
- 7052：peer CLI 端口
- 7053：peer 事件服务端口
- 7054：eCAP
- 7055：eCAA
- 7056：tCAP
- 7057：tCAA
- 7058：tlsCAP
- 7059：tlsCAA

##  5. 搭建自己的网络

###  5.1 生成fabric证书

> 编写生成组织节点证书的配置文件,我们将其命名为: crypto-config.yaml。在里边指定组织的名称, 节点个数, 用户个数和访问节点的域名信息。

```yaml
# crypto-config.yaml
OrdererOrgs: 				# 排序组织
  - Name: Orderer  			# 排序节点组织名称
    Domain: itcast.com  	# 排序节点组织的根域名
    Specs: 
      - Hostname: ubuntu   # 排序节点主机名, 访问该排序节点的域名: ubuntu.itcast.com
                            # “Spec”的作用是不受下面的Template模板生成规则影响，个性化指定一个域名

PeerOrgs:
  - Name: OrgGo    			  # 组织Org1名称
    Domain: orggo.itcast.com  # 组织Org1的域名
    EnableNodeOUs: true  	  # Node.js支持，java的sdk里面默认是注释掉的
   
    Template:  	# 根据上面注释的模板生成2套公私钥+证书，默认生成规则是peer0-9.组织的域名
      Count: 2  # 即生成peer0.orggo.itcast.com、peer1.orggo.itcast.com 两个节点的公私钥和证     
    Users:  	# 除了admin用户，额外生成一个普通用户
      Count: 3	# 普通用户个数为3
  - Name: OrgCPP
    Domain: orgcpp.itcast.com
    EnableNodeOUs: true
    Template:
      Count: 2
    Users:
      Count: 4
```

```shell
# 配置文件编写完成, 在Demo目录下执行该命令:
$ cryptogen generate --config=./crypto-config.yaml
orggo.itcast.com
orgcpp.itcast.com
# 查看Demo目录中的文件
$ tree -L 6
```

### 5.2 创始块和通道文件的创建

> 配置文件名字为`configtx.yaml

```yaml
# configtx.yaml
---
################################################################################
#
#   Section: Organizations
#
#   - This section defines the different organizational identities which will
#   be referenced later in the configuration.
#
################################################################################
Organizations:
    - &OrdererOrg
        Name: OrdererOrg
        ID: OrdererMSP
        MSPDir: ./crypto-config/ordererOrganizations/itcast.com/msp

    - &OrgGo
        Name: OrgGoMSP
        ID: OrgGoMSP
        MSPDir: ./crypto-config/peerOrganizations/orggo.itcast.com/msp
        AnchorPeers:
            - Host: peer0.orggo.itcast.com
              Port: 7051

    - &OrgCPP
        Name: OrgCppMSP
        ID: OrgCppMSP
        MSPDir: ./crypto-config/peerOrganizations/orgcpp.itcast.com/msp
        AnchorPeers:
            - Host: peer0.orgcpp.itcast.com
              Port: 7051

################################################################################
#
#   SECTION: Capabilities
#
#   - This section defines the capabilities of fabric network. This is a new
#   concept as of v1.1.0 and should not be utilized in mixed networks with
#   v1.0.x peers and orderers.  
#
################################################################################
Capabilities:
    Global: &ChannelCapabilities
        V1_1: true

    Orderer: &OrdererCapabilities
        V1_1: true

    Application: &ApplicationCapabilities
        V1_2: true

################################################################################
#
#   SECTION: Application
#
#   - This section defines the values to encode into a config transaction or
#   genesis block for application related parameters
#
################################################################################
Application: &ApplicationDefaults
    Organizations:

################################################################################
#
#   SECTION: Orderer
#
#   - This section defines the values to encode into a config transaction or
#   genesis block for orderer related parameters
#
################################################################################
Orderer: &OrdererDefaults
    # Available types are "solo" and "kafka"
    OrdererType: solo
    Addresses:
        - ubuntu.itcast.com:7050
    # Batch Timeout: The amount of time to wait before creating a batch
    BatchTimeout: 2s
    # Batch Size: Controls the number of messages batched into a block
    BatchSize:
        # Max Message Count: The maximum number of messages to permit in a batch
        MaxMessageCount: 10
        AbsoluteMaxBytes: 99 MB
        PreferredMaxBytes: 512 KB

    Kafka:
        # Brokers: A list of Kafka brokers to which the orderer connects
        # NOTE: Use IP:port notation
        Brokers:
            - 127.0.0.1:9092
    Organizations:

################################################################################
#
#   Profile
#
#   - Different configuration profiles may be encoded here to be specified
#   as parameters to the configtxgen tool
#
################################################################################
Profiles:

    ItcastOrdererGenesis:
        Capabilities:
            <<: *ChannelCapabilities
        Orderer:
            <<: *OrdererDefaults
            Organizations:
                - *OrdererOrg
            Capabilities:
                <<: *OrdererCapabilities
        Consortiums:
            SampleConsortium:
                Organizations:
                    - *OrgGo
                    - *OrgCPP
    TwoOrgsChannel:
        Consortium: SampleConsortium
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *OrgGo
                - *OrgCPP
            Capabilities:
                <<: *ApplicationCapabilities
```

```shell
创建新目录 channel-artifacts, 将生成的创始块和通道文件存储在该目录中
$ mkdir channel-artifacts
# 生成创始块文件
$ configtxgen -profile ItcastOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
```

> 通道（频道）在Fabric中称为channel， 一个channel表示一个账本。fabric和其他区块链平台最大的区别是支持多账本。每个fabric中至少包含一个channel，下边我们将通过命令将通道创建出来。

> 通道（频道）在Fabric中称为channel， 一个channel表示一个账本。fabric和其他区块链平台最大的区别是支持多账本。每个fabric中至少包含一个channel，下边我们将通过命令将通道创建出来。

```shell
# 创建channel
# channel.tx中包含了用于生产channel的信息
$ configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID mychannel
# 生成相关的锚点文件 - 组织1: Go
$ configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/OrgGoMSPanchors.tx -channelID mychannel -asOrg OrgGoMSP
# 生成相关的锚点文件 - 组织2: CPP
$ configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/OrgCppMSPanchors.tx -channelID mychannel -asOrg OrgCppMSP
# 查看channel-artifacts目录下生成的文件
$ tree channel-artifacts/
channel-artifacts/
├── channel.tx				# 通道文件
├── genesis.block			# 创始块文件
├── orgCppMSPanchors.tx		# Cpp组织锚点文件
└── orgGoMSPanchors.tx		# Go组织锚点文件
```

### 5.3 启动docker-compose

> 编写docker-compose启动时加载的配置文件

```yaml
# docker-compose.yml
# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
# compose 版本号
version: '2'
# 数据卷映射, 本地 -> docker镜像
volumes:
  ubuntu.itcast.com:
  peer0.orggo.itcast.com:
  peer1.orggo.itcast.com:
  peer0.orgcpp.itcast.com:
  peer1.orgcpp.itcast.com:

networks: # 指定容器运行的网络, 同一网络中的容器才能相互通信
  byfn:

services:
  ubuntu.itcast.com:  # 定义的第1个服务名
    extends:          # 继承自当前yaml文件或者其它文件中定义的服务
      file:   base/docker-compose-base.yaml
      # 要继承上述file文件中对应的名字叫做 ubuntu.itcast.com 的服务
      service: ubuntu.itcast.com
    container_name: ubuntu.itcast.com  # 容器名称, 可以自定义
    networks: # 指定容器启动后运行的网络名
      - byfn

  peer0.orggo.itcast.com: # 定义的第2个服务名
    container_name: peer0.orggo.itcast.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer0.orggo.itcast.com
    networks:
      - byfn

  peer1.orggo.itcast.com: # 定义的第3个服务名
    container_name: peer1.orggo.itcast.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer1.orggo.itcast.com
    networks:
      - byfn

  peer0.orgcpp.itcast.com:  # 定义的第4个服务名
    container_name: peer0.orgcpp.itcast.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer0.orgcpp.itcast.com
    networks:
      - byfn

  peer1.orgcpp.itcast.com:  # 定义的第5个服务名
    container_name: peer1.orgcpp.itcast.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer1.orgcpp.itcast.com
    networks:
      - byfn

  cli:   # 定义的第6个服务名
    container_name: cli
    image: hyperledger/fabric-tools:latest  # 指定服务的镜像名称或镜像 ID
    tty: true
    stdin_open: true
    environment:  # 环境变量设置
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_LOGGING_LEVEL=DEBUG
        #- CORE_LOGGING_LEVEL=INFO
      - CORE_PEER_ID=cli
      - CORE_PEER_ADDRESS=peer0.orggo.itcast.com:7051
      - CORE_PEER_LOCALMSPID=OrgGoMSP
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orggo.itcast.com/peers/peer0.orggo.itcast.com/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orggo.itcast.com/peers/peer0.orggo.itcast.com/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orggo.itcast.com/peers/peer0.orggo.itcast.com/tls/ca.crt
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orggo.itcast.com/users/Admin@orggo.itcast.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer   # 工作目录
    command: /bin/bash  # 容器启动后执行的命令
    volumes:  # 本地数据卷内容挂载到容器, 挂载到容器中的数据是只读的
        - /var/run/:/host/var/run/
        - ./chaincode/:/opt/gopath/src/github.com/chaincode
        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
        - ./scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/
        - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
    depends_on: # 指定了容器的启动顺序, 下边5个服务全部启动之后, 启动cli服务
      - ubuntu.itcast.com         # 定义的服务器名1
      - peer0.orggo.itcast.com    # 定义的服务器名2
      - peer1.orggo.itcast.com    # 定义的服务器名3
      - peer0.orgcpp.itcast.com   # 定义的服务器名4
      - peer1.orgcpp.itcast.com   # 定义的服务器名5
    networks:
      - byfn
```

##  6.智能合约

- **init方法**

  ```go
  func (t *TestStudy) Init(stub shim.ChaincodeStubInterface) pb.Response {
      // 获取客户端传入的参数, args是一个字符串, 存储传入的字符串参数
      _, args := stub.GetFunctionAndParameters()
      return shim.Success([]byte("sucess init!!!"))
  };
  ```

- **Invoke方法**

  > Invoke方法的主要作用是写入数据，比如发起交易等。在执行命令`peer chaincode invoke`的时候系统会调用该方法，同时会把命令中`-c`后面的参数传入`Invoke`方法中，以下面的Invoke命令为例:

  ```shell
  $ peer chaincode invoke -o 192.168.1.100:7050 -C mychanne -n mytestcc -c '{"Args": ["invoke"，"a"，"b"，"10"]}'
  ```

  > 上面的命令调用Chaincode的Invoke方法并且传入三个参数`“a”`、`"b”`、`“10”`。注意Args后面数组中的第一个值`“invoke”`是默认的固定参数。

  ```go
  func (t *TestStudy) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
      // 进行交易操作的源代码, 调用ChaincodeStubInterface接口中的方法
      // stub.xxx()
      // stub.yyy()
      return shim.Success([]byte("sucess invoke!!!"))
  };
  ```

- **ChaincodeStubInterface接口中的核心方法**

  > 在shim包中有一个接口ChaincodeStubInterface，该接口提供了一组方法，通过这组方法可以非常方便的操作Fabric中账本数据。ChaincodeStubInterface接口的核心方法大概可以分为四大类：系统管理、存储管理、交易管理、调动外部chaincode。

  - 系统管理相关的方法

    ```go
    // 赋值接收调用chaincode的客户端传递过来的参数
    func GetFunctionAndParameters() (function string, params []string);
    // 示例代码
    func (t *TestStudy) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
        // 获取客户端传入的参数, args是一个字符串, 存储传入的字符串参数
        _, args := stub.GetFunctionAndParameters()
       	var a_param = args[0]
        var b_param = args[1]
        var c_param = args[2]
        return shim.Success([]byte("sucess init!!!"))
    };
    ```

  - 存储管理相关的方法

    - PutState

      ```go
      // 把客户端传递过来的数据保存到Fabric中, 数据格式为键值对
      func PutState(key string, value []byte) error;
      // 示例代码
      func (t *TestStudy) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
          // 数据写入
          stub.PutState("user1", []byte("putvalue"))
          return shim.Success([]byte("sucess invoke user1"))
      };
      ```

    - GetState

      ```go
      // 从Fabric中取出数据, 然后把这些数据交给chaincode处理.
      func GetState(key string) ([]byte, error);
      // 示例代码
      func (t *TestStudy) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
          // 读数据
          keyvalue, err := stub.GetState("user1") 
          return shim.Success(keyvalue)
      };
      ```

    - GetStateByRange

      ```go
      // 根据key的访问查询相关数据
      func GetStateByRange(startKey,endKey string)(StateQueryIteratorInterface, error);
      // 示例代码
      func (t *TestStudy) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
          startKey := "startkey"
          endKey := "endkey"
          // 根据范围查询, 得到StateQueryIteratorInterface迭代器接口
          keysIter, err := stub.getStateByRange(startKey, endKey)
          // 最后关闭迭代器接口
          defer keysIter.Close()
          var keys []string
          for keysIter.HasNext() {	// 如果有下一个节点
              // 得到下一个键值对
              response, iterErr := keysIter.Next()
              if iterErr != nil {
                  return shim.Error(fmt.Sprintf("find an error %s", iterErr))
              }
              keys = append(keys, response.Key)	// 存储键值到数组中
          }
          // 遍历keys数组
          for key, value := range keys {
              fmt.Printf("key %d contains %s\n", key, value)
          }
          // 编码keys数组成json格式
          jsonKeys, err := json.Marshal(keys)
          if err := nil {
              return shim.Error(fmt.Sprintf("data Marshal json error: %s", err))
          }
          // 将编码之后的json字符串传递给客户端
          return shim.Success(jsonKeys)
      };
      ```

    - GetHistoryForKey

      ```go
      // 查询某个键的历史记录
      func GetHistoryForKey(key string) (HistoryQueryIteratorInterface, error);
      // 示例代码
      func (t *TestStudy) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
          keysIter, err := stub.GetHistoryForKey("user1")	// user1是一个假设的键值
          if err := nil {
              return shim.Error(fmt.Sprintf("GetHistoryForKey error: %s", err))
          }
          defer keysIter.Close()
          var keys []string
          for keysIter.HasNext() {	// 遍历集合,如果有下一个节点
              // 得到下一个键值对
              response, iterErr := keysIter.Next()
              if iterErr != nil {
                  return shim.Error(fmt.Sprintf("find an error %s", iterErr))
              }
              // 交易编号
              txid := response.TxId
              // 交易的值
              txvalue := response.Value
              // 当前交易的状态
              txStatus := response.IsDelete
              // 交易发生的时间戳
              txtimestamp := response.Timestamp
              // 计算从1970.1.1到时间戳的秒数
              tm := time.Unix(txtimestamp.Seconds, 0)
              // 根据指定的格式将日期格式化
              datestr := tm.Format("2018-11-11 11:11:11 AM")
              fmt.Printf("info - txid:%s, value:%s, isDel:%t, dateTime:%s\n", txid, string(txvalue), txStatus, datestr)
              keys = append(keys, txid)
          }
          // 将数组中历史信息的key编码为json格式
          jsonKeys, err := json.Marshal(keys)
          if err := nil {
              return shim.Error(fmt.Sprintf("data Marshal json error: %s", err))
          }
          // 将编码之后的json字符串传递给客户端
          return shim.Success(jsonKeys)
      }
      ```

    - DelState

      ```go
      // 删除一个key
      func DelState(key string) error;
      // 示例代码
      func (t *TestStudy) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
          err := stub.DelState("delKey")
          if err != nil {
          	return shim.Error("delete key error !!!")
          }
          return shim.Success("delete key Success !!!")
      }
      ```

    - CreateCompositeKey

      ```go
      // 给定一组属性，将这些属性组合起来构造一个复合键
      func CreateCompositeKey(objectType string, attributes []string) (string, error);
      // 示例代码
      func (t *TestStudy) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
          parms := []string("go1", "go2", "go3", "go4", "go5", "go6")
          ckey, _ := stub.CreateCompositeKey("testkey", parms)
          // 复合键存储到账本中
          err := stub.putState(ckey, []byte("hello, go"))
          if err != nil {
              fmt.Println("find errors %s", err)
          }
          // print value: testkeygo1go2go3go4go5go6
          fmt.Println(ckey)
          return shim.Success([]byte(ckey))
      }
      ```

    - GetStateByPartialCompositeKey / SplitCompositeKey

      ```go
      // 根据局部的复合键返回所有的匹配的键值
      func GetStateByPartialCompositeKey(objectType string, keys []string)(StateQueryIteratorInterface, error);
      // 给定一个复合键，将其拆分为复合键所有的属性
      func SplitCompositeKey(compositeKey string) (string, []string, error)
      // 示例代码
      func (t *TestStudy) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
          searchparm := []string{"go1"}
          rs, err := stub.GetStateByPartialCompositeKey("testkey", searchparm)
          if err != nil {
              error_str := fmt.Sprintf("find error %s", err)
              return shim.Error(error_str)
          }
          defer rs.Close()
          var tlist []string
          for rs.HasNext() {
              responseRange, err := rs.Next()
              if err != nil {
                  error_str := fmt.Sprintf("find error %s", err)
                  fmt.Println(error_str)
                  return shim.Error(error_str)
              }
              value1,compositeKeyParts,_ := stub.SplitCompositeKey(responseRange, key)
              value2 := compositeKeyParts[0]
              value3 := compositeKeyParts[1]
              // print: find value v1:testkey, v2:go1, v3:go2
              fmt.Printf("find value v1:%s, v2:%s, V3:%s\n", value1, value2, value3)
          }
          return shim.Success("success")
      }
      ```

  - 交易管理相关的方法

    ```go
    // 获取当前客户端发送的交易时间戳
    func GetTxTimestamp() (*timestamp.Timestamp, error);
    // 示例代码
    func (t *TestStudy) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
        txtime, err := stub.GetTxTimestamp()
        if err != nil {
            fmt.printf("Error getting transaction timestamp: %s", error)
            return shim.Error(fmt.Sprintf("get transaction timestamp error: %s", error))
        }
        tm := time.Unix(txtime.Second, 0)
        return shim.Success([]byte(fmt.Sprint("time is: %s", tm.Format("2018-11-11 23:23:32"))))
    }
    ```

  - 调用其他chaincode的方法

    ```go
    // 调用另一个链码中的Invoke方法
    func InvokeChaincode(chaincodeName string,args [][]byte,channel string) pb.Response
    // 示例代码
    func (t *TestStudy) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
        // 设置参数, a向b转转11
        trans:=[][]byte{[]byte("invoke"),[]byte("a"),[]byte("b"),[]byte("11")}
        // 调用chaincode
    	response := stub.InvokeChaincode("mycc", trans, "mychannel")
        // 判断是否操作成功了
        // 课查询: https://godoc.org/github.com/hyperledger/fabric/protos/peer#Response
        if response.Status != shim.OK {
            errStr := fmt.Sprintf("Invoke failed, error: %s", response.Payload)
            return shim.Error(errStr)
        }
        return shim.Success([]byte("转账成功..."))
    }
    
    // ==================================================
    // 获取客户端发送的交易编号
    func GetTxID() string
    // 示例代码
    func (t *TestStudy) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
        txid := stub.GetTxID()
        return shim.Success([]byte(txid))
    }
    ```

## 7 ChainCode交易的背书（endorse）

> 区块链是一个去中心的，所有参与方集体维护的公共账本。Fabric作为一个典型的区块链的技平台当然也具备这样的特点。Fabric中对数据参与方对数据的确认是通过Chaincode来进行的。
>
> 在Fabric中有一个非常重要的概念称为Endorsement，中文名为背书。背书的过程是一笔交易被确认的过程。而背书策略被用来指示对相关的参与方如何对交易讲行确认。当一个节点接收到一个交易请求的时候，会调用VSCC（系统Chaincode，专门负责处理背书相关的操作）与交易的chaincode共同来验证交易的合法性。在VSCC和交易的Chaincode共同对交易的确认中， 通常会做以下的校验。
>
> - 所有的背书是否有效（参与的背书的签名是否有效）。
>
> - 参与背书的数量是否满足要求。
>
> - 所有背书参与方是否满足要求。
>
> 背书政策是指定第二和第三点的一种方式。这些概念看起来还是比较难懂的，理解它们最好的办法是通过一个具体的实例。背书策略的设置是通过部署时`instantiate`命令中`-P`-参数来设置的。命令样式如下：

```shell
$ peer chaincode instantiate -o oraderer.test.com:7050 -C mychannel -n mycc —v 1.0 -c '{"Args":["init", "a", "100", "b", "200"]}' -P "AND ('Org1MSP.member', 'Org2MSP.member')" 
# 上述命令是对Chaincode进行实例化的操作，我们提取-P后面的参数：
"AND ('Org1MSP.member', 'Org2MSP.member')" 
```

> 这个参数包说明的是当前Chaincode发起的交易，需要组织编号为`Org1MSP`和组织编号为`Org2MSP`的组织中的任何一个用户共同参与交易的确认并且同意，这样交易才能生效并被记录到区块链中。通过上述背书策略的实例我们可以知道背书策略是通过一定的关键字和系统的属性组成的。根据Fabric的系统定义，可以将上面的背书策略拆解如下：
>
> `AND`参与背书者之间的关系，`AND`表示<font color="red">所有参与方共同对交易进行确认</font>。除了AND之外还可以使用关键字OR,如果使用关键字`OR`表示<font color="red">参与方的任何一方参与背书即完成交易的确认</font>。
>
> Org1MSP.member表示参与背书的组和组织中参与背书的用户。Org1MSP表示组织的编号，这个值是怎么来的呢？之前我们介绍过`cryptogen`模块，该模块根据配置文件生成系统的配置和账号信息。在`cryptogen`的配置文件中有一个节点`Organizations->ID`，该节点的值就是该组织的编号，也是在配置背书策略时需要用到的组织的编号。`member`泛指组织内的任何一个用户，当然也可以是组织某个具体的用户。
>
> 通过上面的描述，基本上可以了解背书策略的编写规则，下面通过几个实例进一步了解背书策略的编写规则。

- **背书规则示例1**

  ```shell
  # 按照该背书规则进行交易, 必须通过组织Org1MSP，Org2MSP，Org3MSP中的用户共同验证交易才能生效
  "AND ('Org1MSP.member', 'Org2MSP.member', 'Org3MSP.member')" 
  ```

- **背书规则示例2**

  ```shell
  # 按照该背书规则进行交易,只需要通过组织 Org1MSP 或 Org2MSP 或 Org3MSP 中的任何一个成员验证，即可生效
  "OR ('Org1MSP.member', 'Org2MSP.member', 'Org3MSP.member')" 
  ```

- **背书规则示例3**

  ```shell
  # 按照该背书规则进行交易,有两种办法让交易生效
  # 	1. 组织Org1MSP中的某个成员对交易进行验证。
  #	2. 组织Org2MSP和组织Org3MSP中的成员共同就交易进行验证。
  "OR ('Org1MSP.member', AND（'Org2MSP.member', 'Org3MSP.member'）)" 
  ```

> 以上介绍了背书的规则，有一点需要注意：<font color="red">背书规则只针对chaincode中写入数据的操作进行校验，对于查询类操作不背书。</font>以golang版本的chaincode为例， 需要利用背书规则对操作进行校验的方法如下：

```go
PutState(key string, value []byte) error;
DelState(key string) error;
```

> <font color="red">Fabric中的背书是发生在客户端的，需要进行相关的代码的编写才能完成整个背书的操作。</font>













