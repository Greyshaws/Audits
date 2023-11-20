# NOUNS DAO AUDIT

# PROJECT NAME: TOKEN BUYER

| Repository | Review Commit Hash | 
| -------- | -------- |
| [token-buyer](https://github.com/nounsDAO/token-buyer/tree/23d64ac7093f504ad4731bc4cf8d41b2c2943657)     | 23d64ac7093f504ad4731bc4cf8d41b2c2943657 |

---

**[M-01]**

**Issue category**
Medium

**Issue title**
VALUES ARE HARDCODED

**Where**
TokenBuyer.sol

**Description**
In the function `ethNeeded()`, there is a calculation that involves a hard-coded value of 10,000. More specifically, the buffer applied to the cost of tokens is `(bufferBPs + 10,000) / 10,000`. This means that every time the function is called, the buffer will be calculated using this same value of 10,000. 

**Impact** 
The problem with hard-coding values like this is that they can become difficult to modify later on. It is also problematic if the function is part of a larger system or contract, where changes to one function could affect the behavior of others.
Furthermore, magic numbers like 10,000 can be confusing to other developers who are unfamiliar with the code, as they may not understand why that specific value was chosen. This can lead to confusion, errors, and bugs down the road.

**Recommendations to fix** 
To mitigate this issue, one practice is to define these values as named constants or configuration options that can be easily modified throughout the codebase. For example, in this case, the value of 10,000 could be defined as a constant at the top of the file, like:

`uint256 constant BASIS_POINTS = 10000;`

Then, wherever the value is needed, it can be referenced using this constant rather than the raw number. This makes it easier to modify the value later on if needed, and also makes the code more readable and understandable for others who are reviewing it.

---

**[M-02]**

**Issue category** 
Medium

**Issue title**
RISK OF INFINITE LOOP IN `payBackDebt` FUNCTION

**Where**
Payer.sol

**Description**
The `payBackDebt` function contains a `while` loop that iterates over the queue data structure to pay back debt entries. The loop continues to run until either there are no more tokens left to pay back the debt or the queue is empty.
If the queue is not empty, but there are no more tokens left to pay back the debt, the while loop will continue to run indefinitely. This can happen if there is a bug or error in the code that prevents the loop from exiting when it should, or if there is a malicious attacker who creates a large number of debt entries in the queue, which causes the loop to keep running for an extended period of time.

**Impact**
This can be a serious problem, as it prevents users from being able to interact with the contract, and it can potentially cause them to lose their funds or other assets stored in the contract.

**Recommendations to fix** 
To mitigate the risk of the while loop running indefinitely, it is important to ensure that there are proper exit conditions in place. In the function, the while loop should exit when either there are no more tokens left to pay back the debt, or the queue is empty.
To ensure that the while loop always exits, the code should include a check at the beginning of the loop to ensure that the queue is not empty. If the queue is empty, the loop should exit immediately. Additionally, the code should include a check at the end of the loop to ensure that there are still tokens left to pay back the debt. If there are no more tokens left, the loop should exit.

```
`function payBackDebt(uint256 amount) external {
    uint256 debtPaidBack = 0;
    
    // While there are tokens left, and debt entries exist
    while (amount > 0 && !queue.empty()) {
        // Check if the queue is empty before processing the next debt entry
        if (queue.empty()) {
            break;
        }

        // Get the oldest entry
        DebtQueue.DebtEntry storage debt = queue.front();

        // ...

        // Check if there are still tokens left to pay back debt before processing the next debt entry
        if (amount <= 0) {
            break;
        }
    }`
```

---

**[M-03]**

**Issue category** 
Medium

**Issue title**
MISSING INPUT VALIDATION FOR ADDRESS

**Where**
TokenBuyer.sol

**Description**
In the `buyETH` function, the `to` address is not validated.

**Impact** 
An attacker can enter in a zero address or malicious address, leading to the funds being stolen or other attacks.

**Recommendations to fix** 
Add necessary validation to address.

---

**[L-01]**

**Issue category**
Low

**Issue title**
`buyETH` FUNCTION DOES NOT CHECK ETH BALANCE

**Where**
TokenBuyer.sol

**Description**
The function doesn't have any explicit checks to ensure that the smart contract actually has enough ETH to transfer to the buyer. 

**Impact** 
If there is not enough ETH, this could cause the transfer to fail or result in unexpected behaviors.

**Recommendations to fix**
Add necessary checks to the function. For instance:

`// Check if the contract has enough ETH to send
    require(address(this).balance >= ethAmountPerTokenAmount(amount), "Insufficient ETH in contract to facilitate the transfer.");`
    
---
    
**[L-02]**

**Issue category**
Low

**Issue title**
USE OF `unchecked` KEYWORD

**Where**
TokenBuyer.sol

**Description**
In the `tokenAmountNeeded` function, the `unchecked` keyword is used to disable overflow checking for the addition operation performed on the `baselinePaymentTokenAmount` and `totalDebt` variables. 

**Impact** 
If the result of this addition operation exceeds the max limit of `uint256` variable, it will wrap around and the result will be truncated to fit within the `uint256` range. So, by using the `unchecked` keyword, the addition operation will not throw an exception or error if this happens.

**Recommendations to fix**
It is important to be extra cautious when using the `unchecked` keyword and make sure to handle the overflow situation properly. It is recommended to avoid using the `unchecked` keyword unless you have a good reason to do so and understand the potential risks it introduces.

---

**[L-03]**

**Issue category**
Low

**Issue title**
USE OF FLOATING PRAGMA

**Where**
TokenBuyer.sol

**Description**
Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

**Impact** 

**Recommendations to fix**

---

**[L-04]**

**Issue category**
Low

**Issue title**
USE OF `block.timestamp`

**Where**
PriceFeed.sol

**Description**
`block.timestamp` is a variable that returns the current timestamp of the current block in Unix time format. It is a reliable way to track the time in Ethereum, but it is important to note that it can be manipulated by miners to some extent. Miners can choose which transactions to include in a block and when to publish the block, which means they can potentially manipulate the block timestamp to some extent.

**Impact** 
In the `price()` function, `block.timestamp` is used to check if the oracle's price data is stale. If the timestamp is older than a certain amount of time specified by staleAfter, the function reverts the transaction. However, if miners manipulate the block timestamp to be earlier than the actual time, the function could potentially accept stale data.

**Recommendations to fix**
To mitigate this risk, it is important to ensure that the `staleAfter` value is set to an appropriate value that allows for some network delays and fluctuations in price data, and to use other sources of time verification, such as a trusted external timestamp oracle, to supplement the use of block.timestamp.