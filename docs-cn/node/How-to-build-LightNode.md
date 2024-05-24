# 部署Meeda轻节点

## 安装

本文档将指引您如何安装Meeda轻节点。

### 通过源码安装

安装轻节点包括以下几个步骤：

1. 先删除所有的本地依赖库以及meeda-node，并再次下载所有的依赖库以及meeda-node。

   ```bash
   cd $HOME
   ### 删除所有的本地依赖库和meeda-node
   rm -rf meeda-node did-solidity go-did memov2-contractsv2 go-mefs-v2 relay memo-go-contracts-v2
   ### 获取依赖包
   git clone http://132.232.87.203:8088/did/did-solidity.git
   git clone http://132.232.87.203:8088/did/go-did.git
   git clone http://132.232.87.203:8088/508dev/memov2-contractsv2.git
   git clone http://132.232.87.203:8088/508dev/go-mefs-v2.git
   git clone http://132.232.87.203:8088/508dev/relay.git
   git clone http://132.232.87.203:8088/508dev/memo-go-contracts-v2.git
   ### 将依赖包切换到最新分支(go-mefs-v2)
   cd go-mefs-v2
   git checkout 2.7.4
   cd ..
   ## 将依赖包切换到最新分支(did-solidity)
   cd did-solidity
   git checkout meeda-v1.2.0
   cd ..
   ## 将依赖包切换到最新分支(go-did)
   cd go-did
   git checkout meeda-v1.1.0
   cd ..
   ### 获取meeda-node
   git clone http://132.232.87.203:8088/middleware/meeda-node.git
   ### 
   cd meeda-node
   ### 切换到最新分支
   git checkout 1.2.0
   ```

2. 编译meeda-node。

   ```bash
   CGO_ENABLED=1 make build
   ```

3. 安装meeda-node。

   ```bash
   make install
   ```

4. 检查meeda-node是否安装成功。

   ```bash
   meeda version
   ```

   该命令会输出当前meeda-node的版本信息，commit信息以及编译日期。

您已经安装完成轻节点，现在您需要部署轻节点从而接入Meeda的数据可用性网络。

## 部署

Meeda轻节点会验证Meeda存储全节点提交的KZG证明，若Meeda轻节点验证KZG证明有误，则Meeda轻节点发起多轮挑战，直到一方胜出。

通过Meeda轻节点，普通用户可访问和接入Meeda数据可用性网络。Meeda轻节点包括以下功能：

- 上传下载数据。
- 同步上传的数据信息，包括数据承诺，数据大小，数据过期时间等。
- 同步KZG聚合证明信息。
- 验证KZG聚合证明信息，如发现证明有误，则发起多轮挑战。

普通用户部署轻节点后，可以从多轮挑战中获取收益。

### 运行Meeda轻节点

启动Meeda轻节点时需要设置`sk`参数。`sk`用于轻节点签署以太坊上的交易。

Meeda轻节点部署完成后，轻节点的rpc接口将绑定到8082端口。

> **部署在product链上**
>
> ```bash
> meeda light run --sk=<private key>
> ```

>**部署在dev链上**
>
>```bash
>meeda light run --chain=dev --sk=<private key>
>```

### 关闭Meeda轻节点

为了正常停止轻节点，请在节点运行的终端中使用`Control + C`命令。
