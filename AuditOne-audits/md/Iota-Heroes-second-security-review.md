| Repository | Review Commit Hash | 
| -------- | -------- |
| [IOTA-Heroes](https://github.com/Crelde/lOTA-Heroes-Contracts)     | ff84cafbacf3664ae0ce958ea408bedbf6e53705 |

---

**[L-01]**

**Issue category**
Low

**Issue title**
Missing input validation for `heroId` and `address`

**Where**
Almost all functions where it is used

**Impact** 
An attacker can pass in an invalid or non-existent hero ID, or a zero address potentially causing the function to behave in unexpected ways, leading to security vulnerabilities.

**Description**
There are no checks to ensure that the hero IDs and address are valid or exist in the contract. 

**Recommendations to fix**
A check should be implemented to determine the authenticity of the hero IDs and addres

---

**[QA-01]**

**Issue category**
Informational

**Issue title**
Hardcoded value

**Where**
https://github.com/Crelde/IOTA-Heroes-Contracts/blob/af81f61c1bca1c1ed13f7e09b1b2b44ec7630093/contracts/Factory.sol#L206

**Impact**
Hardcoding values can make the contract less flexible and adaptable to changes.

**Description**
In this context, if the maximum allowable value of "10" needs to be adjusted in the future, you would need to locate and modify every occurrence of "10" in the code. This can be error-prone and time-consuming, especially if the value is used in multiple places.

**Recommendations to fix**
To make the code more flexible and maintainable, it is recommended to define this magic number as a constant with a descriptive name.

---

**[QA-02]**

**Issue category**
Informational

**Issue title**
Import declarations should import specific identifiers rather than the whole file

**Where**
All Contracts
`import "@openzeppelin/contracts/security/ReentrancyGuard.sol";`

**Impact**
Optimisation

**Description**
Using import declarations of the form import {<identifier_name>} from "some/file.sol" avoids polluting the symbol namespace making flattened files smaller, and speeds up compilation.

**Recommendations to fix**
Consider using import declarations of the form import {<identifier_name>} from "some/file.sol".

---

**[QA-03]**

**Issue category**
Informational

**Issue title**
Error in commented code

**Where**
https://github.com/Crelde/IOTA-Heroes-Contracts/blob/af81f61c1bca1c1ed13f7e09b1b2b44ec7630093/contracts/HeroMarketplace.sol#L192

**Impact**
Can lead to confusion

**Description**
There is an error in natspec comment. 
`/// @param tokenId - the NFT assed queried for royalties`

**Recommendations to fix**
It is supposed to be:
`/// @param tokenId - the NFT Id queried for royalties`

---

**[QA-04]**

**Issue category**
Medium risk

**Issue title**
Centralization risk 

**Where**
Whole project

**Impact**
Contracts have owners with privileged rights to perform admin tasks and need to be trusted to not perform malicious updates or drain funds

**Description**
In a smart contract, modifiers such as `onlyOwner`, `onlyResolvers` etc can be used to grant certain addresses with special privileges or roles. For example, the owner variable might be used to grant the owner of the contract the ability to modify the contract’s behavior or to perform certain actions that are not available to other users. However, this also means that the owner’s private key is a critical asset that needs to be protected. If an attacker were to gain access to the owner’s private key, they could potentially perform actions on behalf of the owner that could harm end-users of the contract.

**Recommendations to fix**
To mitigate this risk, it’s important to follow best practices for securing private keys and to use a hardware wallet or other secure storage solution to protect them. It’s also a good idea to regularly review the privileges granted to the owner and to consider whether they are necessary, and to consider using alternative mechanisms for granting privilege
