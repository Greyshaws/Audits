| Repository | Review Commit Hash | 
| -------- | -------- |
| [IOTA-Heroes](https://github.com/Crelde/lOTA-Heroes-Contracts)     | 9ef930a70aa555dacf0f8d402396964ff3178d73 |

---

**[H-01]**

**Issue category**
High

**Issue title**
Bad source of randomness

**Where**
Lib.sol

**Impact**
The usage of block.timestamp as a source of randomness is generally discouraged due to its susceptibility to manipulation by calling contracts. In this instance, an attacker can manipulate the output of the random function, resulting in the ability to accurately predict its outcomes to a significant extent.

**Description**

`function random( uint256 input, uint256 maxNum) public view returns(uint256) {
    uint256 psrand = uint256(
        keccak256(abi.encodePacked(block.timestamp + input))
    );
    return psrand % maxNum;
}
`

The vulnerability in the random function of the given smart contract is due to its reliance on block.timestamp as a source of entropy. block.timestamp returns the timestamp of the block that the current transaction is included in. While this value appears to be random, it is possible for miners to manipulate the timestamp of a block within certain limits to increase their chances of mining it. This manipulation is known as "timestamp" manipulation.
Attackers can exploit this vulnerability by submitting transactions with timestamps that are slightly in the future or the past. By doing so, they can control the output of the random function and predict its results with a high degree of accuracy. For example, an attacker could submit a transaction with a timestamp that is slightly in the future, wait for the transaction to be included in a block, and then call the random function with the same input value. Since the random function relies on the timestamp of the block, which is now the future timestamp submitted by the attacker, the output of the function will be predictable.
Moreover, the random function in the given smart contract only uses a single hash function, keccak256, which is deterministic and can be easily brute-forced by an attacker to predict future values. This makes the function even more vulnerable to exploitation by attackers.

**Recommendations to fix**
To address this vulnerability, the contract should use a more secure source of entropy, such as a random oracle or a verifiable delay function. A random oracle is a source of randomness that cannot be predicted, while a verifiable delay function introduces a delay in the computation of the random value to prevent attackers from manipulating it. A good example is Chainlinks VRF.
Additionally, the contract should use a more robust hash function, such as SHA-3, which is less susceptible to brute-forcing attacks. By using a more secure source of entropy and a robust hash function, the random function can be made more secure and less vulnerable to attacks.

---

**[H-02]**

**Issue category**
High

**Issue title**
setTradeOffer and cancelTradeOffer functions does not use ReentrancyGuard

**Where**
TradingPost.sol

**Impact**
If an attacker initiates a recursive call and enters either of these functions before they are executed completely, they can modify the contract state and potentially take advantage of it for their own gain.

**Description**
In the smart contract, the ReentrancyGuard from OpenZeppelin is imported to prevent reentrancy attacks. However, the functions setTradeOffer and cancelTradeOffer are not utilizing the ReentrancyGuard, making them vulnerable to reentrancy attacks. Both the setTradeOffer function and the cancelTradeOffer transfers tokens and Ether to a contract address, If an attacker can create a recursive call and re-enter either of these functions before they finish executing, they can potentially manipulate the state of the contract and exploit it to their advantage.

**Recommendations to fix**
To prevent this type of attack, it is important to use the ReentrancyGuard or other similar methods to ensure that the functions are executed atomically and not re-entered until the previous invocation has completed.

---

**[H-03]**

**Issue category**
High

**Issue title**
Reentrancy exploit in initiateAdventure function.

**Where**
HeroAdventure.sol

**Impact** 
This issue would allow the attacker to drain the contract's funds by repeatedly calling the payout function.

**Description**
The `_initiateAdventure` function is called both by the `goAdventure` function and the `contractAdventure` function. In the `contractAdventure` function, the user needs to pay an amount of ether to rent the hero before starting the adventure. However, the ether transfer to the contract owner happens before the `_initiateAdventure` function is called. An attacker could potentially create a malicious contract that would call the `contractAdventure` function and then recursively call it again before the first call is complete, exploiting the `goAdventure` function as well.

Recommendations to fix
One possible solution is to use the "Checks-Effects-Interactions" pattern. This means that the contract should first check if the calling conditions are met, then perform all internal state changes, and finally interact with other contracts. By following this pattern, it becomes more difficult for an attacker to exploit the contract's logic. Specifically, in this case, the developer should ensure that the ether transfer to the contract owner happens after the `_initiateAdventure` function is called, rather than before. This can be achieved by using a withdrawal pattern where the contract owner can withdraw the funds after the adventure is completed and the payout function has been called.
It is also recommended to use the ReentrancyGuard library from OpenZeppelin

---

**[M-01]**

**Issue category**
Medium

**Issue title**
Missing input validation for various functions

**Where**
Lib.sol

**Impact**
An attacker can enter in a zero address or malicious address, leading to the funds being stolen or other attacks.

**Description**

`function setGold(address addr) external onlyOwner {
        gold = addr;
    }

    function setHeroTokens(address addr) external onlyOwner {
        heroes = addr;
    }

    function setSkills(address addr) external onlyOwner {
        skills = addr;
    }

    function setGameTokens(address addr) external onlyOwner {
        tokens = addr;
    }

    function setAdventure(address addr) external onlyOwner {
        adventure = addr;
    }

    function setTraining(address addr) external onlyOwner {
        training = addr;
    }
`

The functions above do not explicitly check whether the inputted address is a zero address or a malicious address. This can allow an attacker to set an arbitrary address as the token address, potentially leading to funds being sent to the attacker or other unintended consequences.

**Recommendations to fix**
It would be a good practice to add input validation to ensure that the inputted address is not a zero/malicious address. This can be done by adding a require statement at the beginning of each function to check whether the inputted address is valid. For example:

`function setGold(address addr) external onlyOwner {
    require(addr != address(0), "Invalid address");
    gold = addr;
}
`

---

**[M-02]**

**Issue category**
Medium

**Issue title**
Use of low level calls

**Where**
HeroAdventure.sol

**Impact**
It can lead to unexpected behavior and may result in funds being lost or stolen.

**Description**

`
(bool feeSuccess, ) = _feeReceiver.call{value: fee}("");
(bool payoutSuccess, ) = HeroNFT(lib.heroes()).ownerOf(heroId).call{
                value: rental.rentPrice - fee
            }("");
`

The call function is a low-level function in Solidity that allows contracts to call other contracts and pass in arbitrary data. While it can be useful in certain situations, it can also be dangerous if not used properly. The main issue with the `call` function is that it allows external code to be executed within the context of the calling contract, which can potentially lead to reentrancy attacks and other vulnerabilities. In addition, the `call` function does not perform any input validation or type checking, which can lead to unexpected behavior if the input data is not formatted correctly.
In the first `call` statement, the contract is sending a value amount to the `_feeReceiver` address. An attacker could enter in a malicious or zero address, thereby leading to a loss of funds.

Recommendations to fix
To mitigate this issue, it's generally recommended to avoid using the `call` function wherever possible, and to use higher-level functions like `send` or `transfer` instead. These higher-level functions provide more safety guarantees and are less prone to errors and vulnerabilities.. It is also important to validate the `_feeReceiver` addresses before sending ether.

---

**[L-01]**

**Issue category**
Low

**Issue title**
Missing or insufficient Natspec comments

**Where**
Factory.sol, GameItem.sol, Gold.sol, HeroAdventure.sol, HeroNFT.sol, HeroSkills.sol, Lib.sol, Reforging.sol, Researching.sol, Shop.sol, TradingPost.sol, TrainingFacility.sol.

**Impact**
It can lead to misinterpretations, errors, and vulnerabilities in the code, which can have significant negative consequences for the security and functionality of the smart contract.

**Description**
The issue of missing/insufficient Natspec in Solidity code is one that can have a significant impact on the quality of the code, and ultimately, the functionality of the smart contract. Natspec is a tool that allows developers to add human-readable comments to their code, providing context, and explanations of the code's purpose and functionality. When Natspec is missing from Solidity code, other developers who read the code may struggle to understand what the code is doing, making it harder to modify, maintain, and reuse the code.

**Recommendations to fix**
Consider including Natspec comments in the contract code, explaining its purpose and functionality.

---

**[QA-01]**

**Issue category**
Informational

**Issue title**
Missing event logging

**Where**
GameItem.sol

**Impact**
It makes it difficult to monitor and reflect critical state changes

**Description**
This contract does not emit any events to notify external actors of changes to its state, which makes it harder to monitor and detect potential exploits or attacks.

**Recommendations to fix**
Consider logging events and adding event emissions to functions.

---

**[QA-02]**

**Issue category**
Informational

**Issue title**
Inconsistent naming convention

**Where**
TradingPost.sol

**Impact**
It can cause confusion and error

**Description**
The inconsistency in naming convention in the `setfeeReceiver` function of the contract can cause confusion and errors. It should be changed to `setFeeReceiver` to match the naming convention used in other functions of the contract. This will help improve readability and avoid confusion when interacting with the contract.

**Recommendations to fix**
`setfeeReceiver` should be changed to `setFeeReceiver`

---

**[QA-03]**

**Issue category**
Gas optimization

**Issue title**
Potential gas limit issues

**Where**
Researching.sol

**Impact**
Gas limits could be exceeded

**Description**
The `research()` function has multiple for loops that could cause gas limits to be exceeded if the reward structure is too complex.

**Recommendations to fix**
A potential fix could be to limit the maximum number of rewards that can be added to a token identifier, reducing the number of iterations needed to calculate the reward probability.