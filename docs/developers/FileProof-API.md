# FileProof API

FileProof is the contract implementation of Meeda DA layer.

## 1. addFile

`function addFile(bytes32[4] memory commitment, uint64 sizeByte, uint256 start, uint256 end, bytes memory credential)`

**Function：**

The account adds file commitment and other metadata information to the contract and prepays for file availability services.

**Notice：**

Before calling this method, the account needs to `approve` the corresponding fee of the FileProof contract account. The fee price is: `sizeByte*(end-start)*price`.

**Parameters：**

commitment- The file unique identifier is the kzg commitment of the file

sizeByte - File size, unit Byte

start - File upload time

end - File storage end time

credential - File upload credentials showing that the file was indeed uploaded to da-backend

**Events：**

`event AddFile(address indexed account, bytes32[4] indexed commitment, uint256 start, uint256 end, uint64 size, uint64 price);`

`event Transfer(address indexed from, address indexed to, uint256 value);`

## 2. genRnd

`function genRnd()`

**Function：**

Generate a pseudo-random number for submission proof and verification.

**Notice：**

Effective pseudo-random numbers can only be generated if called at the right time. When it is not the last challenge period (`finalExpire > last+interval+period`), the correct time is between <`last+interval`, `last+interval+period`>; when it is the last challenge period (`finalExpire <= last+interval+period`), the correct time is between <`finalExpire`, `finalExpire+period`>.

**Event：**

`event GenRnd(bytes32 rnd);`

`event NoProofs(uint256 oldLast, uint256 newLast, uint256 missedProfit);`

## 3. submitProof

`function submitProof(bytes32 _rnd, bytes32[4] memory _Cn, ProofInfo memory _Pn)`

**Function：**

The submitter submits the aggregation proof and performs the first phase of verification to verify the correctness of `Pn`.

**Notice：**

Each period (<`last`, `last+interval+period`> period is called an period) can only submit a proof once after generating a valid random number. Even if the proof is wrong, it cannot be submitted again to make up for it.

Regardless of whether this proof verification is successful or not, the benefits from the previous period will be released.

If this proof verification is successful, then this profit is tentatively determined as the linear release of the remaining balance of the contract account to the total time (i.e. `thisProfit = balance * (interval+period) / (finalExpire - last);`). The reason why it is said to be Tentatively, it is because some nodes may initiate a challenge and start the second phase of verification. In the second phase of verification, da-backend may be found to be fraudulent. If there is fraud, the income this time will be 0.

**Parameters：**

_rnd - A valid random number generated after calling genRnd()

_Cn - Aggregation promise of documents to be verified

_Pn - Aggregated kzg proof of documents to be verified

The availability proof proof structure is defined as follows:

```sol
struct ProofInfo {

    bytes32[4] npsi; // G1, (x,y), x:bytes48(represented by bytes32[2]), y:bytes48

    bytes32 y; // Fr

}
```

**Event：**

`event SubmitProof(bytes32 indexed rnd, bytes32[4] cn, ProofInfo pn, bool res);`

`event ProfitRelease(uint256 releasedProfit, uint256 lockedProfit, uint256 missedProfit);`

`event Transfer(address indexed from, address indexed to, uint256 value);`

## 4. doChallenge

`function doChallenge(uint8 _chalIndex)`

**Function：**

Any account initiates a challenge and starts the second phase of verification to verify the correctness of `Cn`.

**Parameters：**

_chaIndex - Specify the index `Cn_i` of multiple aggregated commitment values after being equally divided. The first round of challenge `_chalIndex` is 0

**Notice：**

When initiating a challenge, the challenger needs to pledge to prevent malicious challenging behavior. If the challenge is successful, the pledge will be returned; if the challenge fails, the pledge will be deducted.

Before initiating a challenge, the challenger needs to `approve` the FileProof contract account pledge fee (SettingInfo.chalPledge).

**Event：**

`event CreateChal(bytes32 indexed rnd, bytes32[4] cn, uint256 chalPledge, address challenger)`

`event Transfer(address indexed from, address indexed to, uint256 value);`

## 5. responseChal

`function responseChal(bytes32[4][10] memory _cns)`

**Function：**

submitter response challenge

**Parameters：**

_cns - Divide the challenged `Cn` into equal parts

**Notice：**

If the response to the challenge fails (that is, the _cns aggregation cannot get the correct `Cn`), the event will be triggered.

**Event：**

`event Fraud(bytes32 indexed rnd, address indexed submitter, address indexed challenger, uint256 fine, uint256 reward);`

`event Transfer(address indexed from, address indexed to, uint256 value);`

