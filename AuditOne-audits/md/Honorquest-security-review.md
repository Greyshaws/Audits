**[M-01]**

**Issue category** 
Medium

Issue title
Potential division by zero error in `payExtra` function

Where
https://gitlab.com/hq-team/honorquest-smart-contracts/-/blob/main/contracts/L2/Lair.sol#L264-L295

Impact
If any of the variables `Quartz`, `Jade`, `Ruby`, or `Fire` are zero, then the corresponding calculation `QuartzAmount / Quartz, JadeAmount / Jade, RubyAmount / Ruby, or FireAmount / Fire` will result in a division by zero error, which will cause the function to fail and potentially halt the execution of the entire smart contract

Description
In the `payExtra` function, the variables `Quartz, Jade, Ruby, and Fire` are used as denominators for calculating the bonus amounts. If any of these variables are zero, the function will encounter a division by zero error during the calculation. To prevent this error, it's important to check that these variables are not zero before performing the calculations.

Recommendations to fix
To prevent a division by zero error, you can add an additional check that specifically checks if the value of `Quartz` is zero, and skip the calculation altogether if it is. Here's an example of how you can modify the if statement to include this check:

```
`if (Quartz > 0) {
  QuartzAmount = convertedAmount * HonrPerQuartz / 100;
  convertedAmount = convertedAmount - QuartzAmount;
  QuartzAmount = QuartzAmount / Quartz;
} else {
  QuartzAmount = 0;
}`
```

By adding this additional check, the function will skip the calculation of `QuartzAmount` if `Quartz` is zero, and set `QuartzAmount` to zero to avoid a division by zero error.

---

**[M-02]**

**Issue category** 
Medium

**Issue title**
Missing input validation for various functions 

**Where** 
https://gitlab.com/hq-team/honorquest-smart-contracts/-/blob/main/contracts/L1/Gen1Minter.sol#L230
https://gitlab.com/hq-team/honorquest-smart-contracts/-/blob/main/contracts/L2/Character.sol#L59
https://gitlab.com/hq-team/honorquest-smart-contracts/-/blob/main/contracts/L1/Gen0Minter.sol#L94

**Impact** 
-  An attacker can pass in invalid or non-existent token IDs in the tokenId array, potentially causing the function to behave in unexpected ways, leading to security vulnerabilities.
-  An attacker can set an arbitrary address as the token address, potentially leading to funds being sent to the attacker or other unintended consequences.

