# FileProof API

FileProof是Meeda DA层的合约实现。

## 1. addFile

`function addFile(bytes32[4] memory commitment, uint64 sizeByte, uint256 start, uint256 end, bytes memory credential)`

**功能：**

账户将文件commitment和其他元数据信息添加到合约中，并预付文件可用性服务费用。

**注意：**

调用该方法前，账户需要先`approve`FileProof合约账户相应的费用，费用价格为：sizeByte*(end-start)*price。

**参数：**

commitment- 文件唯一标识符，是文件的kzg承诺

sizeByte - 文件大小，单位Byte

start - 文件上传时间

end - 文件存储结束时间

credential - 文件上传凭据，表明文件确实已上传到da-backend

**事件：**

`event AddFile(address indexed account, bytes32[4] indexed commitment, uint256 start, uint256 end, uint64 size, uint64 price);`

`event Transfer(address indexed from, address indexed to, uint256 value);`

## 2. genRnd

`function genRnd()`

**功能：**

产生一个伪随机数用于提交证明以及验证。

**注意：**

只有在正确的时间调用才能产生有效的伪随机数。当不是最后一个挑战期（`finalExpire > last+interval+period`）时，正确的时间处于<`last+interval`, `last+interval+period`>之间；当处于最后一个挑战期（`finalExpire <= last+interval+period`）时，正确的时间处于<`finalExpire`, `finalExpire+period`>之间。

**事件：**

`event GenRnd(bytes32 rnd);`

`event NoProofs(uint256 oldLast, uint256 newLast, uint256 missedProfit);`

## 3. submitProof

`function submitProof(bytes32 _rnd, bytes32[4] memory _Cn, ProofInfo memory _Pn)`

**功能：**

submitter提交聚合证明，并且进行第一阶段的验证，验证`Pn`的正确性。

**注意：**

每个时期（<`last`, `last+interval+period`>期间称为一个时期）只能在产生有效随机数之后提交一次证明，即使该证明错误也无法再次提交去弥补。

无论本次证明验证是否成功，都会释放上一时期的收益。

若本次证明验证成功，则暂定本次收益为合约账户剩余余额对总时间的线性释放（即`thisProfit = balance * (interval+period) / (finalExpire - last);`），之所以说是暂定，是因为可能有节点会发起挑战，从而开启第二阶段验证，在第二阶段验证中，可能会发现da-backend存在欺诈行为。若存在欺诈行为，则本次收益为0.

**参数：**

_rnd - 调用genRnd()后产生的有效随机数

_Cn - 待验证文件的聚合承诺

_Pn - 待验证文件的聚合kzg证明

可用性证明proof结构体定义如下：

```sol
struct ProofInfo {

    bytes32[4] npsi; // G1, (x,y), x:bytes48(represented by bytes32[2]), y:bytes48

    bytes32 y; // Fr

}
```

**事件：**

`event SubmitProof(bytes32 indexed rnd, bytes32[4] cn, ProofInfo pn, bool res);`

`event ProfitRelease(uint256 releasedProfit, uint256 lockedProfit, uint256 missedProfit);`

`event Transfer(address indexed from, address indexed to, uint256 value);`

## 4. doChallenge

`function doChallenge(uint8 _chalIndex)`

**功能：**

任意账户发起挑战，开启第二阶段验证，验证`Cn`的正确性。

**参数：**

_chaIndex - 指定被等分后的多个聚合承诺值的索引`Cn_i`，第一轮挑战`_chalIndex`为0

**注意：**

发起挑战的时候，挑战者需要进行质押，从而防止恶意挑战行为。若挑战成功，质押将被返回；若挑战失败，质押将被扣除。

发起挑战前，挑战者需要先`approve`FileProof合约账户质押费用（SettingInfo.chalPledge）。

**事件：**

`event CreateChal(bytes32 indexed rnd, bytes32[4] cn, uint256 chalPledge, address challenger)`

`event Transfer(address indexed from, address indexed to, uint256 value);`

## 5. responseChal

`function responseChal(bytes32[4][10] memory _cns)`

**功能：**

submitter响应挑战

**参数：**

_cns - 将被挑战的`Cn`等分

**注意：**

响应挑战失败（也就是_cns聚合不能得到正确的`Cn`）的情况下，将触发事件。

**事件：**

`event Fraud(bytes32 indexed rnd, address indexed submitter, address indexed challenger, uint256 fine, uint256 reward);`

`event Transfer(address indexed from, address indexed to, uint256 value);`

## 6. oneStepProve

`function oneStepProve(bytes32[4][] memory _commitments)`

**功能：**

submitter提交最后一轮的单步证明，从而响应最后一轮的挑战

**参数：**

_commitments- 被挑战的`Cn`的多个文件承诺值

**注意：**

响应挑战失败（也就是_commitments聚合不能得到正确的`Cn`）的情况下，将触发`Fraud`事件；否则，将触发`NoFraud`事件。

**事件：**

