```
DIP:
Title: Add `onBeforeUpdate` hook
Author(s): aggre
Contributors:
Type: Enhancement
Status: <Assigned by DIP Editor>
Date Proposed: 2023-07-03
Date Ratified: <date created on, in ISO 8601 (yyyy-mm-dd) format>
Dependencies: [DIP-66](https://github.com/dev-protocol/DIPs/blob/main/DIPS/dip-66.md)
Replaces:
License: CC0
```

## References

- [DIP-66](https://github.com/dev-protocol/DIPs/blob/main/DIPS/dip-66.md)

## Sentence Summary

Add `onBeforeUpdate` hook to the lifecycle if sTokens to allow dynamic sTokens to be involved in its update flow.

## Paragraph Summary

`ITokenURIDescriptor.onBeforeUpdate` is a new _optional_ interface for TokenURIDescriptor (a.k.a. dynamic sTokens).

The function takes arguments similar to `ITokenURIDescriptor.onBeforeMint` already used, but also includes both the old and new staking positions.

When `ITokenURIDescriptor.onBeforeUpdate` returns `true`, the updating will success, and not when it returns `false`.

## Component Summary

- DIP-66 has been enhanced in its functionality through multiple updates.
- The most significant change was dynamic sTokens, which makes the `tokenURI` freely extensible by external contracts.
  - Users can attach a contract that extends the `ITokenURIDescriptor` interface by calling `STokensManager.setTokenURIDescriptor`.
- `ITokenURIDescriptor` already has `ITokenURIDescriptor.onBeforeMint`, but did not have one that could hook into the update flow of the staking positions.
- `ITokenURIDescriptor.onBeforeUpdate` is hooked every time staking positions are increased/decreased, allowing arbitrary side effects to be performed during the flow.

## Motivation

Allowing dynamic sTokens to be involved in its update flow after mint allows for more diverse use cases.

## Specification / Proposal Details

- `ITokenURIDescriptor.onBeforeUpdate` is a mutable function.
- `ITokenURIDescriptor.onBeforeUpdate` takes 5 arguments:
  1. `_tokenId`: Token ID
  2. `_owner` Owner address
  3. `_oldPositions`: Staking position before updated
  4. `_newPositions`: Staking position after updated
  5. `_payload`: Token payload
- `ITokenURIDescriptor.onBeforeUpdate` returns a boolean, `true` or `false`.

### Proposed Interface

```solidity
/*
 * @dev hooks and run a side-effect before updated
 * @param _tokenId token id
 * @param _owner owner address
 * @param _oldPositions staking position before updated
 * @param _newPositions staking position after updated
 * @param _payload token payload
 * @return bool success or failure
 */
function onBeforeUpdate(
    uint256 _tokenId,
    address _owner,
    ISTokensManager.StakingPositions memory _oldPositions,
    ISTokensManager.StakingPositions memory _newPositions,
    bytes32 _payload
) external returns (bool);
```

### Implementation Notes

`ITokenURIDescriptor.onBeforeUpdate` is optional, so the caller should be always use try-catch to allow it can catch an error.

```solidity
bool res;
try
    ITokenURIDescriptor(descriptor).onBeforeUpdate(/* Args... */)
returns (bool _res) {
    res = _res;
} catch {
    res = true;
}
```

## Use-cases

### Manage total locked for each payload

```solidity!
mapping(address => mapping(bytes32 => uint245)) totalLockedOf;

function onBeforeMint(
    uint256,
    address,
    ISTokensManager.StakingPositions memory _p,
    bytes32 _payload
) external returns (bool) {
    uint256 current = totalLockedOf[_p.property][_payload];
    totalLockedOf[_p.property][_payload] = current + _p.amount;
    return true;
}

function onBeforeUpdate(
    uint256,
    address,
    ISTokensManager.StakingPositions memory _old,
    ISTokensManager.StakingPositions memory _new,
    bytes32 _payload
) external returns (bool) {
    uint256 current = totalLockedOf[_old.property][_payload];
    totalLockedOf[_old.property][_payload] = current - _old.amount + _new.amount;
    return true;
}
```

### Time-locked positions

```solidity!
mapping(address => mapping(bytes32 => uint245)) lockedPeriodOf;
uint256 public durationSec = 2628000;

function onBeforeMint(
    uint256,
    address,
    ISTokensManager.StakingPositions memory _p,
    bytes32 _payload
) external returns (bool) {
    lockedPeriodOf[_p.property][_payload] = block.timestamp + durationSec;
    return true;
}

function onBeforeUpdate(
    uint256,
    address,
    ISTokensManager.StakingPositions memory _old,
    ISTokensManager.StakingPositions memory _new,
    bytes32 _payload
) external returns (bool) {
    bool valid = block.timestamp >= lockedPeriodOf[_p.property][_payload];
    return valid;
}
```

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
