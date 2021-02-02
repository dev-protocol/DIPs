```
DIP: <# to be assigned>
Title: Fixed Maximum APY
Author(s): aggre
Contributors:
Type: Policy
Status: <Assigned by DIP Editor>
Date Proposed: 2021-02-02
Date Ratified: <date created on, in ISO 8601 (yyyy-mm-dd) format>
Dependencies: <DIP number(s)>
Replaces: <DIP number(s)>
License: CC0
```

## References

## Sentence Summary

Remove "number of assets" from the formula for inflation rate.

## Paragraph Summary

"Asset count" and "deposit amount" are used as variables to calculate the Dev Protocol's inflation rate.

Using these variables, the user-proposed Policy determines the inflation rate.

Removing "asset count" from those variables eliminates the risk that a new Market will be an attack on the protocol.

## Component Summary

- Remove "asset count" from `Policy.rewards` method's interface.
- The upper limit of APY is fixed.
- Users need new Policies to be proposed/voted to correct the decline in APY.

## Motivation

Increasing asset count works to increase APY, which reduces the disincentive (= risk that "asset increase" leads to "APY plunge") for creators.

However, the issues surrounding the Market scheme have become increasingly complex. And in some cases, the increase in APY can act as a disincentive for some creators, so using asset count as a variable can be seen as introducing too much complexity into the protocol and hindering upgradability.

## Specification

### Proposed Code

#### Policy Interface

```solidity
function rewards(uint256 _deposits) external view returns (uint256);
```

##### Actual

https://github.com/dev-protocol/protocol/blob/76e57682e7f9f2e375413b445189a38edc66ff97/contracts/interface/IPolicy.sol#L5-L8

#### Allocator Contract

```solidity
function calculateMaxRewardsPerBlock() external view returns (uint256) {
	return IPolicy(config().policy()).rewards(ILockup(config().lockup()).getAllValue());
}
```

##### Actual

https://github.com/dev-protocol/protocol/blob/76e57682e7f9f2e375413b445189a38edc66ff97/contracts/src/allocator/Allocator.sol#L25-L30

### Test Cases

```solidity
interface IPolicy {
	function rewards(uint256 _deposits) external view returns(uint256);
}

interface ILockup {
	function getAllValue() external view returns(uint256);
}

contract Policy is IPolicy {
	function rewards(uint256 _deposits) external view returns(uint256) {
		return _deposits * 2;
	}
}

contract Lockup is ILockup {
	function getAllValue() external view returns(uint256) {
		return 100000000000000000000;
	}
}
```

```ts
const policy: PolicyInstance;

describe("Policy.rewards", () => {
	it("Returns 2x the passed value", async () => {
		const result = await policy.rewards(100);
		expect(result).to.be(200);
	});
});
```

```ts
const allocator: AllocatorInstance;
const lockup: LockupInstance;
const policy: PolicyInstance;

describe("Allocator.calculateMaxRewardsPerBlock", () => {
	it("Returns the result when the argument of Policy.rewards is set to the value of Lockup.getAllValue", async () => {
		const result = await allocator.calculateMaxRewardsPerBlock();
		const allValue = await lockup.getAllValue();
		const expected = await policy.rewards(allValue);
		expect(result).to.be(expected);
	});
});
```

### Security Considerations

The obvious risk to the protocol is an "unexpected increase in the inflation rate," and this proposal eliminates that risk.

There is no additional technical risk associated with this proposal.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
