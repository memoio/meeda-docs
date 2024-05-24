# LightNode HTTP API

You can follow the steps in [Light Node Deployment Document](../node/How-to-build-LightNode.md) to start Meeda's light node. Light nodes provide some http API interfaces that ordinary users need to access the Meeda network. The default listening port of this service is 8082, that is, the baseURL is https://localhost:8082; all the following request URLs should be changed according to the actual situation.

## 1. Upload Data

Request Url: https://localhost:8082/putObject

Request Method: POST

Request Header Information:

| Param       | Value             |
| ------------ | ---------------- |
| Content-Type | application/json |

Request Params(Json):

| Param | Value        | Type | Required | Description                               |
| ------ | ------------ | ---------------- | ---- | ---------------------------------- |
| data   | data to be uploaded | String   | yes  | data to be uploaded represented using hexadecimal encoding |

Return Params:

| Param | Value   | Type | Required | Description           |
| ------ | ------ | ---------------- | ---- | -------------- |
| id     | file id | string           | yes   | file unique identifier |

Return Example:

```json
{"id":"0x1010674e8b5d7167e335fb4f1237fa11857b8f890c0b2051b303e4c9120fc6732962c67309653bf3c80cba3207ddf4d808735fbf50c706b68761ec94b5ef9c00564503621f21eb8513fd2c7a6a2fa5fb548872b3bee34961b888124b0caa92a7"}
```

## 2. Download Data

Request Url: https://localhost:8082/getObject?id={id}

Request Method: GET

Request Header Information:

none

Request Params:

none

Request Params(Data):

| Param      | Description         |
| ----------- | ------------ |
| contentType | data type     |
| data        | data binary value |

## 3. Get File Information

Request Url: https://localhost:8082/getObjectInfo?id={id}

Request Method: GET

Request Header Information:

none

Request Params:

none

Request Params(JSON):

| Param     | Description         |
| ---------- | ------------ |
| id         | data unique indentifier   |
| size       | data size     |
| expiration | data expiration time |

## 4. Get Proof Information

Request Url: https://localhost:8082/getProofInfo?id={id}

Request Method: GET

Request Header Information:

none

Request Params:

none

Request Params(JSON):

| Param              | Description                                                  |
| ------------------- | ----------------------------------------------------- |
| id                  | data unique indentifier      |
| selectedNumber      | the number of times data has been selected to participate in KZG certification  |
| provedSuccessNumber | the number of times the storage node uploaded the correct certification when the data was selected to participate in the KZG certification |
