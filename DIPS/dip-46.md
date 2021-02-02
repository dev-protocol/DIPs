```
DIP: 46
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

Remove "asset count" from the formula for Dev Protocol's inflation rate.

## Paragraph Summary

"Asset count" and "deposit amount" are used as variables to calculate the Dev Protocol's inflation rate.

Using these variables, the user-proposed Policy determines the inflation rate.

Removing "asset count" from those variables eliminates the risk that a new Market will be an attack on the protocol.

## Component Summary

- Remove "asset count" from `Policy.rewards` method's interface.
- The inflation rate does not change even if asset count increases.
- Users need new Policies to be proposed/voted to correct the decline in APY.
- When this DIP is applied, the latest "inflation upper limit by old scheme" is used as the inflation upper limit's initial value.
  The inflation rate decreases only with increasing deposit (=staking). That inflation curve is similar to the validator rewards from deposits on the Ethereum 2.0 beacon chain.

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

#### Policy for DIP46

```solidity
contract DIP46 is DIP1 {
	uint256 private constant basis = 10000000000000000000000000;
	uint256 private constant power_basis = 10000000000;
	uint256 private constant mint_per_block = 120000000000000; // **SHOULD REWRITE WITH THE LATEST VALUE**

	constructor(address _config) public DIP1(_config) {}

	function rewards(uint256 _deposits)
		external
		view
		returns (uint256)
	{
		uint256 t = ERC20(config().token()).totalSupply();
		uint256 s = (_deposits.mul(basis)).div(t);
		uint256 max = mint_per_block;
		uint256 _d = basis.sub(s);
		uint256 _p =
			(
				(power_basis.mul(12)).sub(
					s.div((basis.div((power_basis.mul(10)))))
				)
			)
				.div(2);
		uint256 p = _p.div(power_basis);
		uint256 rp = p.add(1);
		uint256 f = _p.sub(p.mul(power_basis));
		uint256 d1 = _d;
		uint256 d2 = _d;
		for (uint256 i = 0; i < p; i++) {
			d1 = (d1.mul(_d)).div(basis);
		}
		for (uint256 i = 0; i < rp; i++) {
			d2 = (d2.mul(_d)).div(basis);
		}
		uint256 g = ((d1.sub(d2)).mul(f)).div(power_basis);
		uint256 d = d1.sub(g);
		uint256 mint = max.mul(d);
		mint = mint.div(basis).div(basis);
		return mint;
	}
}
```

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
