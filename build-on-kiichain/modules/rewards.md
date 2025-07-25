# Rewards

The rewards module adds the distribution of rewards from a specific community pool. Anyone can fund the pool but to change or initiate a reward distribution, a proposal needs to be passed.

### Flow

1. Fund community pool with reward
2. Create and pass a proposal to create a release schedule
3. At the end of every block, a linear % of the reward will be forwarded for distribution
4. When the end time of the release is reached, all rewards will have been given away, and it will go inactive

### Internal state

To properly release on time, calculate rewards, and keep track, we have a RewardReleaser with the following internal information:

```go
type RewardReleaser struct {
    // Total amount to be rewarded
    TotalAmount types.Coin `protobuf:"bytes,1,opt,name=total_amount,json=totalAmount,proto3" json:"total_amount" yaml:"total_amount"`
    // Amount released
    ReleasedAmount types.Coin `protobuf:"bytes,2,opt,name=released_amount,json=releasedAmount,proto3" json:"released_amount" yaml:"released_amount"`
    // Timestamp of end of release
    EndTime time.Time `protobuf:"bytes,3,opt,name=end_time,json=endTime,proto3,stdtime" json:"end_time" yaml:"end_time"`
    // Last height released
    LastReleaseTime time.Time `protobuf:"bytes,5,opt,name=last_release_time,json=lastReleaseTime,proto3,stdtime" json:"last_release_time" yaml:"last_release_time"`
    // If reward pool is active
    Active bool `protobuf:"varint,6,opt,name=active,proto3" json:"active,omitempty" yaml:"active"`
}
```

At the end of each block, if the releaser is active:

* It will calculate the amount to be distributed linearly across time based on the last release and the current block time.
* If the amt to be distributed is zero, it goes inactive
* It sends the amount from the pool to the fee collector
* It increases the released amount, the last release time, and the community pool with the changes.

### Messages

#### FundPool

Sends funds to the community pool, to be used in a future release.

```protobuf
message MsgFundPool {
  string sender = 1 [ (gogoproto.moretags) = "yaml:\"sender\"" ];
  cosmos.base.v1beta1.Coin amount = 2 [
    (gogoproto.moretags) = "yaml:\"amount\"",
    (amino.encoding) = "legacy_coin"
  ];
}
```

**State Modifications:**

* The community pool funds will increase, as well as the module's balance

#### ChangeSchedule

Set the schedule to match what is sent. Only the governor can utilize this call, others need to pass a proposal.

```protobuf
message MsgChangeSchedule {
  // authority is the address of the governance account.
  string authority = 1 [ (cosmos_proto.scalar) = "cosmos.AddressString" ];


  // New information for the schedule
  ReleaseSchedule schedule = 2 [ (gogoproto.nullable) = false ];
}

message ReleaseSchedule {
  // Total amount to be rewarded
  cosmos.base.v1beta1.Coin total_amount = 1 [
    (gogoproto.moretags) = "yaml:\"total_amount\""
  ];
  // Amount released
  cosmos.base.v1beta1.Coin released_amount = 2 [
    (gogoproto.moretags) = "yaml:\"released_amount\""
  ];
  // Timestamp of end of release
  google.protobuf.Timestamp end_time = 3 [
    (gogoproto.stdtime) = true,
    (gogoproto.moretags) = "yaml:\"end_time\""
  ];
  // Last height released
  google.protobuf.Timestamp last_release_time = 5 [
    (gogoproto.stdtime) = true,
    (gogoproto.moretags) = "yaml:\"last_release_time\""
  ];
  // If reward pool is active
  bool active = 6 [
    (gogoproto.moretags) = "yaml:\"active\""
  ];
}
```

**State Modifications:**

* Safety check the following
  * Denom of the amt must be the one being used
  * End time must be in the future
  * Funds must be available in the pool
* Changes the reward release schedule to match what is sent

#### Update Params

Changes module params. Only the governor can utilize this call; others need to pass a proposal.

```protobuf
message MsgUpdateParams {
  // authority is the address of the governance account.
  string authority = 1 [ (cosmos_proto.scalar) = "cosmos.AddressString" ];

  // params defines the x/rewards parameters to update.
  //
  // NOTE: All parameters must be supplied.
  Params params = 2 [ (gogoproto.nullable) = false ];
}
message Params {
  // Minimal deposit
  string governance_min_deposit = 1;

  // Denom used
  string token_denom = 2;
}
```

**State Modifications:**

* Changes the token\_denom and min deposit

### Other important flows

The releaser has a few edge cases that happen when it is initializing or going inactive:

#### First iteration

* There is no previous release time, so we cannot calculate a reward distribution
* Instead of calculating the reward, we just set the last release as the block time
* Next iteration will cover it well

#### Last iteration

* As the first release is delayed, so will be the last one
* Once the EndTime is passed, all the remaining rewards will be distributed
* The releaser will just go inactive a block after, when there is no amount to distribute
