# Baba Marta Audit Report

Prepared by: Sammy0101 (AlgorinthLabs)
Lead Auditors:

- Sammy0101

Assisting Auditors:

- None

# Table of contents

<details>

<summary>See table</summary>

- [Baba Marta Audit Report](#passwordstore-audit-report)
- [Table of contents](#table-of-contents)
- [About Sammy0101 and Disclaimer](#about-sammy0101-and-disclaimer)
- [Disclaimer](#disclaimer)
- [Risk Classification](#risk-classification)
- [Audit Details](#audit-details)
  - [Scope](#scope)
- [Protocol Summary](#protocol-summary)
  - [Roles](#roles)
- [Executive Summary](#executive-summary)
  - [Issues found](#issues-found)
- [Findings](#findings)
  - [High Risk Findings](#high)
    - [\[H-1\] `joinEvent::MartenitsaEvent` does not update state variables before external calls, making it susceptible to reentrancy attacks](#h-1-joinEvent-is-susceptible-to-reentracy-attacks-since-the-state-variables-are-not-updated-before-external-calls)
    - [\[H-2\] `buyMartenitsa::MartenitsaMarketplace` may be vulnerable to reentrancy attacks as external calls are made before updating state variables](#h-2-buyMartenitsa-is-susceptible-to-reentracy-attacks-as-external-calls-are-made-before-updating-state-variables)
    - [\[H-3\] `announceWinner::MartenitsaVoting` may be vulnerable to potential integer overflow](#h-3-potential-integer-overflow-vulnerability)
- [Medium Risk Findings](#medium-risk-findings)
- [\[M-1\] `distributeHealthToken::HealthToken` does not check the return value of the transferFrom function, which could result in a loss of tokens if the transfer fails](#m-1-distribute-health-tokens-could-be-lost-if-the-transfer-fails)
  [Low Risk Findings](#low-risk-findings)
- [\[L-1\] `requiredMartenitsaTokens::MartenitsaMarketplace` is declared but not used](#l-1-is-declared-but-not-used-in-the-contract)
  - [Relevant GitHub Links](#relevant-github-links)
- [Summary](#summary)
- [Vulnerability Details](#vulnerability-details)
- [Impact](#impact)
- [Tools Used](#tools-used)
</details>
</br>

# About Sammy0101 and Disclaimer

Sammy0101, on behalf of the Algorinth Labs LLC team, carried out this audit as a personal initiative within a competition. The team invested substantial time and effort to uncover potential vulnerabilities within the code within the allotted timeframe. It's crucial to clarify that while this report presents our findings, neither Sammy0101 nor Algorinth Labs LLC assume any liability for the results. Additionally, it's important to understand that our audit does not signify an endorsement of the underlying business or product. This assessment was time-limited and focused specifically on evaluating the security aspects of the Solidity implementation within the contracts.

# Risk Classification

|            |        | Impact |        |     |
| ---------- | ------ | ------ | ------ | --- |
|            |        | High   | Medium | Low |
|            | High   | H      | H/M    | M   |
| Likelihood | Medium | H/M    | M      | M/L |
|            | Low    | M      | M/L    | L   |

## Scope

```
src/
--- HealthToken.sol
--- MartenitsaEvent.sol
--- MartenitsaMarketplace.sol
--- MartenitsaToken.sol
--- MartenitsaVoting.sol
```

# Protocol Summary

The "Baba Marta" protocol offers users the opportunity to purchase and gift `MartenitsaTokens` to friends. Additionally, users can become producers, enabling them to create and sell `MartenitsaTokens`. There's also a competition where producers can submit their `MartenitsaTokens` for voting, with non-producers having the ability to vote. The winning `MartenitsaToken` earns its producer a `HealthToken` . If a user isn't a producer but desires a `HealthToken` , they can obtain one by accumulating three different `MartenitsaTokens`. The more `MartenitsaTokens` a user collects, the more `HealthTokens` they receive. `HealthTokens` serve as tickets to an exclusive event where participants assume the producer role, enabling them to create and sell their own `MartenitsaTokens` during the event.

## Issues found

| Severity          | Number of issues found |
| ----------------- | ---------------------- |
| High              | 3                      |
| Medium            | 1                      |
| Low               | 1                      |
| Info              | 1                      |
| Gas Optimizations | 0                      |
| Total             | 6                      |

# Findings

## High

### [H-1] `joinEvent::MartenitsaEvent` is susceptible to reentracy attacks since the state variables are not updated before external calls

**Description:** This report identifies a critical vulnerability with the `joinEvent` function in the `MartenitsaEvent` contract and there are several issues associated with it.

Overview: The `joinEvent` function allows users to participate in an event managed by the `MartenitsaEvent contract`. Participants are added to the event if they meet certain criteria, such as not being a producer and having a sufficient balance of `HealthToken`.

Issues: However, the function has several issues that need to be addressed:

Reentrancy Vulnerability: The `joinEvent` function allows external calls before completing internal state changes, which could potentially lead to reentrancy attacks.

Lack of Check for Event Participation: There is no check to verify if the event is currently active or if the caller has already joined the event. Without these checks, participants may be able to join the event multiple times or after it has ended.

Inefficient Participant Tracking: The method of tracking participants using the `_participants` mapping may not be efficient for certain operations, such as iterating over all participants or verifying if a given address is a participant.

Potential Race Condition: If multiple transactions attempt to join the event concurrently, there may be a race condition where the state is updated inconsistently.

**Impact:** These issues could undermine the security, reliability, and usability of the `MartenitsaEvent` contract, potentially resulting in financial losses, user dissatisfaction, and reputational damage to the project or platform deploying the contract. Therefore, it's crucial to address these issues promptly to mitigate their impact and ensure the integrity and robustness of the contract.
**Proof of Concept:** A malicious contract could exploit the reentrancy vulnerability in the `joinEvent` function to repeatedly drain funds or manipulate the state of the `MartenitsaEvent` contract

1.To provide a proof of concept for the potential reentrancy vulnerability in the `joinEvent` function of the `MartenitsaEvent` contract, we can create a malicious contract that exploits this vulnerability. Below is a simplified example demonstrating how an attacker could exploit reentrancy to drain funds from the MartenitsaEvent contract:

```javascript

 // Malicious contract to exploit reentrancy
contract MaliciousContract {
    MartenitsaEvent private _targetContract;

    constructor(address targetAddress) {
        _targetContract = MartenitsaEvent(targetAddress);
    }

    // Function to perform the reentrancy attack
    function attack() external payable {
        // Call joinEvent in a loop to repeatedly trigger the reentrancy
        for (uint256 i = 0; i < 10; i++) {
            _targetContract.joinEvent();
        }
    }

    // Callback function to be called by the target contract during reentrancy
    function receiveTokens() external payable {
        // Do nothing or perform further malicious actions
    }
}


```

**Recommended Mitigation:** To address these issues, you can consider the following improvements:

-Implement the checks-effects-interactions pattern to ensure that internal state changes are completed before interacting with external contracts or calling external functions.
-Add checks to verify if the event is active and if the caller has already joined the event.
-Consider using a data structure like a dynamic array or a mapping to efficiently track participants and prevent duplicate entries.
-Use synchronization mechanisms such as mutex locks or reentrancy guards to prevent race conditions in concurrent transactions.

Here's a modified version of the joinEvent function addressing these issues:

```javascript
function joinEvent() external {
    require(block.timestamp < eventEndTime, "Event has ended");
    require(!_participants[msg.sender], "You have already joined the event");
    require(!isProducer[msg.sender], "Producers are not allowed to participate");
    require(_healthToken.balanceOf(msg.sender) >= healthTokenRequirement, "Insufficient HealthToken balance");

    _participants[msg.sender] = true;
    participants.push(msg.sender);
    emit ParticipantJoined(msg.sender);

    // Transfer health tokens to the contract
    require(_healthToken.transferFrom(msg.sender, address(this), healthTokenRequirement), "HealthToken transfer failed");

    _addProducer(msg.sender);
}

```

### [H-2] `buyMartenitsa::MartenitsaMarketplace` is susceptible to reentracy attacks as external calls are made before updating state variables

**Description:** The `buyMartenitsa` function, reentrancy can occur due to the external call to seller.call{value: salePrice}(""). While the function seems straightforward, reentrancy vulnerabilities often arise from unexpected interactions with external contracts

```javascript
   function buyMartenitsa(uint256 tokenId) external payable {
        Listing memory listing = tokenIdToListing[tokenId];
        require(listing.forSale, "Token is not listed for sale");
        require(msg.value >= listing.price, "Insufficient funds");

        address seller = listing.seller;
        address buyer = msg.sender;
        uint256 salePrice = listing.price;

        martenitsaToken.updateCountMartenitsaTokensOwner(buyer, "add");
        martenitsaToken.updateCountMartenitsaTokensOwner(seller, "sub");

         // Clear the listing
        delete tokenIdToListing[tokenId];

        emit MartenitsaSold(tokenId, buyer, salePrice);

        // Transfer funds to seller
        (bool sent, ) = seller.call{value: salePrice}("");
        require(sent, "Failed to send Ether");

        // Transfer the token to the buyer
        martenitsaToken.safeTransferFrom(seller, buyer, tokenId);
    }
```

**Impact:** The `buyMartenitsa` function extends beyond financial losses to encompass token ownership manipulation, service disruptions, data integrity risks, and loss of user trust

**Recommended Mitigation:** To mitigate this reentrancy vulnerability, we can follow the checks-effects-interactions pattern by refactoring the code in this way:

```javascript

function buyMartenitsa(uint256 tokenId) external payable {
    Listing storage listing = tokenIdToListing[tokenId];
    require(listing.forSale, "Token is not listed for sale");
    require(msg.value >= listing.price, "Insufficient funds");

    // Prevent re-entrancy by updating state before any external call
    address seller = listing.seller;
    address buyer = msg.sender;
    uint256 salePrice = listing.price;

    // Clear the listing
    delete tokenIdToListing[tokenId];

    // Transfer the token to the buyer first
    martenitsaToken.safeTransferFrom(seller, buyer, tokenId);

    // Update token counts after the transfer
    martenitsaToken.updateCountMartenitsaTokensOwner(buyer, "add");
    martenitsaToken.updateCountMartenitsaTokensOwner(seller, "sub");

    // Transfer funds to seller after all state changes
    (bool sent, ) = seller.call{value: salePrice}("");
    require(sent, "Failed to send Ether");

    // Emit event after all actions are completed
    emit MartenitsaSold(tokenId, buyer, salePrice);
}

```

### [H-3] `announceWinner::MartenitsaVoting` may be vulnerable to potential integer overflow

**Description:** The potential integer overflow vulnerability in the `announceWinner` function arises from the calculation of the maximum number of votes. If there is a large number of tokens with a significant number of votes, the variable `maxVotes` might exceed the maximum value that can be represented by a uint256, leading to an overflow

```javascript
   function announceWinner() external onlyOwner {
        require(block.timestamp >= startVoteTime + duration, "The voting is active");

        uint256 winnerTokenId;
        uint256 maxVotes = 0;

        for (uint256 i = 0; i < _tokenIds.length; i++) {
            if (voteCounts[_tokenIds[i]] > maxVotes) {
                maxVotes = voteCounts[_tokenIds[i]];
                winnerTokenId = _tokenIds[i];
            }
        }

        list = _martenitsaMarketplace.getListing(winnerTokenId);
        _healthToken.distributeHealthToken(list.seller, 1);

        emit WinnerAnnounced(winnerTokenId, list.seller);
    }
```

**Impact:** The `announceWinner` The impact of the potential integer overflow vulnerability in the `announceWinner` function can be significant if exploited. This can lead to incorrect winner determination, unintended reward distribution, loss of trust and reputation and operational disruption.

**Recommended Mitigation:** To address this vulnerability, we should ensure that the calculation of `maxVotes` is performed safely and does not result in an overflow. We can achieve this by using proper input validation and implementing safe arithmetic operations. Here's how we can modify the announceWinner function to mitigate the integer overflow vulnerability:

```javascript

function announceWinner() external onlyOwner {
    require(block.timestamp >= startVoteTime + duration, "The voting is active");

    uint256 winnerTokenId;
    uint256 maxVotes = 0;

    for (uint256 i = 0; i < _tokenIds.length; i++) {
        // Use SafeMath to ensure safe addition
        maxVotes = maxVotes.add(voteCounts[_tokenIds[i]]);
    }

    // Iterate again to find the token with the maximum votes
    for (uint256 i = 0; i < _tokenIds.length; i++) {
        if (voteCounts[_tokenIds[i]] == maxVotes) {
            winnerTokenId = _tokenIds[i];
            break; // Stop searching after finding the first token with max votes
        }
    }

    // Ensure winnerTokenId is not zero before proceeding
    require(winnerTokenId != 0, "No winner found");

    list = _martenitsaMarketplace.getListing(winnerTokenId);
    _healthToken.distributeHealthToken(list.seller, 1);

    emit WinnerAnnounced(winnerTokenId, list.seller);
}



# Medium Risk Findings

## <a id='M-01'></a>M-01 `distributeHealthToken::HealthToken` distribute health tokens could be lost if the transfer fails.

## Summary

The `distributeHealthToken` function does not check the return value of the `transferFrom` function, which could result in a loss of tokens if the transfer fails.

## Recommendations

Check the return value of `transferFrom` and handle failures appropriately to prevent potential loss of funds.






# Low Risk Findings

## <a id='L-01'></a>L-01 `requiredMartenitsaTokens::MartenitsaMarketplace` is declared but not used in the contract
## Summary

Some variables are declared but not used in the contracts. For example, the `requiredMartenitsaTokens` variable in the `MartenitsaMarketplace` contract is declared but not used.

## Recommendations

Remove unused variables to enhance code cleanliness and efficiency



## Tools Used

No automated analysis tools were utilized to identify this issue. The vulnerability was discovered through manual inspection of the contract code.

### [I-1]
```
