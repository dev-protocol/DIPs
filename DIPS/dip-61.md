```
DIP: 61
Title: Ability to delist rogue creators
Author(s): @Akira-Taniguchi
Contributors:
Type: Informational
Status: Proposed
Date Proposed: 2021-06-16
Date Ratified:
Dependencies: -
```

## DIP61: Ability to delist rogue creators

[不正なクリエイターの上場廃止機能(Ability to delist rogue creators.)](https://community.devprotocol.xyz/t/ability-to-delist-rogue-creators/258)

## Sentence Summary

This problem solves the problem of not being able to unauthenticate with the market except for the property author.

## Paragraph Summary

This DIP proposes a feature that allows the market owner to freely deauthenticate the property.

## Component Summary

Modify the onlyLinkedPropertyAuthor modifier.
Currently, an error occurs unless the sender is an author of the property.
This will be changed so that the error will not occur if the "sender is the author of the property or the owner of the market".

## Motivation

There have been cases where development repositories have been deleted or made private. We provide this feature to deal with this.

## Specification

### Proposed Code

I'll implement it now.

### Test Cases

・Only the market owner and the property's author have the authority to execute the market deauthentication function.
・Check that the following conditions are met for the deauthenticated property.
　・Staking cannot be performed.
　・Staking can be canceled.
　・Creator reward and staking reward cannot be obtained.

### Security Considerations

Make sure that only the owner and the author of the property to be deauthorized are authorized to execute the Market deauthorization function.
(Author-based deauthentication has already been implemented.)

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
