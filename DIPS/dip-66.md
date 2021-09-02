```
DIP: 66
Title: sTokens, NFTs mirroring DEV staking
Author(s): aggre
Contributors:
Type: Enhancement
Status: <Assigned by DIP Editor>
Date Proposed: 2021-08-09
Date Ratified: <date created on, in ISO 8601 (yyyy-mm-dd) format>
Dependencies:
Replaces:
License: CC0
```

## References

- [[RFC] sTokens, a token mirroring DEV staking](https://community.devprotocol.xyz/t/rfc-stokens-a-token-mirroring-dev-staking/303/)

## Sentence Summary

A proposal for ERC-721 tokens mirroring DEV staking, called sTokens.

## Paragraph Summary

Users earn NFTs called sTokens, by staking DEV tokens.

sTokens stores the state of the staking.

sTokens improves staking composability.

## Component Summary

- **[Uniqueness]**
  sTokens is a unique NFT for each staking.
- **[Certificate of Staking]**
  A sTokens holder is considered to be the person who ran the staking stored in sTokens. If you transfer sTokens to another person, that right will also be transferred.

  sTokens stores the values of staking destination, staking amount, last reward unit price, cumulative reward amount, and pending reward amount into each NFT.

- **[NFT, not ERC-20]**
  By staking 10 DEV on a Property token, the staker gets 1 sTokens NFT. By staking 1,000 DEV on a Property token, the staker gets 1 sTokens NFT.
- **[Update]**
  sTokens change the state by withdrawing or adding staking. There is no burn.
- **[Bridge]**
  In some cases, ERC-20 is preferable to NFT. In that case, a user can extend the composability by creating a third-party bridge contract for NFT<>ERC-20.

## Motivation

DEV staking users can prove that they are staking by the function in the Lockup contract on Dev Protocol. However, the function is not a standardized interface such as ERC-721 or ERC-20, which creates an inconvenient situation for composability.

## Specification

sTokens are automatically issued upon staking and can be updated upon unstake.

- **[Creating sTokens]**
  sTokens is created as an NFT by STokensManager when users call `Lockup.deposit` and the Lockup contract calls STokensManager.
- **[Updating sTokens]**
  By users call the `Lockup.deposit` or `Lockup.withdraw`, STokensManager is called from the Lockup contract, data of existing sTokens will be updated by STokensManager.
- **[Deleting sTokens]**
  There is no deletions interface in sTokens. By the same procedure as updating sTokens, the amount of staking position is set to 0 to indicate that the position no longer exists.
- **[STokensManager]**
  A new contract to manage minting and updating sTokens, the deployed address by the local variables of Lockup Contract have been retained.

### Why not ERC-20?

Dev Protocol staking rewards change the rate of return at unpredictable timings as third parties increase or decrease staking. To implement that property, Dev Protocol handles a cumulative sum of the reward unit prices that continue to be increased and a snapshot of that cumulative sum at the time for each staking. The staking rewards are unique, and fungible tokens are inconvenient to retain that unique data. This is similar to why Uniswap v3's liquidity tokens are now NFTs instead of ERC-20.

### Proposed Code

#### STokensManager

ISTokensManager declares an interface for the STokensManager contract.

```solidity
interface ISTokensManager {
	/*
	 * @dev Event emitted when a token is updated.
	 * @param tokenId The ID of the updated staking position
	 * @param amount The new staking amount
	 * @param price The latest unit price of the cumulative staking reward
	 * This value equals the 3rd return value of the Lockup.calculateCumulativeRewardPrices
	 * @param cumulativeReward The cumulative withdrawn reward amount
	 * @param pendingReward The pending withdrawal reward amount amount
	 */
	event Updated(
		uint256 tokenId,
		uint256 amount,
		uint256 price,
		uint256 cumulativeReward,
		uint256 pendingReward
	);

	/*
	 * @dev Struct to declares a staking position.
	 * @param owner The address of the owner of the new staking position
	 * @param property The address of the Property as the staking destination
	 * @param amount The amount of the new staking position
	 * @param price The latest unit price of the cumulative staking reward
	 * @param cumulativeReward The cumulative withdrawn reward amount
	 * @param pendingReward The pending withdrawal reward amount amount
	 */
	struct StakingPosition {
		address owner;
		address property;
		uint256 amount;
		uint256 price;
		uint256 cumulativeReward;
		uint256 pendingReward;
	}

	/*
	 * @dev Struct to declares staking rewards.
	 * @param entireReward The reward amount of adding the cumulative withdrawn amount
	 to the withdrawable amount
	 * @param cumulativeReward The cumulative withdrawn reward amount
	 * @param withdrawableReward The withdrawable reward amount
	 */
	struct Rewards {
		uint256 entireReward;
		uint256 cumulativeReward;
		uint256 withdrawableReward;
	}

	/*
	 * @dev Struct to declares mint parameters.
	 * @param owner The address of the owner of the new staking position
	 * @param property The address of the Property as the staking destination
	 * @param amount The amount of the new staking position
	 * @param price The latest unit price of the cumulative staking reward
	 * This value should be equal to the 3rd return value of the Lockup.calculateCumulativeRewardPrices
	 */
	struct MintParams {
		address owner;
		address property;
		uint256 amount;
		uint256 price;
	}

	/*
	 * @dev Struct to declares update parameters.
	 * @param tokenId The ID of the staking position
	 * @param amount The new staking amount
	 * @param price The latest unit price of the cumulative staking reward
	 * This value equals the 3rd return value of the Lockup.calculateCumulativeRewardPrices
	 * @param cumulativeReward The cumulative withdrawn reward amount
	 * @param pendingReward The pending withdrawal reward amount amount
	 */
	struct UpdateParams {
		uint256 tokenId;
		uint256 amount;
		uint256 price;
		uint256 cumulativeReward;
		uint256 pendingReward;
	}

	/*
	 * @dev Creates the new staking position for the caller.
	 * Mint must be called from the Lockup contract.
	 * @param _params MintParams
	 * @return tokenId The ID of the created new staking position
	 * @return position The StakingPosition of the new staking position
	 */
	function mint(MintParams calldata _params) external returns(uint256 tokenId, StakingPosition position);

	/*
	 * @dev Updates the existing staking position.
	 * Update must be called from the Lockup contract.
	 * @param _params UpdateParams
	 * @return position The StakingPosition of the updated staking position
	 */
	function update(UpdateParams calldata _params) external returns(StakingPosition position);

	/*
	 * @dev Gets the existing staking position.
	 * @param _tokenId The ID of the staking position
	 * @return position The results of StakingPosition
	 */
	function positions(uint256 _tokenId) external view returns(StakingPosition position);

	/*
	 * @dev Gets the reward status of the staking position.
	 * @param _tokenId The ID of the staking position
	 * @return rewards The results of Rewards
	 */
	function rewards(uint256 _tokenId) external view returns(Rewards rewards);
}
```

Since it is not gas efficient to hold the `tokenURI` as a character string, define STokensDescriptor as follows and dynamically generate the URI string.

```solidity
contract STokensDescriptor {
	function getTokenURI(StakingPosition memory _position) internal pure returns (string memory) {
		string memory name = string(
			abi.encodePacked(
				'Dev Protocol sTokens - ',
				_position.property,
				' - ',
				_position.amount,
				' DEV',
				' - ',
				_position.cumulativeReward
			)
		);
		string memory description = string(
			abi.encodePacked(
				'This NFT represents a staking position in a Dev Protocol Property tokens. The owner of this NFT can modify or redeem the position.\nProperty Address: ',
				_position.property,
				'\n\n⚠️ DISCLAIMER: Due diligence is imperative when assessing this NFT. Make sure token addresses match the expected tokens, as token symbols may be imitated.'
			)
		);
		string memory image = string(
			abi.encodePacked(
				'<svg xmlns="http://www.w3.org/2000/svg" width="290" height="500" viewBox="0 0 290 500" fill="none"><rect width="290" height="500" fill="url(#paint0_linear)"/><path fill-rule="evenodd" clip-rule="evenodd" d="M192 203H168.5V226.5V250H145H121.5V226.5V203H98H74.5V226.5V250V273.5H51V297H74.5H98V273.5H121.5H145H168.5H192V250V226.5H215.5H239V203H215.5H192Z" fill="white"/><text fill="white" xml:space="preserve" style="white-space: pre" font-family="monospace" font-size="11" letter-spacing="0em"><tspan x="27.4072" y="333.418">',
				_position.property,
				'</tspan></text><defs><linearGradient id="paint0_linear" x1="0" y1="0" x2="290" y2="500" gradientUnits="userSpaceOnUse"><stop stop-color="#00D0FD"/><stop offset="0.151042" stop-color="#4889F5"/><stop offset="0.552083" stop-color="#D500E6"/><stop offset="1" stop-color="#FF3815"/></linearGradient></defs></svg>'
			)
		);
		return string(
			abi.encodePacked(
				'data:application/json;base64,',
				Base64.encode(
					bytes(
						abi.encodePacked(
							'{"name":"',
							name,
							'", "description":"',
							description,
							'", "image": "',
							'data:image/svg+xml;base64,',
							image,
							'"}'
						)
					)
				)
			)
		);
	}
}
```

STokensManager inherits the ERC-721 interfaces.

```solidity
contract STokensManager is ISTokensManager, STokensDescriptor, ERC721, ERC721Metadata {
	function tokenURI(uint256 _tokenId) external view returns (string){
		return getTokenURI(getStoragePositionsV1(_tokenId));
	}

	// Implementations...
}
```

#### Upgradability

STokensManager implements updatablity according to [EIP-1967](https://eips.ethereum.org/EIPS/eip-1967).

#### The NFT specific data of STokensManager is the below:

- **name:**
  `Dev Protocol sTokens V1`
- **symbol:**
  `DEV-STOKENS-V1`

#### Updating Lockup Contract

The concept of the changes for the Lockup Contract is shown below. These concepts cite existing source code but exclude all comments for readability.

Related to constructor:

```diff
+ ISTokensManager public STokensManager;

- constructor(address _config, address _devMinter)
+ constructor(address _config, address _devMinter, address _sTokensManager)
	public
	UsingConfig(_config)
 {
	devMinter = _devMinter;
+	STokensManager = ISTokensManager(_sTokensManager);
}
```

Related to deposit:

```diff
+ modifier onlyAuthenticatedProperty(address _property) {
+	require(
+		IMetricsGroup(config().metricsGroup()).hasAssets(_property),
+		"unable to stake to unauthenticated property"
+	);
+	_;
+ }

+ modifier onlyPositionOwner(uint256 _tokenId) {
+	require(
+		ISTokensManager.ownerOf(_tokenId) == msg.sender,
+		"illegal sender"
+	);
+	_;
+ }

+ function depositToProperty(address _property, uint256 _amount) external onlyAuthenticatedProperty(_property) returns(ISTokensManager.StakingPosition) {
+	RewardPrices memory prices = calculateCumulativeRewardPrices();
+	updateValues(true, _property, _amount, prices);
+	require(IDev(config().token()).transferFrom(msg.sender, _property, _amount), "dev transfer failed");
+ 	emit Lockedup(_from, _property, _value);
+	return STokensManager.mint(
+		ISTokensManager.MintParams(msg.sender, _property, _amount, prices.interest)
+	);
+ }

+ function depositToPosition(uint256 _tokenId, uint256 _amount) external onlyPositionOwner(_tokenId) onlyAuthenticatedProperty(_property) returns(ISTokensManager.StakingPosition) {
+	ISTokensManager.StakingPosition position = ISTokensManager.positions(_tokenId);
+	(uint256 withdrawable, RewardPrices memory prices) = _calculateWithdrawableInterestAmount(position);
+	updateValues(true, _property, _amount, prices);
+	require(IDev(config().token()).transferFrom(msg.sender, _property, _amount), "dev transfer failed");
+	uint256 nextAmount = position.amount.add(_amount);
+	uint256 cumulative = position.cumulativeReward.add(withdrawable);
+	uint256 pending = position.pendingReward.add(withdrawable);
+ 	emit Lockedup(_from, _property, _value);
+	return STokensManager.update(
+		ISTokensManager.UpdateParams(_tokenId, nextAmount, prices.interest, cumulative, pending)
+	);
+ }

function lockup(
	address _from,
	address _property,
	uint256 _value
) external {
	require(msg.sender == config().token(), "this is illegal address");
	require(_value != 0, "illegal lockup value");
	require(
		IMetricsGroup(config().metricsGroup()).hasAssets(_property),
		"unable to stake to unauthenticated property"
	);
	RewardPrices memory prices = updatePendingInterestWithdrawal(
		_property,
		_from
	);
-	updateValues(true, _from, _property, _value, prices);
+	updateValues4Legacy(true, _from, _property, _value, prices);
	emit Lockedup(_from, _property, _value);
}

function updatePendingInterestWithdrawal(address _property, address _user)
	private
	returns (RewardPrices memory _prices)
{
	(
		uint256 withdrawableAmount,
		RewardPrices memory prices
-	) = _calculateWithdrawableInterestAmount(_property, _user);
+	) = _calculateWithdrawableInterestAmount4Legacy(_property, _user);
	setStoragePendingInterestWithdrawal(
		_property,
		_user,
		withdrawableAmount
	);
	__updateLegacyWithdrawableInterestAmount(_property, _user);
	return prices;
}
```

Related to updateValues:

```diff
function beforeStakesChanged(
	address _property,
-	address _user,
	RewardPrices memory _prices
) private {
	uint256 cHoldersReward = _calculateCumulativeHoldersRewardAmount(
		_prices.holders,
		_property
	);
	if (
		getStorageLastCumulativeHoldersRewardPricePerProperty(_property) ==
		0 &&
		getStorageInitialCumulativeHoldersRewardCap(_property) == 0 &&
		getStoragePropertyValue(_property) == 0
	) {
		setStorageInitialCumulativeHoldersRewardCap(
			_property,
			_prices.holdersCap
		);
	}
-	setStorageLastStakedInterestPrice(_property, _user, _prices.interest);
	setStorageLastStakesChangedCumulativeReward(_prices.reward);
	setStorageLastCumulativeHoldersRewardPrice(_prices.holders);
	setStorageLastCumulativeInterestPrice(_prices.interest);
	setStorageLastCumulativeHoldersRewardAmountPerProperty(
		_property,
		cHoldersReward
	);
	setStorageLastCumulativeHoldersRewardPricePerProperty(
		_property,
		_prices.holders
	);
	setStorageCumulativeHoldersRewardCap(_prices.holdersCap);
	setStorageLastCumulativeHoldersPriceCap(_prices.holders);
}

function updateValues(
	bool _addition,
-	address _account,
	address _property,
	uint256 _value,
	RewardPrices memory _prices
) private {
-	beforeStakesChanged(_property, _account, _prices);
+	beforeStakesChanged(_property, _prices);
	if (_addition) {
		addAllValue(_value);
		addPropertyValue(_property, _value);
-		addValue(_property, _account, _value);
	} else {
		subAllValue(_value);
		subPropertyValue(_property, _value);
-		subValue(_property, _account, _value);
	}
	update();
}
```

Related to withdraw:

```diff
+ function withdrawByPosition(uint256 _tokenId, uint256 _amount) external onlyPositionOwner(_tokenId) returns(ISTokensManager.StakingPosition) {
+	ISTokensManager.StakingPosition position = ISTokensManager.positions(_tokenId);
+	require(
+		position.amount >= _amount,
+		"insufficient tokens staked"
+	);
+	(uint256 value, RewardPrices memory prices) = _withdrawInterest(position);
+	if (_amount != 0) {
+		IProperty(position.property).withdraw(msg.sender, _amount);
+	}
+	updateValues(false, position.property, _amount, prices);
+	uint256 cumulative = position.cumulativeReward.add(value);
+	return STokensManager.update(
+		ISTokensManager.UpdateParams(_tokenId, position.amount.sub(_amount), prices.interest, cumulative, 0)
+	);
+ }

+ function calculateWithdrawableInterestAmountByPosition(
+	uint256 _tokenId,
+	address _user
+ ) public view returns (uint256) {
+	ISTokensManager.StakingPosition position = ISTokensManager.positions(_tokenId);
+	(uint256 amount, ) = _calculateWithdrawableInterestAmount(_position);
+	return amount;
+ }


function withdraw(address _property, uint256 _amount) external {
	require(
		hasValue(_property, msg.sender, _amount),
		"insufficient tokens staked"
	);
-	RewardPrices memory prices = _withdrawInterest(_property);
+	RewardPrices memory prices = _withdrawInterest4Legacy(_property);
	if (_amount != 0) {
		IProperty(_property).withdraw(msg.sender, _amount);
	}
-	updateValues(false, msg.sender, _property, _amount, prices);
+	updateValues4Legacy(false, msg.sender, _property, _amount, prices);
}

- function _withdrawInterest(address _property)
+ function _withdrawInterest(ISTokensManager.StakingPosition calldata _position)
	private
	returns (uint256 _value, RewardPrices memory _prices)
{
	(
		uint256 value,
		RewardPrices memory prices
-	) = _calculateWithdrawableInterestAmount(_property, msg.sender);
+	) = _calculateWithdrawableInterestAmount(_position);
-	setStoragePendingInterestWithdrawal(_property, msg.sender, 0);
-	setStorageLastStakedInterestPrice(
-		_property,
-		msg.sender,
-		prices.interest
-	);
-	__updateLegacyWithdrawableInterestAmount(_property, msg.sender);
	require(
		IDevMinter(devMinter).mint(msg.sender, value),
		"dev mint failed"
	);
	update();
	return (value, prices);
}

function _calculateWithdrawableInterestAmount(
-	address _property,
-	address _user
+	ISTokensManager.StakingPosition calldata _position
) private view returns (uint256 _amount, RewardPrices memory _prices) {
	if (
-		IMetricsGroup(config().metricsGroup()).hasAssets(_property) == false
+		IMetricsGroup(config().metricsGroup()).hasAssets(_position.property) == false
	) {
		return (0, RewardPrices(0, 0, 0, 0));
	}
-	uint256 pending = getStoragePendingInterestWithdrawal(_property, _user);
+	uint256 pending = _position.pendingReward;
-	uint256 legacy = __legacyWithdrawableInterestAmount(_property, _user);
	(
		uint256 amount,
		,
		RewardPrices memory prices
-	) = _calculateInterestAmount(_property, _user);
+	) = _calculateInterestAmount(_position);
-	uint256 withdrawableAmount = amount.add(pending).add(legacy);
+	uint256 withdrawableAmount = amount.add(pending);
	return (withdrawableAmount, prices);
}

- function _calculateInterestAmount(address _property, address _user)
+ function _calculateInterestAmount(ISTokensManager.StakingPosition calldata _position)
	private
	view
	returns (
		uint256 _amount,
		uint256 _interestPrice,
		RewardPrices memory _prices
	)
{
-	uint256 lockedUpPerAccount = getStorageValue(_property, _user);
-	uint256 lastInterest = getStorageLastStakedInterestPrice(
-		_property,
-		_user
-	);
	(
		uint256 reward,
		uint256 holders,
		uint256 interest,
		uint256 holdersCap
	) = calculateCumulativeRewardPrices();
	uint256 result = interest >= _position.price
-		? interest.sub(lastInterest).mul(lockedUpPerAccount).divBasis()
+		? interest.sub(_position.price).mul(_position.amount).divBasis()
		: 0;
	return (
		result,
		interest,
		RewardPrices(reward, holders, interest, holdersCap)
	);
}
```

Legacy methods:
_Legacy methods and the latest method use common storage, so it will be implemented on the same contract to prioritize the gas efficiency._

```diff
+ function updateValues4Legacy(
+	bool _addition,
+	address _account,
+	address _property,
+	uint256 _value,
+	RewardPrices memory _prices
+ ) private {
+	setStorageLastStakedInterestPrice(_property, _account, _prices.interest);
+	updateValues(_addition, _property, _value, _prices);
+	if (_addition) {
+		addValue(_property, _account, _value);
+	} else {
+		subValue(_property, _account, _value);
+	}
+ }

+ function _withdrawInterest4Legacy(address _property)
+	private
+	returns (RewardPrices memory _prices)
+ {
+	(
+		uint256 value,
+		RewardPrices memory prices
+	) = _calculateWithdrawableInterestAmount4Legacy(_property, msg.sender);
+	setStoragePendingInterestWithdrawal(_property, msg.sender, 0);
+	setStorageLastStakedInterestPrice(
+		_property,
+		msg.sender,
+		prices.interest
+	);
+	__updateLegacyWithdrawableInterestAmount(_property, msg.sender);
+	require(
+		IDevMinter(devMinter).mint(msg.sender, value),
+		"dev mint failed"
+	);
+	update();
+	return prices;
+ }

+ function _calculateWithdrawableInterestAmount4Legacy(
+	address _property,
+	address _user
+ ) private view returns (uint256 _amount, RewardPrices memory _prices) {
+	if (
+		IMetricsGroup(config().metricsGroup()).hasAssets(_property) == false
+	) {
+		return (0, RewardPrices(0, 0, 0, 0));
+	}
+	uint256 pending = getStoragePendingInterestWithdrawal(_property, _user);
+	uint256 legacy = __legacyWithdrawableInterestAmount(_property, _user);
+	(
+		uint256 amount,
+		,
+		RewardPrices memory prices
+	) = _calculateInterestAmount4Legacy(_property, _user);
+	uint256 withdrawableAmount = amount.add(pending).add(legacy);
+	return (withdrawableAmount, prices);
+ }

+ function _calculateInterestAmount4Legacy(address _property, address _user)
+	private
+	view
+	returns (
+		uint256 _amount,
+		uint256 _interestPrice,
+		RewardPrices memory _prices
+	)
+ {
+	uint256 lockedUpPerAccount = getStorageValue(_property, _user);
+	uint256 lastInterest = getStorageLastStakedInterestPrice(
+		_property,
+		_user
+	);
+	(
+		uint256 reward,
+		uint256 holders,
+		uint256 interest,
+		uint256 holdersCap
+	) = calculateCumulativeRewardPrices();
+	uint256 result = interest >= lastInterest
+		? interest.sub(lastInterest).mul(lockedUpPerAccount).divBasis()
+		: 0;
+	return (
+		result,
+		interest,
+		RewardPrices(reward, holders, interest, holdersCap)
+	);
+ }

```

Related to migration:

```diff
+ function migrateToSTokens(address _property) external returns(ISTokensManager.StakingPosition) {
+	uint256 amount = getStorageValue(_property, msg.sender);
+	require(amount > 0, "not staked");
+	setStoragePendingInterestWithdrawal(
+		_property,
+		msg.sender,
+		0
+	);
+	setStorageValue(_property, msg.sender, 0);
+	uint256 price = getStorageLastStakedInterestPrice(
+		_property,
+		msg.sender
+	);
+	uint256 pending = getStoragePendingInterestWithdrawal(
+		_property,
+		msg.sender
+	);
+	(
+		uint256 tokenId,
+		ISTokensManager.StakingPosition memory position
+	) = STokensManager.mint(
+		ISTokensManager.MintParams(msg.sender, _property, amount, price)
+	);
+	rerurn STokensManager.update(
+		ISTokensManager.UpdateParams(tokenId, amount, price, amount, pending)
+	);
+ }
```

### Test Cases

#### STokensManager

##### mint

- Should fail to call when the sender is not Lockup
- Creates a new sTokens
- Should success to call even the amount is 0

##### update

- Should fail to call when the sender is not Lockup
- Updates a new sTokens
- Should success to call even the amount is 0

#### Lockup Contract

Due to backward compatibility, all existing test cases do not need to be modified except for construction and so on.

##### deposit

- If the first argument type is address
  - Should fail to call when the first argument is unauthenticated Property tokens
  - Should fail to call when the sender's DEV amount is insufficient
  - Should fail to call when the sender's DEV amount is sufficient but allowance is insufficient
  - Creates a new sTokens
- If the first argument type is uint256
  - Should fail to call when the position linked Proerty tokens is unauthenticated
  - Should fail to call when the sender is not the position owner
  - Should fail to call when the sender's DEV amount is insufficient
  - Should fail to call when the sender's DEV amount is sufficient but allowance is insufficient
  - Updates the existing sTokens

##### withdraw

- If the first argument type is address
  - _No changes required_
- If the first argument type is uint256
  - Should fail to call when the sender is not the position owner
  - Should failt to call when the passed amount is greater than staked amount
  - Updates the existing sTokens

### Security Considerations

Security evaluation is skipped because sTokens extends the traditional specification and does not involve any new external assets.

In addition, test cases ensure backward compatibility with traditional specifications that do not use sTokens.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
