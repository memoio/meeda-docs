# http API说明

可以按照[轻节点部署文档](../node/How-to-build-LightNode.md)中的步骤启动Meeda的轻节点，轻节点提供了普通用户访问Meeda网络所需的一些http API接口。该服务默认监听端口为8082，即baseURL为https://localhost:8082；以下所有请求URL应根据实际情况进行更改。

## 1. 上传数据

请求Url：https://localhost:8082/putObject

请求方式：POST

请求头信息：

| 参数名       | 变量             |
| ------------ | ---------------- |
| Content-Type | application/json |

请求参数：

| 参数名 | 变量         | 类型【长度限制】 | 必填 | 描述                               |
| ------ | ------------ | ---------------- | ---- | ---------------------------------- |
| data   | 待上传的数据 | String           | 是   | 将要上传的数据，使用16进制编码表示 |

返回参数(JSON)：

| 参数名 | 变量   | 类型【长度限制】 | 必填 | 描述           |
| ------ | ------ | ---------------- | ---- | -------------- |
| id     | 文件ID | string           | 是   | 文件唯一标识符 |

返回例子：

```json
{"id":"0x1010674e8b5d7167e335fb4f1237fa11857b8f890c0b2051b303e4c9120fc6732962c67309653bf3c80cba3207ddf4d808735fbf50c706b68761ec94b5ef9c00564503621f21eb8513fd2c7a6a2fa5fb548872b3bee34961b888124b0caa92a7"}
```

## 2. 下载数据

请求Url：https://localhost:8082/getObject?id={id}

请求方式：GET

请求头信息：

无

请求参数：

无

返回参数（Data):

| 参数名      | 描述         |
| ----------- | ------------ |
| contentType | 数据类型     |
| data        | 数据二进制值 |

## 3. 查看文件相关信息

请求Url: https://localhost:8082/getObjectInfo?id={id}

请求方式：GET

请求头信息：

无

请求参数：

无

返回参数（JSON）：

| 参数名     | 描述         |
| ---------- | ------------ |
| id         | 数据唯一ID   |
| size       | 数据大小     |
| expiration | 数据过期时间 |

## 4. 查看证明信息

请求Url: https://localhost:8082/getProofInfo?id={id}

请求方式：GET

请求头信息：

无

请求参数：

无

返回参数（JSON）：

| 参数名              | 描述                                                  |
| ------------------- | ----------------------------------------------------- |
| id                  | 数据唯一ID                                            |
| selectedNumber      | 数据被抽中参与KZG证明的次数                           |
| provedSuccessNumber | 数据被抽中参与KZG证明，且存储节点上传了正确证明的次数 |
