```
DIP: 38
Title: Creator Reward Withdrawal Limit
Author(s): @aggre
Contributors: Scott G <scott@devprotocol.xyz>
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

インターフェースを以下のように更新します。ストレージコントラクトに必要な永続化値を適宜追加します。

#### MetricsGroup Contract

認証済みプロパティの数を計算する関数を追加します。

| Method/Struct                  | Type | Spec                                                  |
| ------------------------------ | ---- | ----------------------------------------------------- |
| `totalAuthenticatedProperties` | New  | 認証済みの一意の数を返します。 |

#### Lockup Contract

ステーキングが増減するたびに幾何平均を再計算します。

最新の幾何平均は `geometricMeanLockedUp` 関数で参照できる。

_幾何平均を計算するためには大量の反復計算が不可欠であるため、オンチェーンで幾何平均を計算するためには大量のガスが消費される。したがって、オラクルは、有効な複雑さ削減計画が見つかるまで幾何平均を定期的に更新する。_

| Method/Struct                             | Type   | Spec                                                                                                                                                                                                                                                                                                                                                          |
| ----------------------------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `geometricMeanLockedUp`                   | New    | 最新の幾何平均数を返します。                                                                                                                                                                                                                                                                                                           |
| `lockup`                                  | Change | ステーキングアカウントが対象Propertyホルダーの場合は、「disabled lockedups」の格納値にステーキング数を加算します。                                                                                                                                                                                              |
| `withdraw`                                | Change | 「disabled lockedups」の格納値が0でない場合は、以下の式のように、その値からロックを解除する杭数を引いた値になります。`新disabled lockedups = (disabled lockedups) - (amount) <= (staking amount - amount) ? (disabled lockedups) : (disabled lockedups) - (amount)`                                                                                                                                                                                                    |
| `RewardPrices`                            | Change | 数値に「幾何平均ホルダープライス(geometric mean holders price)」を追加します。                                                                                                                                                                                                                                                                                                             |
| `calculateCumulativeRewardPrices`         | Change | 戻り値に「幾何平均ホルダープライス」を追加する。「幾何平均ホルダープライス」は `geometricMeanLockedUp` と `getStorageAllValue` の比率から計算される。                                                                                                                                                                                |
| `_calculateCumulativeHoldersRewardAmount` | Change | 引数に「幾何平均によるホルダープライス」を追加します。戻り値に「幾何平均によるキャップ」を追加します。幾何平均値によるキャップは、「幾何平均ホルダープライス」にステーキング数を乗じて算出する。計算で使用するステーキング数( `getStoragePropertyValue(PROPERTY)` の値)から "disabled lockedups"が差し引かれる。 |
| `calculateEffectiveHoldersRewardAmount`   | New    | `_calculateCumulativeHoldersRewardAmount` の新しい戻り値 "cap by geometric mean" を返します。                                                                                                                                                                                                                                                           |

#### Withdraw Contract

クリエイター報酬を出金する際には、幾何平均値で出金上限を超えないようにしてください。

| Method/Struct      | Type   | Spec                                                                                                                                                                                                                                                                                                                   |
| ------------------ | ------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `_calculateAmount` | Change | 元の引き出し可能額が「引き出し限度額」を超えた場合は、「引き出し限度額」を返します。引き出し限度額は、`Lockup.calculateEffectiveHoldersRewardAmount` の値を `Property.totalSupply` の値で割って、渡されたアカウントの残高を掛けて計算します。 |

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
