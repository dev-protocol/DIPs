```
DIP: 3
Title: Decrease lock-up blocks in temporary
Author(s): Aggre (@aggre)
Contributors:
Type: Policy
Status: Finished
Date Proposed: 2020-6-3
Date Ratified: 2020-6-4
Dependencies: n/a
Replaces: [DIP1](https://github.com/dev-protocol/protocol/blob/3b224068182a9b88113e11d577d8632411c5a63b/contracts/src/policy/DIP1.sol)
```

## References

## Sentence Summary

Change `lockUpBlocks` to 1

## Paragraph Summary

Reduce the lock-up blocks for stakers from 175200 to 1.

## Component Summary

- **Update the DIP1**: Change `lockUpBlocks` to 1

## Motivation

Dev Protocol's user-friendly Dapp is not yet complete. In the meantime, it would be hard to ask users for a long-term lock-up. Set the lock-up block to 1 so that it can be unlocked in one block (about 15 seconds).

We believe that this should revert to a long-term lock-up in the future. We believe that if it works properly, the ecosystem will be more reliable.

## Specification / Proposal Details

As mentioned above.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
