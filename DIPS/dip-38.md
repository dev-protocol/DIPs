```
DIP: 38
Title: Creator Reward Withdrawal Limit
Author(s): @aggre
Contributors: Scott G <scott@devprotocol.xyz>
Type: Informational
Status: Draft
Date Proposed: 2021-01-02
Date Ratified: 2021-01-17
Dependencies: -
Replaces: -
```

## References

## Sentence Summary

This DIP solves the self-staking problem.

## Paragraph Summary

This DIP proposes to set a limit per block on a withdrawable amount of creator rewards. The limit is equal to the withdrawable amount assuming that the _"geometric mean of the number of stakings for all authenticated Properties"_ is staked.

## Component Summary

- Set a limit per block on a withdrawable amount of creator rewards.
  - The limit is equal to the withdrawable amount assuming that the _"geometric mean of the number of stakings for all authenticated Properties"_ is staked.
  - The limit functions only as like vesting, and there is no change in the total creator rewards for each property.
  - Even if a creator or property holder does self-staking to increase his/her creator rewards, he/she will not be able to get a large amount of creator rewards.
    - Because is that the geometric mean cannot be increased efficiently without increasing the number of stakings on many Properties.
  - An efficient way to increase creator rewards is to increase the limit by staking to someone else's Property instead of self-staking.
- Disabling staking by Property holders
  - Staking by a wallet account holding a "Property of staking destination" will not increase the creator reward for that Property.
  - It will be applied if you hold even 1 token (converted to the token decimals is 0.000000000000000001) of that Property.
    - If the formula reflects the percentage of equity in the token, the bigger holder may ask a smaller holder to do self-staking.
    - Only the number of Property holdings at the time of staking is used as the applicable condition.
    - It does not consider if the user holds the Property later or gives it away later.

## Motivation

The act of a creator staking on his Property is defined as "self-staking" here. Self-staking allows creators to earn both staking rewards and creator rewards. Self-staking acts like a tax to stakers, as that is not possible for someone who does not have their own Property. This interferes with the protocol neutrality.

Those who do not have Property can purchase Properties. Still, it is thought that a fundamental change in the reward system is necessary because the matter that creators continue to hold the majority of allocations will occur. (Besides, Property liquidity needs to be improved, but that is not the scope of this DIP)

## Specification / Proposal Details

### Limit per block

The withdrawal limit per block (`L`) can be calculated as follows:

