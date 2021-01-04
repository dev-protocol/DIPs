```
DIP: <# to be assigned>
Title: Creator Reward Withdrawal Limit
Author(s): @aggre
Contributors: -
Type: Informational
Status: Draft
Date Proposed: 2021-01-02
Date Ratified: <date created on, in ISO 8601 (yyyy-mm-dd) format>
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

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).