`event Fraud(bytes32 indexed rnd, address indexed submitter, address indexed challenger, uint256 fine, uint256 reward);`

`event NoFraud(bytes32 indexed rnd, address indexed submitter, address indexed challenger, uint256 compensation);`

`event Transfer(address indexed from, address indexed to, uint256 value);`

## 7. endChallenge

`function endChallenge()`

**功能：**

当submitter没有及时响应挑战，或者挑战者没有及时发起下一轮的挑战，任何账户都可以调用该方法，从而结束挑战

**注意：**

submitter没有及时响应挑战的情况下，将触发`Fraud`事件；否则，将触发`NoFraud`事件。

**事件：**

`event Fraud(bytes32 indexed rnd, address indexed submitter, address indexed challenger, uint256 fine, uint256 reward);`

`event NoFraud(bytes32 indexed rnd, address indexed submitter, address indexed challenger, uint256 compensation);`

`event Transfer(address indexed from, address indexed to, uint256 value);`

## 8. withdrawMissedProfit

`function withdrawMissedProfit()`

**功能：**

任何账户可以调用该方法，从而将错失收益转给基金会账户。错失收益即submitter未按时响应挑战，或者在第一阶段提交证明的时候就失败的情况下，错失的收益。

**事件：**

`event Transfer(address indexed from, address indexed to, uint256 value);`

## 9. alterSetting

`function alterSetting(IFileProof.SettingInfo memory si, bytes[5] memory signs)`

**功能：**

社区成员根据实际情况，获取多签，从而修改FileProof基本配置

**参数：**

si - 新的配置信息

signs - 社区成员的多签信息

`SettingInfo`结构体定义如下：

```sol
struct SettingInfo {
        uint32 interval; // the interval of submitProof, unit: s
        uint32 period; // the period of submitProof, unit: s
        uint32 chalSum;
        uint32 respondTime; // effective respond time for each step in the multi-round interaction process
        // size price(attomemo/byte/second), used to calculate the addFile fee based on the file size
        uint64 price;
        address submitter; // who submit proofs to get rewards
        address receiver; // submitter specifies the receiver receives rewards
        address foundation; // receive the missedProfit
        uint8 chalRewardRatio; // the ratio of pendingProfit to be rewarded to challenger when submitter is indeed fraudulent
        uint256 chalPledge; // the amount that should be staked when create challenge
        bytes32[8] vk; // verifyKey, s * g_2, mpc -> s; G2, x:bytes96(two Fp elements, represented by bytes32[4]), y:bytes96
    }
```

**事件：**

`event AlterSetting(SettingInfo si);`

## 10. selectFiles

`function selectFiles(uint256 i) external view returns (bytes32[4] memory commitment)`

**功能：**

每个时期产生有效随机数后，合约会根据随机数挑选一批待证明的文件，输入这批文件的索引，从而返回对应的文件承诺值。

**注意：**

被挑选文件的总数由`SettingInfo.chalSum`决定。

## 11. getCommit

`function getCommit(uint256 i) external view returns (uint256 sum, bytes32[4] memory commitment)`

**功能：**

根据序号，获取合约中上传文件的总数和对应的承诺值。

## 12. getFileInfo

`function getFileInfo(bytes memory commit) external view returns (uint64 index, uint256 expiration)`

**功能：**

根据文件承诺值，获取文件的序号以及到期时间。

## 13. getProofInfo

`function getProofInfo() external view returns (bytes32 y)`

**功能：**

获取当前时期的证明信息。

**注意：**

可通过监控event获取历史证明信息，以及当前完整的证明信息。

## 14. getVerifyInfo

`function getVerifyInfo() external view returns (bytes32 rnd, bool lock, uint256 last)`

**功能：**

获取当前时期的验证信息。

**注意：**

可通过监控event获取历史验证信息，以及当前完整的验证信息。

## 15. getProfitInfo

`function getProfitInfo() external view returns (uint256 pendingProfit, uint256 missedProfit, uint256 finalExpire)`

**功能：**

获取当前时期的收益信息。

**注意：**

可通过监控event获取历史收益信息。

## 16. getChallengeInfo

`function getChallengeInfo() external view returns (uint8 chalStatus, address challenger, uint8 chalIndex, uint256 startIndex, uint256 chalLength)`

**功能：**

获取当前时期的挑战信息。

**注意：**

可通过监控event获取历史挑战信息，以及当前完整的挑战信息。

## 17. getSettingInfo

`function getSettingInfo() external view returns (uint32 interval, uint32 period, uint32 chalSum, uint32 respondTime, uint64 price, address submitter, address receiver, address foundation, uint8 chalRewardRatio, uint256 chalPledge)`

**功能：**

获取当前的配置信息。

**注意：**

可通过监控event获取历史配置信息。

## 18. getVK

`function getVK() external view returns (bytes32[8] memory vk)`

**功能：**

获取当前的验证公钥。

**注意：**

可通过监控event获取历史验证公钥。
