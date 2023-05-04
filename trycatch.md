**Context:**  [SocketDst.sol](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/socket/SocketDst.sol#L190)

**Severity:** Low

**Description:**
The contract `SocketDst` uses try-catch mechanism to catch the errors while executing the call at the dst chain. However, this might not be fully implemented properly, as passing an EOA or contract with 0 code size (at that moment) will revert. 

- At the `_execute` function `localPlug_` is called with try-catch.
- At try-catch when we call an EOA or contrct with 0 code size (at that moment) will revert the entire transaction, preventing the execution of the entire transaction, without registering the error in the catch condition. 

```solidity
    function _execute(
        address executor,
        uint256 executionFee,
        address localPlug_,
        uint256 remoteChainSlug_,
        uint256 msgGasLimit_,
        bytes32 msgId_,
        bytes calldata payload_
    ) internal {
        try
            IPlug(localPlug_).inbound{gas: msgGasLimit_}(
                remoteChainSlug_,
                payload_
            )
        {
            executionManager__.updateExecutionFees(
                executor,
                executionFee,
                msgId_
            );
            emit ExecutionSuccess(msgId_);
        } catch Error(string memory reason) {
            // catch failing revert() and require()
            messageExecuted[msgId_] = false;
            emit ExecutionFailed(msgId_, reason);
        } catch (bytes memory reason) {
            // catch failing assert()
            messageExecuted[msgId_] = false;
            emit ExecutionFailedBytes(msgId_, reason);
        }
    }
 ```

This means, that the transaction can't be relayed or execution can never be done, as it will always revert, bypassing the design decision for which try-cath was used.

**Recommendation:**
Always check if the contract exists before using it at try-catch. Adding a small check would mitigate this issue.
