```
DIP: <# to be assigned>
Title: Sponsor Program
Author(s): Scott G <scott@devprotocol.xyz>
Contributors: aggre (@aggre)
Type: Enhancement
Status: <Assigned by DIP Editor>
Date Proposed: 2021-01-01
Date Ratified: <date created on, in ISO 8601 (yyyy-mm-dd) format>
Dependencies:
Replaces:
```

## References

## Sentence Summary

Allow users to lock DEV tokens via staking for OSS projects not yet onboarded. If OSS projects onboard then users and projects get boosted rewards.

## Paragraph Summary

The Sponsor Program allows Dev Protocol users to lock up their DEV for 60-90 days by staking for OSS projects not yet onboarded. Sponsors’ rewards begin to accumulate as soon as they stake, but are locked until the OSS project tokenizes. If the OSS project doesn’t onboard then rewards are burned and users receive their DEV back.

## Component Summary

1. Escrow- Users deposit (stake) DEV in the pool which is locked until 60-90 days or the OSS project authenticates. Allocated DEV is also held in escrow.
2. Boosted Rewards' APY Calculation-Geyser logic based on DEV allocated to Sponsor Program which rewards users based on time and liquidity provided

## Motivation

The motivation behind this proposal is to allow the Dev Protocol community to onboard projects they're passionate about, show OSS projects that they have patrons on Dev Protocol, and reward all participants for working together.

## Specification / Proposal Details

### Preliminary Considerations

The following questions were considered before drawing my proposal:

1. On-chain or off-chain rewards or a mix? (DEV Inflation + Boosted Rewards or Boosted Rewards)
2. How are Boosted Rewards determined? (Github metrics or Manually determined)
3. How are Boosted Rewards APY determined? (Geyser)
4. How are Sponsored Projects determined? (Community vote or Database of pre-chosen projects)
5. Sponsored Program format? (New round of projects every X days or users can Sponsor a Project of their choice at any time)
6. How long should a Sponsored Program run? (30, 60, 90, 120 days)
7. How is Dev Protocol APY affected? (APY is lowered if users stake in Sponsor Program)
8. Do Sponsor Program's increase APY? (Not until they're authenticated)
9. What is the Sponsor Program's APY formula for inflationary DEV rewards?

### Specification Proposal

#### Inflationary Rewards + Boosted Rewards (Geyser)

1. Community vote every 90 days to add 5 projects to Sponsor Program
2. Team decides Boosted Reward allocation for each project
3. Users stake DEV in the Escrow which holds the rewards
4. Inflationary + Boosted Rewards begin to accumulate
5. Geyser's logic is used to distribute Boosted Rewards based on time and liquidity provided
6. If an OSS project authenticates then rewards are unlocked. If project doesn't authenticate by X days then allocated DEV for Boosted Rewards are kept in escrow, and inflationary rewards are burned.

#### Questions Remaining

1. How to determine Boosted Reward (DEV allocation) for each Sponsored OSS?<br/>
   a) Chosen by team<br/>
   b) Set upper and lower bounds for DEV allocation which is determined by Github metrics

2. How are Sponsor Program projects chosen?<br/>
   a) Users can choose to Sponsor at anytime<br/>
   b) In Rounds every 60-90 days where 5 projects are chosen by a Community vote

3. What is the Sponsor Program's APY formula for inflationary DEV rewards?<br/>
   a) Dev Protocol's inflation APY needs to be revised to account for Sponsor Program and Incubator

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
