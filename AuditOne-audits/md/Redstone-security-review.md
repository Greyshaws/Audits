| Repository | Review Commit Hash | 
| -------- | -------- |
| [Redstone-finance](https://github.com/redstone-finance/redstone-oracles-monorepo)     | 3f466fb7d6b74acc4d94457c6aca666f0410bc5 |

---

**[M-01]**

**Issue title**
The minimum interval between updates is short and poses risk of front-running attack

**Where**
https://github.com/redstone-finance/redstone-oracles-monorepo/blob/e19d97d0f3bb5d93a59af7a115da42908c2b9777/packages/on-chain-relayer/contracts/core/RedstoneAdapterBase.sol#L26

**Impact**
An attacker can exploit the time gap between transactions to gain an advantage or manipulate the contract's state.

**Description**
The `MIN_INTERVAL_BETWEEN_UPDATES` constant, as its name suggests, represents the minimum amount of time that must pass between updates. In this case, a 3-second interval has been set, meaning that the updates cannot occur more frequently than once every 3 seconds.

This suggests that a 3-second interval is considered low, which means that it may not provide enough time for transactions to be processed before a new update occurs. This can create a vulnerability known as front-running, where an attacker can exploit the time gap between transactions to gain an advantage or manipulate the contract's state.

**Recommendations to fix**
To mitigate this risk, it is necessary to set a higher minimum interval between updates or implement other additional security measures, to prevent front-running attacks.

---

**[M-02]**

**Issue title**
Missing input validation for validateAndUpdateDataFeedsValues and _updateDataFeedValue functions

**Where**
Whole project
PriceFeedAdapterBase.sol

**Impact**
An attacker can enter in an invalid `dataFeedsId` or malicious value, leading to the funds being stolen or other attacks.

**Description**
The input parameters of most functions in the contracts are not validated, meaning that incorrect or malicious input could cause unexpected behavior or even result in errors or contract failures. For instance, the input parameters of the `validateAndUpdateDataFeedsValues` are not validated. An attacker could enter in a malicious value thereby leading to errors.

**Recommendations to fix**
It would be a good practice to add input validation to ensure that the inputted values are not malicious. This can be done by adding a require statement at the beginning of each function to check whether the input is valid.

---

**[L-01]**

**Issue category**
Low

**Issue title**
Use require instead of revert statement

**Where**
Whole project

**Impact**
Using `revert` statement can have unintended consequences, such as funds being locked up or the state of the contract being left in an inconsistent state.

**Description**
In Solidity, when writing smart contracts, it's important to have proper error handling to prevent unintended behavior and ensure the contract functions as intended. One common practice for error handling in Solidity is to use the `revert` statement. When the `revert` statement is called, all changes made during the transaction are reverted, and the remaining gas is refunded to the caller.

However, using `revert` statements can sometimes result in unexpected behavior if not used carefully. For example, if a sub-contract called during a transaction triggers a `revert` statement, the entire transaction will be reverted, including any changes made in the main contract before the sub-contract was called. This can have unintended consequences, such as funds being locked up or the state of the contract being left in an inconsistent state.

**Recommendations to fix**
To avoid this issue, it's generally recommended to use `require` statements instead of `revert` statements for input validation. `require` statements also revert the transaction when a condition is not met, but they provide more fine-grained control over the transaction flow. Specifically, they can be used to validate input before any state changes are made, which helps prevent unexpected behavior and keeps the contract in a consistent state. They also make the intent of the code clearer.

---

**[QA-01]**

**Issue category**
Gas optimization

**Issue title**
Loop optimization

**Where**
Each for-loop in the project

**Impact**
Save gas for each loop interaction

**Description**
When a loop iterates many times, it causes the amount of gas required to execute the function to increase significantly. In Solidity, excessive looping can cause a function to use more than the maximum allowed gas, which causes the function to fail.
For instance, the `validateAndUpdateDataFeedsValues()` function in the `PriceFeedsAdapterWithRounds.sol` contract loops through an array of data feed values and updates each value in the contract storage. If the number of data feeds to update is too large, it may consume too much gas and exceed the block gas limit, causing the transaction to fail. This could be exploited by an attacker to perform a denial-of-service attack on the contract by sending a transaction with a large array of data feeds to update, causing legitimate transactions to fail.

**Recommendations to fix**
To reduce gas consumption, it's recommended to find ways to optimize the loop or potentially break the loop into smaller batches. The following pattern can also be used:

```
`uint256 cachedLen = array.length;
for(uint i; i < cachedLen;){

  unchecked {
    ++i;
  }
}`
```

---

**[QA-02]**

**Issue category**
Informational

**Issue title**
Missing event logging and emission

**Where**
Whole project

**Impact**
It makes it difficult to monitor and reflect critical state changes

**Description**
None of the functions in the contracts log nor emit any events to notify external actors of changes to its state, which makes it harder to monitor and detect potential exploits or attacks.

**Recommendations to fix**
Consider logging events and adding event emissions to functions.

---

**[QA-03]**

**Issue category**
Informational

**Issue title**
Consider a higher Solidity version

**Where**
Whole project

**Impact**
Contract could be susceptible to attacks or hacks, which could be prevented in a higher version

**Description**
When a new version of Solidity is released, it typically includes security updates and bug fixes, which can make your contract less vulnerable to attacks from hackers or malicious actors. Additionally, new versions of Solidity can also introduce new language features and improve existing ones, making it easier for developers to write efficient and safe code.

**Recommendations to fix**
It is recommended to use a higher solidity version primarily for security purposes.

---

**[QA-04]**

**Issue category**
Informational

**Issue title**
Unlocked/floating pragma

**Where**
Whole project

**Impact**
Contracts could get accidentally deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

**Description**
Every Solidity file specifies a version number of the format pragma solidity (^)0.8.* in the header. The caret (^) before the version number implies an unlocked pragma, meaning that the compiler will use the specified version and above, hence the term “unlocked”.

**Recommendations to fix**
For consistency and to prevent unexpected behavior in the future, it is recommended to remove the caret to lock the file onto a specific Solidity version.

---

**[QA-05]**

**Issue category**
Informational

**Issue title**
Missing/Insufficient Natspec comments

**Where**
Whole project

**Impact**
Natspec issues can lead to misinterpretations, errors, and vulnerabilities in the code, which can have significant negative consequences for the security and functionality of the smart contract.

**Description**
There are cases of missing or insufficient Natspec comments in the code.

**Recommendations to fix**
Consider including Natspec comments in the contract code, explaining its purpose and functionality.

---

**[QA-06]**

**Issue category**
Informational

**Issue title**
The getValueForDataFeedUnsafe function is unnecessary

**Where**
https://github.com/redstone-finance/redstone-oracles-monorepo/blob/e19d97d0f3bb5d93a59af7a115da42908c2b9777/packages/on-chain-relayer/contracts/price-feeds/with-rounds/PriceFeedsAdapterWithRounds.sol#L35-L37

**Impact**
This redundant function might clutter the code and make it harder to follow or update

**Description**
The `getValueForDataFeedUnsafe` function is unnecessary because it is equivalent to `getValueForDataFeedAndRound` with the getLatestRoundId as the second argument. It is redundant to have a separate function for this purpose, as the same result can be achieved with an existing function.

**Recommendations to fix**
It is to remove the `getValueForDataFeedUnsafe` function and use `getValueForDataFeedAndRound` instead whenever the latest round ID is needed. This simplifies the code and reduces redundancy, making it easier to maintain and less error-prone. Additionally, it is better to have only one function that returns a value for a given data feed and round ID, rather than having two functions that essentially do the same thing but with slightly different arguments.