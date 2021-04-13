```
DIP:
Title: Alternative DIP38: Withdrawal Limit For Creator Rewards Based On Liquidity Pools
Author(s): @aggre
Contributors:
Type: Informational
Status: Proposed
Date Proposed: 2021-04-13
Date Ratified:
Dependencies: -
Replaces: DIP38
```

# Alternative DIP38: Withdrawal Limit For Creator Rewards Based On Liquidity Pools

This proposal makes up for the shortcomings of DIP38.

The withdrawal limit for creator rewards in DIP38 is tiny. The current geometric mean staking number is about `1.3`, so DIP38 had the problem that the withdrawal limit for creator rewards was significantly smaller than expected.

This proposal corrects the withdrawal limit for creator rewards and fluctuates in line with the value of DEV pooled in the liquidity pools and staking flatness.

## Sentence Summary

This DIP solves the self-staking problem.

## Paragraph Summary

This DIP proposes to set a limit per block on a withdrawable amount for creator rewards. The limit is equal to the withdrawable amount assuming that
_"the value of multiplied the balance of DEV in the liquidity pools of the exchange protocols by the following values; 12, flatness, and the value of subtracted the ratio of DEV to ETH in the liquidity pools from 1"_ is staked.

The liquidity pool used as the basis for the calculation will be the DEV:ETH pool on Uniswap, but this selection process can be managed by governance in the future.

- [0x4168CEF0fCa0774176632d86bA26553E3B9cF59d](https://etherscan.io/address/0x4168cef0fca0774176632d86ba26553e3b9cf59d)

## Component Summary

- Set a limit per block on a withdrawable amount for creator rewards.
  - The limit is equal to the withdrawable amount assuming that _"the value of multiplied the balance of DEV in the liquidity pools of the exchange protocols by the following values; 12, flatness, and the value of subtracted the ratio of DEV to ETH in the liquidity pools from 1"_ is staked.
  - The limit functions only as like vesting, and there is no change in the total creator rewards for each property.
  - Even if a creator or property holder does self-staking to increase his/her creator rewards, he/she will not be able to get a large amount of creator rewards.
  - An efficient way to increase creator rewards is to increase the limit by staking to someone else's Property instead of self-staking.
  - The limit decreases as the DEV value in the liquidity pools increases and increases as the staking flatness increases.
- The liquidity pool used as the basis for the calculation.
  - The initial liquidity pool will be the [DEV:ETH pool on Uniswap](https://etherscan.io/address/0x4168cef0fca0774176632d86ba26553e3b9cf59d).
  - In the future, governance may allow the selection of liquidity pools.

## Motivation

The act of a creator staking on his Property is defined as "self-staking" here. Self-staking allows creators to earn both staking rewards and creator rewards. Self-staking acts like a tax to stakers, as that is not possible for someone who does not have their own Property. This interferes with the protocol neutrality.

Those who do not have Property can purchase Properties. Still, it is thought that a fundamental change in the reward system is necessary because the matter that creators continue to hold the majority of allocations will occur. (Besides, Property liquidity needs to be improved, but that is not the scope of this DIP)

## Specification / Proposal Details

### Limit per block

The withdrawal limit per block (`L`) can be calculated as follows:

- `D` = Pooled DEV on the liquidity pools
- `E` = Pooled ETH (or WETH) on the liquidity pools
- `GM` = Geometric mean of staking for all authenticated Properties with 0 as 1
- `AM` = Arithmetic mean of staking for all authenticated Properties
- `T` = `D*(1-(E/D))*12*GM/AM`
- `L` = Creator reward per block for staked `T` (Depends on the latest Policy)

### Simulation

#### Simulation based on the situation at the time of proposal

| Pooled Token | Balance |
| ------------ | ------- |
| DEV          | 111,969 |
| WETH         | 571     |

The `GM` (geometric mean of staking for all authenticated Properties with 0 as 1) is `1.28583`, the `AM` (arithmetic mean of staking for all authenticated Properties) is `252.69502`. Assume that APY for creators remains unchanged.

Results:

```
T = 111969*(1-(571/111969))*12*1.28583/252.69502 = 6802.13913
ANNUAL_L = 6802.13913*0.3929 = 2672.560464177
```

#### If DEV value is high

| Pooled Token | Balance |
| ------------ | ------- |
| DEV          | 58,000  |
| WETH         | 1200    |

Assume that GM, AM, and APY for creators remain unchanged.

Results:

```
T = 58000*(1-(1200/58000))*12*1.28583/252.69502 = 3468.29837
L = 3468.29837*0.3929 = 1362.694429573
```

#### Further, if staking flatness is high

| Pooled Token | Balance |
| ------------ | ------- |
| DEV          | 58,000  |
| WETH         | 1200    |

The `GM` (geometric mean of staking for all authenticated Properties with 0 as 1) is `2.5`, the `AM` (arithmetic mean of staking for all authenticated Properties) is `252.69502`. Assume that APY for creators remains unchanged.

Results:

```
T = 58000*(1-(1200/58000))*12*2.5/252.69502 = 6743.30661
L = 6743.30661*0.3929 = 2649.445167069
```

### Proposed Code Updating

Since a large amount of iterative calculation is indispensable to calculate the geometric mean, a large amount of gas is consumed to calculate the geometric mean with on-chain. Therefore, an oracle will update the geometric mean regularly until a valid complexity reduction plan is found.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
