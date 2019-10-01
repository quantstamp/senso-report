<!-- markdownlint-disable MD033 -->

# senso

This smart contract audit was prepared by [Quantstamp](https://www.quantstamp.com/), the protocol for securing smart contracts.


## Executive Summary

| Category                  | Description                                                                                                                                                           |
| :------------------------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Type                      | Token and Crowdsale Contract Audit                                                                                                                                    |
| Auditor(s)                | Ed Zulkoski, Senior Security Engineer<br/>John Bender, Senior Research Engineer<br/>Poming Lee, Research Engineer<br/>                                                |
| Timeline                  | 2019-08-11 through 2019-09-27                                                                                                                                         |
| EVM                       | Constantinople                                                                                                                                                        |
| Language(s)               | Solidity                                                                                                                                                              |
| Method(s)                 | Architecture Review, Unit Testing, Functional Testing, Computer-Aided Verification, Manual Review                                                                     |
| Specification(s)          | README.md: <>                                                                                                                                                         |
| Source code               | Repository: [senso](https://github.com/sensoriumxr/senso)<br/>Commit: [11a0e9b](https://github.com/sensoriumxr/senso/commit/11a0e9bc80787876a3c2dbb316d8efdb731dbc82) |
| Total Issues              | 13                                                                                                                                                                    |
| High Risk Issues          | 0                                                                                                                                                                     |
| Medium Risk Issues        | 1                                                                                                                                                                     |
| Low Risk Issues           | 3                                                                                                                                                                     |
| Informational Risk Issues | 8                                                                                                                                                                     |
| Undetermined Risk Issues  | 1                                                                                                                                                                     |

| Severity Level | Explanation                                                                                                                                                                                                   |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| High           | The issue puts a large number of users’ sensitive information at risk, or is reasonably likely to lead to catastrophic impact for client’s reputation or serious financial implications for client and users. |
| Medium         | The issue puts a subset of users’ sensitive information at risk, would be detrimental for the client’s reputation if exploited, or is reasonably likely to lead to moderate financial impact.                 |
| Low            | The risk is relatively small and could not be exploited on a recurring basis, or is a risk that the client has indicated is low-impact in view of the client’s business circumstances.                        |
| Informational  | The issue does not pose an immediate threat to continued operation or usage, but is relevant for security best practices, software engineering best practices, or defensive redundancy.                       |
| Undetermined   | The impact of the issue is uncertain.                                                                                                                                                                         |

### Goals

This report focused on evaluating security of smart contracts, as requested by the senso team. Specific questions to answer:

- Check that the token is ERC20 compliant
- Ensure that funds cannot be locked
- Ensure that users will receive their purchased tokens

### Changelog

- Date: 2019-08-21 - Initial Report
- Date: 2019-09-03 - Report updated based on commit [15b7049](https://github.com/sensoriumxr/senso/commit/15b7049b3e952eae0c8aeaaea7bc45cdcc658479)

### Overall Assessment

The smart contracts generally adhere to the provided specification, with some small changes required. However, several issues may occur if contract owners or privileged token minters behave dishonestly or mint tokens outside the scope of the crowdsale. Users should be made aware of the responsibilities and powers of the SENSO contract owners.

**Update:** Sensorium addressed our comments as of commit [15b7049](https://github.com/sensoriumxr/senso/commit/15b7049b3e952eae0c8aeaaea7bc45cdcc658479).

-----

## Quantstamp Audit Breakdown

Quantstamp's objective was to evaluate the repository for security-related issues, code quality, and adherence to specification and best practices.
Possible issues we looked for included (but are not limited to):

- Transaction-ordering dependence
- Timestamp dependence
- Mishandled exceptions and call stack limits
- Unsafe external calls
- Integer overflow / underflow
- Number rounding errors
- Reentrancy and cross-function vulnerabilities
- Denial of service / logical oversights
- Access control
- Centralization of power
- Business logic contradicting the specification
- Code clones, functionality duplication
- Gas usage
- Arbitrary token minting  

### Methodology

The Quantstamp auditing process follows a routine series of steps:

1. Code review that includes the following:
    1. Review of the specifications, sources, and instructions provided to Quantstamp to make sure we understand the size, scope, and functionality of the smart contract.
    2. Manual review of code, which is the process of reading source code line-by-line in an attempt to identify potential vulnerabilities.
    3. Comparison to specification, which is the process of checking whether the code does what the specifications, sources, and instructions provided to Quantstamp describe.
2. Testing and automated analysis that includes the following:
    1. Test coverage analysis, which is the process of determining whether the test cases are actually covering the code and how much code is exercised when we run those test cases.  
    2. Symbolic execution, which is analyzing a program to determine what inputs cause each part of a program to execute.
3. Best practices review, which is a review of the smart contracts to improve efficiency, effectiveness, clarify, maintainability, security, and control based on the established industry and academic practices, recommendations, and research.
4. Specific, itemized, and actionable recommendations to help you take steps to secure your smart contracts.

### Toolset

The below notes outline the setup and steps performed in the process of this audit.

### Setup

Tool setup:

- [Truffle](https://truffleframework.com/) v5.0.31
- [Ganache](https://truffleframework.com/ganache) v2.7.0
- [solidity-coverage](https://github.com/sc-forks/solidity-coverage) v0.6.4
- [Oyente](https://github.com/melonproject/oyente) v1.4.0
- [Mythril](https://github.com/ConsenSys/mythril) v0.2.7
- [truffle-flattener](https://github.com/alcuadrado/truffle-flattener) v0.20.4
- [MAIAN](https://github.com/MAIAN-tool/MAIAN) commit sha: ab387e1
- [Securify](https://github.com/eth-sri/securify) commit sha: 13d4784
- [NodeJS](https://nodejs.org/en/) v8.16.0
      
## Assessment

### Findings
        
#### The total number of distributed tokens may exceed `tokensaleAmount`, which may cause `finalize()` to fail

**Status:** Fixed

**Contract(s) affected:** `SENSOCrowdsale`


**Severity:** Medium

**Description:** There are no checks that the distributed tokens are within the limits of the `_token.tokensaleAmount`. Further, if the total amount of approved and bought tokens exceeds `_token.tokensaleAmount()  + _token.closedSaleAmount()`, then on L440-444 of `finalize()`, `_token.tokensaleAmount()  + _token.closedSaleAmount()  - _token.totalFrozenTokens()  - _token.totalSupply()` will underflow, and subsequently the mint operation will fail due to the token cap, which will ultimately prevent the sale from ever being finalized. Note that the check in ERC20Capped is insufficient here, since `_token.tokensaleAmount()  + _token.closedSaleAmount() < _token.cap()`.

**Recommendation:** Add new checks in both `_preValidatePurchase()` functions ensuring that the total amount of purchased and frozen tokens does not exceed `SENSOToken.tokensaleAmount`. Avoid approving more token purchases than `SENSOToken.tokensaleAmount`.

#### Incorrect transfer from `beneficiary` instead of `msg.sender`

**Status:** Fixed

**Contract(s) affected:** `SENSOCrowdsale`


**Severity:** Low

**Description:** In `buyTokensWithTokens()`, on L283: `tradedToken.safeTransferFrom(beneficiary, _wallet, tokenAmountPaid);` should transfer from `msg.sender`, not the `beneficiary`. Note that `_isTokenApproved` correctly checks for approval for `msg.sender`, not the `beneficiary`. Alternatively, the `beneficiary` parameter should be removed from `buyTokens()` and `buyTokensWithTokens()`, instead relying upon `msg.sender`.

**Recommendation:** Change the argument `beneficiary` to `msg.sender` on L283.

#### Race Conditions / Front-Running

**Status:** Fixed

**Contract(s) affected:** `SENSOToken`, `SENSOCrowdsale`


**Severity:** Low

**Description:** A block is an ordered collection of transactions from all around the network. It's possible for the ordering of these transactions to manipulate the end result of a block. A miner attacker can take advantage of this by generating and moving transactions in a way that benefits themselves.
  

**Exploit Scenario:** If the amount of approved token purchases exceeds the token cap, there is a potential transaction-ordering dependence between two purchasers that may exceed the cap. For example, if the cap is 100 tokens and both Alice and Bob are approved to purchases 75 tokens, then only the first mined transaction will be allowed.

**Recommendation:** Ensure that the total amount of approved token purchases does not exceed the token cap.

*Update:* Sensorium will be using backend (off-chain) checks to ensure that `approve`/`tokenApprove` allowances will not exceed the cap, therefore avoiding such race-conditions.

#### `buyTokens()` may charge a higher rate than intended
**Contract(s) affected:** `SENSOCrowdsale`


**Severity:** Low

**Description:** If the `weiAmount * rate` does not evenly divide into `1e18`, the user may be charged more wei than intended. A similar issue occurs in `buyTokensWithTokens()`.

For example, if the rate is 2 tokens per 1 ETH and the user sends 1.25 ETH, then they only receive 2 tokens and be charged an extra 0.25 ETH.

**Recommendation:** In `buyTokens()`, transfer any excess wei back to the user. Add a similar transfer in `buyTokensWithTokens()`. Alternatively, add a require statement such as `require(weiAmount.mul(approval.rate).mod(1e18) == 0)`.

*Update:* Although this may still occur if a user interacts directly with the smart contract, Sensorium will provide a user-interface to perform these calculations in order to aid investors in sending the correct amounts.

#### Centralization of Power
**Contract(s) affected:** `SENSOToken`, `SENSOCrowdsale`


**Severity:** Informational

**Description:** Smart contracts will often have `owner` variables to designate the person with special privileges to make modifications to the smart contract. However, this centralization of power needs to be made clear to the users, especially depending on the level of privilege the contract allows to the owner.

In particular, the following privileges may be problematic:
* Minting can occur at any time from a minter address (including after the crowdsale completes).
* If the crowdsale owner never finalizes the sale, funds cannot be recovered, and frozen tokens cannot be unfrozen.


**Recommendation:** Ensure that the role of the SENSO contract owners are made explicit to users. If a more trustless system is desired, consider restricting the minting capabilities to only the crowdsale contract. Note that this would require restricting the use of `MinterRole._addMinter()`.

*Update:* Although the issue with minting is resolved as discussed above, funds may still be locked if the crowdsale is never finalized. Sensorium will ensure that this behaviour is communicated to investors in an open, clear and explicit manner.

#### `unfreezeTokens()` does not update `totalFrozenTokens`

**Status:** Fixed

**Contract(s) affected:** `SENSOToken`, `SENSOCrowdsale`


**Severity:** Informational

**Description:** While this is unlikely to be issue during the crowdsale, this could be problematic if it is necessary to mint additional frozen tokens after the crowdsale, as previously frozen tokens will be double counted. This could be an issue if, for example, a second crowdsale occurs.

**Recommendation:** Update `SENSOToken.totalFrozenTokens` when an unfreeze occurs.

#### Integer Overflow / Underflow

**Status:** Fixed

**Contract(s) affected:** `SENSOCrowdsale`


**Severity:** Informational

**Description:** Integer overflow/underflow occur when an integer hits its bit-size limit. Every integer has a set range; when that range is passed, the value loops back around. A clock is a good analogy: at 11:59, the minute hand goes to 0, not 60, because 59 is the largest possible minute.

Integer overflow and underflow may cause many unexpected kinds of behavior and was the core reason for the `batchOverflow` attack. Here's an example with `uint8` variables, meaning unsigned integers with a range of `0..255`.

```
function under_over_flow() public {
    uint8 num_players = 0;
    num_players = num_players - 1;  // 0 - 1 now equals 255!
    if (num_players == 255) {
        emit LogUnderflow();        // underflow occurred
    }

    uint8 jackpot = 255;
    jackpot = jackpot + 1;          // 255 + 1 now equals 0!
    if (jackpot == 0) {
        emit LogOverflow();         // overflow occurred
    }
}
```

**Recommendation:** As an added layer of security, consider using `add()` and `sub()` in place of `+` and `-` throughout (including uses of `+=`).

#### Unlocked Pragma

**Status:** Fixed

**Contract(s) affected:** `SENSOCrowdsale`, `SENSOToken`


**Severity:** Informational

**Description:** Every Solidity file specifies in the header a version number of the format `pragma solidity (^)0.4.*`. The caret (`^`) before the version number implies an unlocked pragma, meaning that the compiler will use the specified version _and above_, hence the term "unlocked." For consistency and to prevent unexpected behavior in the future, it is recommended to remove the caret to lock the file onto a specific Solidity version.

**Recommendation:** Remove the caret to lock the file onto a recent Solidity version.

#### All increases of `frozenTokens` should also increment `totalFrozenTokens`

**Status:** Fixed

**Contract(s) affected:** `SENSOCrowdsale`


**Severity:** Informational

**Description:** On L451-453, the `frozenTokens` mapping is updated, however `_deliverTokens()` should also be called (with the amount as the third argument) in order to update `SENSOToken.totalFrozenTokens`. This approach performs the token `cap()` checks appropriately.

**Recommendation:** For each of the above lines, also call `_deliverTokens(address, 0, amountFrozen)`.

#### `approve()` should restrict inputs that do not permit token sales

**Status:** Fixed

**Contract(s) affected:** `SENSOCrowdsale`


**Severity:** Informational

**Description:** If `rate / 1e18 > approval.limit`, then it is impossible to purchase tokens because `totalTokensAmount` in `_getTokenAmount()` will always be greater than `approval.limit`. A similar issue occurs in `tokenApprove()`.

**Recommendation:** Add `require(approval.rate / 1e18 <= approval.limit)` to `approve()`, and add a similar require statement to `tokenApprove()`.

#### Block Timestamp Manipulation

**Status:** Fixed

**Contract(s) affected:** `SENSOCrowdsale`


**Severity:** Informational

**Description:** Projects may rely on block timestamps for various purposes. However, it's important to realize that miners individually set the timestamp of a block, and attackers may be able to manipulate timestamps for their own purposes. If a smart contract relies on a timestamp, it must take this into account.
  

**Exploit Scenario:** Although the approved purchasing window is relatively large at 7 days, miners can manipulate the block's timestamp if a buyer is trying to buy tokens right before their cut-off time. This may be favorable for the miner if they wish to sell their tokens to the buyer at a higher price at a later time.

**Recommendation:** Consider using `block.number` instead of `block.timestamp`.

*Update:* As the purchasing windows are guaranteed to be relatively large (> 100 blocks), it is not feasible for miners to interfere with business logic.

#### `_deliverTokens()` should be invoked any time frozen tokens are allocated

**Status:** Fixed

**Contract(s) affected:** `SENSOCrowdsale`


**Severity:** Informational

**Description:** `SENSOToken.totalFrozenTokens` maintains the size of the full set of frozen tokens. However, this is only updated when `mint()` is called from `_deliverTokens()`. Since this is not called for the frozen token balance creations on L451-453, the `totalFrozenTokens` amount will be incorrect.

**Recommendation:** Add corresponding calls to `_deliverTokens()` for each of L451-453 (using the amount as the third parameter to indicate that they are frozen tokens). This will update `totalFrozenTokens` as well as perform the token cap checks.

#### Users may have difficulty unfreezing tokens due to the `unfreezeTime` parameter

**Status:** Fixed

**Contract(s) affected:** `SENSOCrowdsale`


**Severity:** Undetermined

**Description:** While having `unfreezeTime` as a parameter to `unfreezeTokens()` works (due to a user potentially having multiple token purchases), novice users may have difficulty unfreezing their funds if they have not recorded or forgotten their `unfreezeTime`.

**Recommendation:** Ensure users are explicitly provided with their `unfreezeTime` when approved, and if necessary provide instructions for obtaining their `unfreezeTime` on-chain through the indexed event parameters. Alternatively, consider storing only the latest `unfreezeTime` for each user such that all associated funds could be unfrozen after that time (in which case no `unfreezeTime` parameter is required in `unfreezeTokens`).

*Update:* Sensorium will be providing a Metamask-integrated user interface to aid in all necessary computations and perform function calls under the hood. If more advanced users wish to bypass this interface, they may obtain the necessary parameters on-chain through the events log.

### Test Results

#### Test Suite Results

```
Contract: SENSOCrowdsale
    Initial configuration
      ✓ Total supply is 769 200 000
      ✓ Closed sale balance is 200 000 000
      ✓ Reserve is 269 200 000 (115ms)
    Tokensale stage
      ✓ Closed sale wallet can transfer (125ms)
      ✓ advisory can NOT transfer
      ✓ userLoalty can NOT transfer
      ✓ partners can NOT transfer
      ✓ team can NOT transfer
      ✓ safeSupport can NOT transfer
      ✓ community can NOT transfer
      ✓ Investor can NOT transfer
    Approvals
      ✓ Cannot buy before approval (60ms)
      Initial approval (no freeze)
        ✓ Can approve (152ms)
        ✓ Can not approve second time (66ms)
        ✓ Can not approve with 0 limit (67ms)
        ✓ Can approve another investor (134ms)
        ✓ Can not buy more than approved (76ms)
        ✓ Can buy (130ms)
        ✓ Cannot be purchased by a random person in favour of someone approved (45ms)
        ✓ Rate is zeroed after the purchase
        ✓ Best before is zeroed after the purchase
        ✓ Can not buy again
        ✓ Another investor can buy (116ms)
      Initial approval (with freeze)
        ✓ Can NOT approve with no freeze time
        ✓ Can NOT approve with no share
        ✓ Can NOT approve with frozen share exceeeds 100%
        ✓ Can approve (101ms)
        ✓ Can buy with frozen amount (118ms)
        ✓ have correct number of frozen tokens
        ✓ Can approve 2nd time (102ms)
        ✓ Can buy with frozen amount 2nd time (114ms)
        ✓ frozen tokens stacks
        ✓ Can approve 3rd time (99ms)
        ✓ Can buy with frozen amount 3nd time (106ms)
        ✓ frozen tokens with different freeze time are fine living together
      Second approval
        ✓ Can be approved second time with another rate (78ms)
        ✓ Can buy using new approval (81ms)
    Purchasing with tokens
      ✓ Minting side tokens (112ms)
      ✓ Cannot be bought if not approved (73ms)
      ✓ Can not approve with 0 limit
      ✓ Can approve (98ms)
      ✓ Can not buy with more than approved (114ms)
      ✓ Can buy with tokens (174ms)
      ✓ Rate is zeroed after the purchase
      ✓ Best before is zeroed after the purchase
      ✓ Cannot buy again (82ms)
      ✓ Can approve again (93ms)
      ✓ Can approve another investor (97ms)
      ✓ Cannot buy with another token (79ms)
      ✓ Can not buy when purchase exceeds token balance (103ms)
    Crowdsale finalization
      ✓ Cannot be finalized by random person
      ✓ Can be finalized by crowdsale owner (108ms)
      ✓ Advisory can transfer (93ms)
      ✓ UserLoalty can transfer (96ms)
      ✓ Partners can transfer (109ms)
      ✓ Team can not unfreeze
      ✓ Team can transfer (40ms)
      ✓ SafeSupport can transfer (43ms)
      ✓ Community can transfer (39ms)
      ✓ Closed sale can transfer (90ms)
      ✓ Investor can transfer (88ms)
      ✓ Can not buy (39ms)
      ✓ Can not buy with tokens
      ✓ Can not approve (38ms)
      ✓ Investor can unfreeze tokens (83ms)
      ✓ Investor can not unfreeze twice
      ✓ Investor can not unfreeze tokens before freeze time
      ✓ Investor can unfreeze second frozen part after waiting appropriate time (86ms)
      12 months wait...
        ✓ Team can transfer (223ms)
        ✓ SafeSupport can transfer (89ms)
        ✓ Community can transfer (92ms)


  71 passing (5s)
```


#### Code Coverage

```
---------------------|----------|----------|----------|----------|----------------|
File                 |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
---------------------|----------|----------|----------|----------|----------------|
 contracts/          |       96 |    67.65 |    84.85 |    96.18 |                |
  SENSOCrowdsale.sol |    95.61 |    67.74 |    82.76 |     95.8 |... 182,190,430 |
  SENSOToken.sol     |      100 |    66.67 |      100 |      100 |                |
  TokenA.sol         |      100 |      100 |      100 |      100 |                |
  TokenB.sol         |      100 |      100 |      100 |      100 |                |
---------------------|----------|----------|----------|----------|----------------|
All files            |       96 |    67.65 |    84.85 |    96.18 |                |
---------------------|----------|----------|----------|----------|----------------|
```

The majority of missing branch coverage relates to untested failing paths of `require` statements. These include the checks to ensure that address arguments are non-zero, as well as the bounds-checks of `SENSOCrowdsale.approve()` and `SENSOCrowdsale.tokenApprove()`. Tests should be added to cover all branches. There were also several getter functions that were not covered.
### Automated Analyses


#### Oyente

Repository: <https://github.com/melonproject/oyente>

Oyente is a symbolic execution tool that analyzes the bytecode of Ethereum smart contracts. It checks if a contract features any of the predefined vulnerabilities before the contract gets deployed on the blockchain.

##### Oyente Findings

Oyente was unable to complete analysis due to an unsupported contract version.

#### Mythril

Repository: <https://github.com/ConsenSys/mythril>

Mythril is a security analysis tool for Ethereum smart contracts. It uses concolic analysis, taint analysis and control flow checking to detect a variety of security vulnerabilities.

##### Mythril Findings

Mythril reported no issues.

#### MAIAN

Repository: <https://github.com/MAIAN-tool/MAIAN>

MAIAN is a tool for automatic detection of trace vulnerabilities in Ethereum smart contracts. It processes a contract's bytecode to build a trace of transactions to find and confirm bugs.

##### MAIAN Findings

Maian was unable to successfully deploy the contracts, and hence was unable to complete analysis. This is again possibly related to an unsupported contract pragma version.
#### Securify

Repository: <https://github.com/eth-sri/securify>
##### Securify Findings

Securify reported three warnings, all of which were deemed false positives. Specifically, there were two `Unrestricted Write` warnings for the `nonReentrant` and `whenNotPaused` modifiers, and one `Locked Ether` warning associated with a blank line in the smart contract.

-----
  
## Adherence to Specification

The contract mostly adheres to the specification as described in the README, however the tokens associated with the closed sale appear to be double-minted, as described above.

## Code Documentation

The code is generally well-commented and properly specified in the `README.md`.

## Adherence to Best Practices

The smart contracts are generally well documented and mostly follow best practices, apart from the suggestions as described above. As some additional suggestions: 
* In `SENSOToken`, declare `SENSOToken.name` and `SENSOToken.symbol` as constants;
* In `SENSOCrowdsale.approve()`, convert `freezeShare < 101` to `freezeShare <= 100` for clarity;
* In `SENSOCrowdsale._getTokenAmount()`, declare `1e18` as a constant (e.g. "DECIMALS").
* In `SENSOCrowdsale`, `tokensPurchasedAmound` should instead be `tokensPurchasedAmount`.




-----

## Appendix

### File Signatures
The following are the SHA‌-256 hashes of the audited contracts and/or test files. A smart contract or file with a different SHA‌-256 hash has been modified, intentionally or otherwise, after the audit. You are cautioned that a different SHA-256 hash could be (but is not necessarily) an indication of a changed condition or potential vulnerability that was not within the scope of the audit.

#### Contracts


```
contracts/SENSOCrowdsale.sol: 89b81d9e7eebb2958d9d3761f95e53b11010423f785b118621ff40d2c256b5ff
contracts/TokenB.sol: 6746064bd96221e1424f098b5dc1b10ea7165893365aa9369bcb829c4622f37b
contracts/TokenA.sol: 53319880892489bfe2bd5b5142ae4966556cf7092a7c7ee8c8813d70e92af4ff
contracts/Migrations.sol: 1c4e30fd3aa765cb0ee259a29dead71c1c99888dcc7157c25df3405802cf5b09
contracts/SENSOToken.sol: cb36e63810cdbbaf1cd38bbea49ffbdaf2787e63a6cfd6b9ed588e969e4a8a42
```


#### Tests


```
test/01_test.js: 0f9272bd45e236085c653c3b81d3732c6ee52e3cf875dc7214fed15f50ac9ac2
```



#### Steps

Steps taken to run the tools:

- Installed Truffle: `npm install -g truffle`
- Installed Ganache: `npm install -g ganache-cli`
  - Installed the solidity-coverage tool (within the project's root directory): `npm install --save-dev solidity-coverage`
- Ran the coverage tool from the project's root directory: `./node_modules/.bin/solidity-coverage`
- Flattened the source code using `truffle-flattener` to accommodate the auditing tools.
- Installed the Mythril tool from Pypi: `pip3 install mythril`
- Ran the Mythril tool on each contract: `myth -x path/to/contract`  
- Ran the Securify tool `java -Xmx6048m -jar securify-0.1.jar -fs contract.sol`  
- Installed the Oyente tool from Docker: `docker pull luongnguyen/oyente`
- Migrated files into Oyente (root directory): `docker run -v $(pwd):/tmp -it luongnguyen/oyente`
- Ran the Oyente tool on each contract: `cd /oyente/oyente && python oyente.py /tmp/path/to/contract`
- Ran the MAIAN tool on each contract: `cd maian/tool/ && python3 maian.py -s path/to/contract contract`

- Installed NodeJS: https://nodejs.org/en/download/


## About Quantstamp

Quantstamp is a Y Combinator-backed company that helps to secure smart contracts at scale using computer-aided reasoning tools, with a mission to help boost adoption of this exponentially growing technology.

Quantstamp’s team boasts decades of combined experience in formal verification, static analysis, and software verification. Collectively, our individuals have over 500 Google scholar citations and numerous published papers. In its mission to proliferate development and adoption of blockchain applications, Quantstamp is also developing a new protocol for smart contract verification to help smart contract developers and projects worldwide to perform cost-effective smart contract security audits.
To date, Quantstamp has helped to secure hundreds of millions of dollars of transaction value in smart contracts and has assisted dozens of blockchain projects globally with its white glove security auditing services. As an evangelist of the blockchain ecosystem, Quantstamp assists core infrastructure projects and leading community initiatives such as the Ethereum Community Fund to expedite the adoption of blockchain technology.

Finally, Quantstamp’s dedication to research and development in the form of collaborations with leading academic institutions such as National University of Singapore and MIT (Massachusetts Institute of Technology) reflects Quantstamp’s commitment to enable world-class smart contract innovation.


### Timeliness of content

The content contained in the report is current as of the date appearing on the report and is subject to change without notice, unless indicated otherwise by Quantstamp; however, 
Quantstamp does not guarantee or warrant the accuracy, timeliness, or completeness of any report you access using the internet or other means, and assumes no obligation to update any information following publication.
publication.

### Notice of Confidentiality

This report, including the content, data, and underlying methodologies, are subject to the confidentiality and feedback provisions in your agreement with Quantstamp. 
These materials arenot to be disclosed, extracted, copied, or distributed except to the extent expressly authorized by Quantstamp.

### Links to other websites

You may, through hypertext or other computer links, gain access to web sites operated by persons other than Quantstamp, Inc. (Quantstamp). Such hyperlinks are provided for your reference and convenience only, and are the exclusive responsibility of such web sites&apos; owners. 
You agree that Quantstamp are not responsible for the content or operation of such web sites, and that Quantstamp shall have no liability to you or any other person or entity for the use of third-party web sites. 
Except as described below, a hyperlink from this web site to another web site does not imply or mean that Quantstamp endorses the content on that web site or the operator or operations of that site. 
You are solely responsible for determining the extent to which you may use any content at any other web sites to which you link from the report. 
Quantstamp assumes no responsibility for the use of third-party software on the website and shall have no liability whatsoever to any person or entity for the accuracy or completeness of any outcome generated by such.

### Disclaimer

This report is based on the scope of materials and documentation provided for a limited review at the time provided. Results may not be complete nor inclusive of all vulnerabilities. 
The review and this report are provided on an as-is, where-is, and as-available basis. 
You agree that your access and/or use, including but not limited to any associated services, products, protocols, platforms, content, and materials, will be at your sole risk. 
Cryptographic tokens are emergent technologies and carry with them high levels of technical risk and uncertainty. 
The Solidity language itself and other smart contract languages remain under development and are subject to unknown risks and flaws. 
The review does not extend to the compiler layer, or any other areas beyond Solidity or the smart contract programming language, or other programming aspects that could present security risks. 
You may risk loss of tokens, Ether, and/or other loss. A report is not an endorsement (or other opinion) of any particular project or team, and the report does not guarantee the security of any particular project. 
A report does not consider, and should not be interpreted as considering or having any bearing on, the potential economics of a token, token sale or any other
product, service or other asset. No third party should rely on the reports in any way, including for the purpose of making any decisions to buy or sell any token, product, service or other asset.
To the fullest extent permitted by law, we disclaim all warranties, express or implied, in connection with this report, its content, and the related services and products and your use thereof, including,
without limitation, the implied warranties of merchantability, fitness for a particular purpose, and non-infringement.  
We do not warrant, endorse, guarantee, or assume responsibility for any product or service advertised or offered by a third party through the product, any open source or third party software, code,
libraries, materials, or information linked to, called by, referenced by or accessible through the report, its content, and the related services and products, any hyperlinked website, or any
website or mobile application featured in any banner or other advertising, and we will not be a party to or in any way be responsible for monitoring any transaction between you and any
third-party providers of products or services. As with the purchase or use of a product or service through any medium or in any environment, you should use your best judgment and exercise caution where appropriate. You may risk loss of QSP tokens or other loss.
FOR AVOIDANCE OF DOUBT, THE REPORT, ITS CONTENT, ACCESS, AND/OR USAGE THEREOF, INCLUDING ANY ASSOCIATED SERVICES OR MATERIALS, SHALL NOT BE CONSIDERED OR RELIED UPON AS ANY FORM OF FINANCIAL, INVESTMENT, TAX, LEGAL, REGULATORY, OR OTHER ADVICE.

