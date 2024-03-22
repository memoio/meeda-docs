# How to build DABackend

启动DABackend节点是启动Meeda服务中重要的一环。启动DABackend节点可分为以下两个步骤：

- 运行mefs-user节点
- 运行DABackend节点

## 运行mefs user节点

运行mefs user节点可查看以下文档：

https://github.com/memoio/memo-docs/blob/main/docs/start-and-usage/how-to-start-with-Docker/megrez/start-user-in-Docker-Megrez.md

https://github.com/memoio/memo-docs/blob/main/docs/start-and-usage/how-to-start-with-Windows/start-user-in-Windows.md

## 运行DABackend节点

```shell
### 获取代码
git@132.232.87.203:middleware/backend.git
### 编译
cd backend
make
### 运行
./backend daemon run
```

用户在运行DABackend节点时，可以添加以下参数：

- endpoint(别名e)：指定服务监听端口，默认8080；若需更改，可以通过执行`backend daemon run -e=:8081`，修改为8081。

用户在第一次运行backend时会在当前目录创建一个配置文件`config.json`。

默认生成的`config.json`文件如下：

```json
{
        "storage": {
                "mefs": {
                        "api": "/ip4/192.168.1.46/tcp/26812",
                        "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBbGxvdyI6WyJyZWFkIiwid3JpdGUiLCJzaWduIiwiYWRtaW4iXX0.OzYt9dIYMEuUoaEQeXah2wkPUZ1O6Yya7mwuuIAP89s"
                },
                "ipfs": {
                        "host": "127.0.0.1:5002"
                },
                "prices": null,
                "traffic_cost": 0
        },
        "contract": {
                "chain": "dev",
                "caddr": "",
                "saddr": "0xdFF2A42524df7574361A90aac9141DE3f4D8eA02"
        },
        "da": {
                "UserSecurityKey": "xxxxx",
                "SubmitterSecurityKey": "xxxxx"
        },
        "securityKey": "xxxxx",
        "domain": "memo.io",
        "ethDriveUrl": "https://ethdrive.net"
}
```

在DABackend中，我们仅需要维护storage中的mefs的参数配置，以及da中的参数配置即可，其余参数的配置可以完全忽略，对于DABackend的运行并不影响。

DABackend各个参数的详细描述以及配置方法如下：

- `storage`：调用底层存储系统需要的配置信息。
  - `mefs`：底层存储系统mefs的配置信息。
    - `api`：mefs中mefs-user节点的API接口信息。Linux用户在运行mefs-user后，通常会在`~`目录下创建`.mefs-user`目录（包含mefs-user节点的配置信息），在文件`~/.mefs-user/api`中包含有api的详细信息，可以直接复制到`api`参数中。
    - `token`：若其他用户需要访问mefs-user的受限制的API接口，需要添加`token`信息。`token`信息可以从文件`~/.mefs-user/token`文件中读取，可以直接复制到`token`参数中。
- `da`：调用FileProof合约需要的配置信息。
  - `SubmitterSecurityKey`：合约中Submitter对应的密钥，主要用于上传证明并响应其他人的挑战。submitter的详细信息可查看contract下的文档。
  - `UserSecurityKey`：用户密钥（无特别要求，需要对应的地址中有memo币），主要用于上传文件。