- ![x](https://latex.codecogs.com/gif.latex?%5Cinline%20%5Cdpi%7B200%7D%20%5Cbg_white%20x_i) = Total staked amount for a Property
- `n` = The number of total authenticated Property
- `T` = Geometric mean of ![x](https://latex.codecogs.com/gif.latex?%5Cinline%20%5Cdpi%7B200%7D%20%5Cbg_white%20x_i)<br/>
  ![T = Geometric mean of x](https://latex.codecogs.com/gif.latex?%5Cdpi%7B240%7D%20%5Cbg_white%20%5Ctext%7BT%7D%3D%5Cleft%28%20%5Cprod_%7Bi%3D1%7D%5En%20x_i%5Cright%29%5E%7B%5Cfrac%7B1%7D%7Bn%7D%7D)
- `L` = Creator reward per block for staking `T` (Depends on the latest Policy)

If ![x](https://latex.codecogs.com/gif.latex?%5Cinline%20%5Cdpi%7B200%7D%20%5Cbg_white%20x_i) is `0`, it is treated as `1`. `1` is `0.000000000000000001` when converted to the token decimals.

### Simulation

The following table simulates how creator rewards are calculated when this DIP is applied and includes the **effective** APY assuming an APY of 25%.

As the table shows, intensive staking negatively affects creators by using the geometric mean. In other words, it often makes more sense to stake-to-others than to self-stake.

|                                                                         | Total Staked    | Total Staked'    | Total Staked''  |
| ----------------------------------------------------------------------- | --------------- | ---------------- | --------------- |
| Property A                                                              | 1               | 1                | 3               |
| Property B                                                              | 1               | 1                | 4               |
| Property C                                                              | 1               | 1                | 2000            |
| Property D                                                              | 1               | 1                | 2000            |
| Property E                                                              | 1               | 1                | 2000            |
| Property F                                                              | 1               | 1                | 2000            |
| Property G                                                              | 1               | 1                | 2000            |
| Property H                                                              | 10000           | 20000            | 10000           |
| Sum                                                                     | 10007           | 20007            | 20007           |
| **Geometric mean**                                                      | **3.16227766**  | **3.448488241**  | **498.9320504** |
| **Estimated max annually earn**<br/> when using the geometric mean      | **0.790569415** | **0.8621220603** | **124.7330126** |
| Arithmetic mean                                                         | 1250.875        | 2500.875         | 2500.875        |
| Estimated max annually earn<br/> when using the arithmetic mean         | 312.71875       | 625.21875        | 625.21875       |
| Estimated max annually earn<br/> using current formula (for Property H) | 2500            | 5000             | 2500            |

### Proposed Code Updating

Update the interfaces as follows. Add the required persistence values to the storage contracts as appropriate.

#### MetricsGroup Contract

Add a new function to calculate the number of authenticated Properties.

| Method/Struct                  | Type | Spec                                                  |
| ------------------------------ | ---- | ----------------------------------------------------- |
| `totalAuthenticatedProperties` | New  | Returns the number of authenticated unique Properties |

#### Lockup Contract

Recalculate the geometric mean each time the staking increases or decreases.

The latest geometric mean can be referenced by `geometricMeanLockedUp` function.

_Since a large amount of iterative calculation is indispensable to calculate the geometric mean, a large amount of gas is consumed to calculate the geometric mean with on-chain. Therefore, an oracle will update the geometric mean regularly until a valid complexity reduction plan is found._

| Method/Struct                             | Type   | Spec                                                                                                                                                                                                                                                                                                                                                          |
| ----------------------------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `geometricMeanLockedUp`                   | New    | Returns the latest geometric mean number of staked.                                                                                                                                                                                                                                                                                                           |
| `lockup`                                  | Change | If the staking account is the target Property holder, add the number of staking to the "disabled lockedups" stored value.                                                                                                                                                                                                                                     |
| `withdraw`                                | Change | If the "disabled lockedups" stored value is not zero, follows the below formula, subtract the number of staking to be unlocked from that value: `new disabled lockedups = (disabled lockedups) - (withdrawal amount) <= (staked amount - withdrawal amount) ? (disabled lockedups) : (disabled lockedups) - (withdrawal amount)`                               |
| `RewardPrices`                            | Change | Add "geometric mean holders price" to the values.                                                                                                                                                                                                                                                                                                             |
| `calculateCumulativeRewardPrices`         | Change | Add "geometric mean holders price" to the return value. The "geometric mean holders price" is calculated follows the below formula from the ratio of `geometricMeanLockedUp` and `getStorageAllValue`: `geometric mean holders price = geometricMeanLockedUp / getStorageAllValue * (holders price)`                                                          |
| `_calculateCumulativeHoldersRewardAmount` | Change | Add "geometric mean holders price" to the arguments. Add "cap by geometric mean" to the return value. The "cap by geometric mean" is calculated by multiplying the "geometric mean holders price" by the number of stakings. The number of stakings( value of `getStoragePropertyValue(PROPERTY)` ) uses in the calculation deducts the "disabled lockedups." |
| `calculateEffectiveHoldersRewardAmount`   | New    | Returns the new return value "cap by geometric mean" for `_calculateCumulativeHoldersRewardAmount`.                                                                                                                                                                                                                                                           |

#### Withdraw Contract

When withdrawing creator rewards, do not exceed the withdrawal limit by geometric mean.

| Method/Struct      | Type   | Spec                                                                                                                                                                                                                                                                                                                   |
| ------------------ | ------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `_calculateAmount` | Change | If the original withdrawable amount exceeds the "withdrawal limit," the "withdrawal limit" will be returned. The "withdrawal limit" is calculated by dividing the value of `Lockup.calculateEffectiveHoldersRewardAmount` by the value of `Property.totalSupply` and multiplying by the balance of the passed account. |

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