## 6. oneStepProve

`function oneStepProve(bytes32[4][] memory _commitments)`

**Function：**

submitter submit the final round of single-step proofs to respond to the final round of challenges.

**Parameters：**

_commitments- Multiple file promise values of `Cn` being challenged

**Notice：**

If the response to the challenge fails (that is, the _commitments aggregation cannot get the correct `Cn`), the `Fraud` event will be triggered; otherwise, the `NoFraud` event will be triggered.

**Event：**

`event Fraud(bytes32 indexed rnd, address indexed submitter, address indexed challenger, uint256 fine, uint256 reward);`

`event NoFraud(bytes32 indexed rnd, address indexed submitter, address indexed challenger, uint256 compensation);`

`event Transfer(address indexed from, address indexed to, uint256 value);`

## 7. endChallenge

`function endChallenge()`

**Function：**

When the submitter does not respond to the challenge in time, or the challenger does not initiate the next round of challenge in time, any account can call this method to end the challenge.

**Notice：**

If the submitter does not respond to the challenge in time, the `Fraud` event will be triggered; otherwise, the `NoFraud` event will be triggered.

**Event：**

`event Fraud(bytes32 indexed rnd, address indexed submitter, address indexed challenger, uint256 fine, uint256 reward);`

`event NoFraud(bytes32 indexed rnd, address indexed submitter, address indexed challenger, uint256 compensation);`

`event Transfer(address indexed from, address indexed to, uint256 value);`

## 8. withdrawMissedProfit

`function withdrawMissedProfit()`

**Function：**

Any account can call this method to transfer the missed profits to the foundation account. Missed revenue refers to the revenue missed when the submitter fails to respond to the challenge on time, or fails when submitting the proof in the first stage.

**Event：**

`event Transfer(address indexed from, address indexed to, uint256 value);`

## 9. alterSetting

`function alterSetting(IFileProof.SettingInfo memory si, bytes[5] memory signs)`

**Function：**

Community members obtain multi-signatures based on actual conditions to modify the basic configuration of FileProof.

**Parameters：**

si - new configuration information

signs - Multi-signature information of community members

`SettingInfo` structure is defined as follows:

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

**Event：**

`event AlterSetting(SettingInfo si);`

## 10. selectFiles

`function selectFiles(uint256 i) external view returns (bytes32[4] memory commitment)`

**Function：**

After a valid random number is generated in each period, the contract will select a batch of documents to be certified based on the random number, enter the index of this batch of files, and return the corresponding file commitment value.

**Notice：**

The total number of selected files is determined by `SettingInfo.chalSum`.

## 11. getCommit

`function getCommit(uint256 i) external view returns (uint256 sum, bytes32[4] memory commitment)`

**Function：**

According to the serial number, get the total number of uploaded files in the contract and the corresponding commitment value.

## 12. getFileInfo

`function getFileInfo(bytes memory commit) external view returns (uint64 index, uint256 expiration)`

**Function：**

According to the file commitment value, obtain the file serial number and expiration time.

## 13. getProofInfo

`function getProofInfo() external view returns (bytes32 y)`

**Function：**

Get certification information for the current period.

**Notice：**

Historical proof information and current complete proof information can be obtained by monitoring events.

## 14. getVerifyInfo

`function getVerifyInfo() external view returns (bytes32 rnd, bool lock, uint256 last)`

**Function：**

Get verification information for the current period.

**Notice：**

Historical verification information, as well as current complete verification information, can be obtained by monitoring events.

## 15. getProfitInfo

`function getProfitInfo() external view returns (uint256 pendingProfit, uint256 missedProfit, uint256 finalExpire)`

**Function：**

Get earnings information for the current period.

**Notice：**

Historical income information can be obtained by monitoring events.

## 16. getChallengeInfo

`function getChallengeInfo() external view returns (uint8 chalStatus, address challenger, uint8 chalIndex, uint256 startIndex, uint256 chalLength)`

**Function：**

Get challenge information for the current period.

**Notice：**

Historical challenge information and current complete challenge information can be obtained by monitoring events.

## 17. getSettingInfo

`function getSettingInfo() external view returns (uint32 interval, uint32 period, uint32 chalSum, uint32 respondTime, uint64 price, address submitter, address receiver, address foundation, uint8 chalRewardRatio, uint256 chalPledge)`

**Function：**

Get the current configuration information.

**Notice：**

Historical configuration information can be obtained by monitoring events.

## 18. getVK

`function getVK() external view returns (bytes32[8] memory vk)`

**Function：**

Get the current verification public key.

**Notice：**

Historical verification public keys can be obtained by monitoring events.