**Description** 
- The [ `mintKnight` ](https://gitlab.com/hq-team/honorquest-smart-contracts/-/blob/main/contracts/L1/Gen1Minter.sol#L230) function takes an array of `tokenIds` as input, with the parameter name `tokenId`. The function expects this array to contain a specific number of token IDs based on the value of `_howMany` parameter passed to the function.
The function checks that the length of the `tokenId` array is equal to` _howMany * 2`. However, this check only ensures that the number of token IDs passed as input to the function is correct. It does not ensure that the token IDs are valid or exist in the contract.
- The [`lockLoot`](https://gitlab.com/hq-team/honorquest-smart-contracts/-/blob/main/contracts/L2/Character.sol#L59) function does not explicitly check whether the inputted address is a zero address or a malicious address.
- The [`mintTo`](https://gitlab.com/hq-team/honorquest-smart-contracts/-/blob/main/contracts/L1/Gen0Minter.sol#L94) function also does not explicitly check whether the `to` address is a zero address or a malicious address

**Recommendations to fix** 
- In the `mintKnight` function, a `for loop` should be added to validate each token ID within the `_tokenIds` array.  Within the for loop, a check should be implemented to verify if each token ID exists in the contract by calling the 1registry.tokenExists1 function. If the token ID is found to be invalid, the function should revert and throw an error.
Additionally, a check should added to verify if the token ID belongs to the user calling the function by calling the `registry.ownerOf` function. In cases where the token does not belong to the user, the function should revert and throw an error.
Through the incorporation of this additional input validation, the `mintKnight` function can effectively prevent attackers from passing invalid or non-existent token IDs in the `tokenId` array, effectively mitigating the vulnerability. 

```
`function mintKnight(
        uint256 _howMany,
        uint256 _amount,
        uint256 _deadline,
        uint256 _nonce,
        uint8 v,
        bytes32 r,
        bytes32 s,
        bool _moveToL2,
        uint256[] calldata _tokenIds
    ) external payable nonReentrant checkKnightPrice(_howMany, _amount) {
        // Check that the number of tokenIds is correct
        require(
            _tokenIds.length == _howMany * 2,
            "Invalid number of tokenIds"
        );
        
        // Check that each tokenId exists and belongs to the user
        for (uint i = 0; i < _tokenIds.length; i++) {
            require(
                registry.tokenExists(_tokenIds[i]),
                "Invalid token ID"
            );
            require(
                registry.ownerOf(_tokenIds[i]) == _msgSender(),
                "Token does not belong to the user"
            );
        }
        
        // Check for whitelisting and prevent unauthorized access
        _isValid(_msgSender(), _howMany, _deadline, _nonce, v, r, s);
        _beforeMint(_howMany, 0);
        
        // Transfer payment to honor quest contract
        IERC20Upgradeable honor = IERC20Upgradeable(registry.hqToken());
        honor.safeTransferFrom(_msgSender(), address(registry.hqContract()), _amount);
        
        _nonces[_msgSender()] = ++_nonce;
        _mintProcess(_msgSender(), _howMany, _moveToL2, true, 0);
        Vault(registry.vaultContract()).releaseToken(
            _tokenIds,
            _msgSender()
        );
        emit BurnSquireHorse(_msgSender(), _tokenIds);
    }`
```
- It is recommended to add input validation to the `lockLoot` and `mintTo` functions to ensure that the inputted addresses are not zero/malicious addresses. This can be done by adding a require statement at the beginning of each function to check whether the inputted address is valid.


---

**[M-03]**

**Issue category**
Medium

**Issue title**
Use of `blockhash` as a sole source of randomness

**Where**
https://gitlab.com/hq-team/honorquest-smart-contracts/-/blob/main/contracts/L2/HonrQuest.sol#L164-L181
https://gitlab.com/hq-team/honorquest-smart-contracts/-/blob/main/contracts/L2/Lair.sol#L414-L422
https://gitlab.com/hq-team/honorquest-smart-contracts/-/blob/main/contracts/L2/MetaGeneration.sol#L243-L250
https://gitlab.com/hq-team/honorquest-smart-contracts/-/blob/main/contracts/L2/Quest.sol#L313-L321

**Impact**
This could allow attackers to gain an unfair advantage in the game

**Description**
`blockhash()` is only available for the most recent 256 blocks (with the most recent block being block number `block.number-1`). Therefore, attackers who are aware of the block number used can brute force the input to the smart contract in order to discover the random number beforehand.

**Recommendations to fix**
It is not recommended to use `blockhash()` as the sole source of randomness in the random() function. Rather, it should be used in combination with other sources to ensure the random number generation is more secure and resistant to attack.

---

**[M-04]**

**Issue category**
Medium

**Issue title**
Privileged roles and ownership

**Where**
All contracts

**Impact**
If the owner's private key is compromised, it could leave the door open for the attacker to act on the owner's behalf and potentially carry out actions that could harm the contracts end users

**Description**
In a smart contract, modifiers such as `onlyOwner`, `onlyTrusted` etc can be used to grant certain addresses with special privileges or roles. For example, the owner variable might be used to grant the owner of the contract the ability to modify the contract’s behavior or to perform certain actions that are not available to other users. However, this also means that the owner’s private key is a critical asset that needs to be protected. If an attacker were to gain access to the owner’s private key, they could potentially perform actions on behalf of the owner that could harm end-users of the contract.

**Recommendations to fix**
To mitigate this risk, it’s important to follow best practices for securing private keys and to use a hardware wallet or other secure storage solution to protect them. It’s also a good idea to regularly review the privileges granted to the owner and to consider whether they are necessary, and to consider using alternative mechanisms for granting privileges (such as multisignature contracts or contract upgrades) that may be more secure.

---

**[QA-01]**

**Issue category** 
Gas optimization

**Issue title**
Storage variable is updated twice in `lockLoser` function 

**Where** 
https://gitlab.com/hq-team/honorquest-smart-contracts/-/blob/main/contracts/L2/Jousting/Jousting.sol#L251-L256

**Impact** 
It could lead to increased gas consumption

**Description** 
`-player` variable is updated here:
`CharacterLib.TournamentPartcipant storage _player = knights[_tokenId];`
And it is also updated here:
`knights[_tokenId] = _player;`

**Recommendations to fix** 
`knights[_tokenId] = _player;` This should be removed.

---

**[QA-02]**

**Issue category**
Informational 

**Issue title** 
Misspelt variable and function names in `Jousting.sol`

**Where** 
Jousting.sol

**Impact** 
It can make it harder to read and understand the code.

**Description** 
There is a typo in the variable and function names "CharacterLib.TournamentPartcipant" and " finishTournment" 

**Recommendations to fix** 
It should be "CharacterLib.TournamentParticipant" and "finishTournament"

---

**[QA-03]**

**Issue category** 
Informational

**Issue title** 
Use descriptive variable names

**Where** 
https://gitlab.com/hq-team/honorquest-smart-contracts/-/blob/main/contracts/L2/MetaGeneration.sol#L156
https://gitlab.com/hq-team/honorquest-smart-contracts/-/blob/main/contracts/L2/MetaGeneration.sol#L219

**Impact** 
Could lead to confusion and unreadability

**Description** 
The code uses some variable names like "t", "s", which may not be very descriptive. 

**Recommendations to fix** 
It's recommended to use more descriptive variable names to make the code more understandable.

---

**[QA-04]**

**Issue category**
Informational 

**Issue title** 
Values are Hardcoded

**Where** 
https://gitlab.com/hq-team/honorquest-smart-contracts/-/blob/main/contracts/L2/MetaGeneration.sol#L88-L97

**Impact** 
Hard-coded booleans, numbers, and strings in the code are hard to understand, and they are prone to error, because after some time, their origin and context can be forgotten or mistaken.

**Description** 
In this context, the code contains a few magic numbers such as 100, 99 etc, which are not explained and may be difficult to understand for someone who is not familiar with the code.

**Recommendations to fix** 
To make the code more readable and maintainable, it is recommended to define these magic numbers as constants with descriptive names. For example, instead of hard-coding the value 99, the code can define a constant named MAX_COUNT with a value of 99. This makes the code more readable and easier to understand, as well as allowing for easier modifications to the code in the future.

For the complex values, consider adding a comment explaining how they were calculated or why they were chosen.

---

**[QA-05]**

**Issue category** 
Informational

**Issue title** 
Error in `require` statements

**Where** 
https://gitlab.com/hq-team/honorquest-smart-contracts/-/blob/main/contracts/L2/HonrQuest.sol#L185-L186

**Impact** 
It could be misleading and lead to confusion

**Description** 
In the `require` statements, both are checking for "Transfer to the zero address". The second one should be checking for "Transfer from the zero address".

**Recommendations to fix** 
Change second require statement to "Transfer from the zero address".

---

**[QA-06]**

**Issue category** 
Informational

**Issue title** 
Missing/Insufficient/Misspelt Natspec comments

**Where** 
All contracts
[https://gitlab.com/hq-team/honorquest-smart-contracts/-/blob/main/contracts/L1/Claim.sol#L59](url)

**Impact** 
Natspec issues can lead to misinterpretations, errors, and vulnerabilities in the code, which can have significant negative consequences for the security and functionality of the smart contract.

**Description** 
There are cases of missing or insufficient Natspec comments in the code. And some comments are misspelt.

**Recommendations to fix** 
Consider including Natspec comments in the contract code, explaining its purpose and functionality. Also fix misspelt comments.

---

**[QA-07]**

**Issue category** 
Informational

**Issue title**
 Consider a higher Solidity version

**Where** 
All contracts

**Impact** 
Contract could be susceptible to attacks or hacks, which could be prevented in a higher version

**Description** 
When a new version of Solidity is released, it typically includes security updates and bug fixes, which can make your contract less vulnerable to attacks from hackers or malicious actors. Additionally, new versions of Solidity can also introduce new language features and improve existing ones, making it easier for developers to write efficient and safe code. 

**Recommendations to fix** 
It is recommended to use a higher solidity version primarily for security purposes.

---

**[QA-08]**

**Issue category** 
Gas optimization

**Issue title** 
Excessive gas consumption in loop

**Where** 
[https://gitlab.com/hq-team/honorquest-smart-contracts/-/blob/main/contracts/L1/HQ.sol#L58-L59](url)

**Impact** 
It can increase gas costs and hinder smooth execution of code

**Description** 
When a loop iterates many times, it causes the amount of gas required to execute the function to increase significantly. In Solidity, excessive looping can cause a function to use more than the maximum allowed gas, which causes the function to fail.
In the case of the provided line of code, if the `_length` variable is very large, the loop could iterate thousands or even millions of times. This would lead to the `giveaway` function being called several times, causing gas costs to increase. 

**Recommendations to fix** 
To reduce gas consumption, it's recommended to find ways to optimize the loop or potentially break the loop into smaller batches. 

---

**[QA-09]**

**Issue category** 
Informational

**Issue title** 
Use of unlocked/floating pragma

**Where** 
All contracts

**Impact** 
Contracts could get accidentally deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

**Description** 
Every Solidity file specifies a version number of the format pragma solidity (^)0.8.*in the header. The caret (^) before the version number implies an unlocked pragma, meaning that the compiler will use the specified version and above, hence the term “unlocked”.

**Recommendations to fix** 
For consistency and to prevent unexpected behavior in the future, it is recommended to remove the caret to lock the file onto a specific Solidity version.




