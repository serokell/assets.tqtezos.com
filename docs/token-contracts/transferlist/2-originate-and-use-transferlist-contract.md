---
id: 2-originate-and-use-transferlist-contract
title: Originate and Use
---


Make sure you have already followed the 
[setup steps](/docs/token-contracts/transferlist/1-transferlist-intro#setting-up) before continuing.

To see a list of supported contracts and actions, run:
`lorentz-contract-multisig --help`


# Generating the Transferlist Contract Code

## Standalone Transferlist Contract

An example initial storage:

```bash
$ lorentz-contract-transferlist Transferlist init --issuer "\"$FRED_ADDRESS\"" \
  --transferlists "[]" \
  --users "[]" \
  --admin "\"$FRED_ADDRESS\"" \
  --initialStorageType 'address'

Pair (Pair "tz1RwoEdg4efDQHarsw6aKtMUYvg278Gv1ir" { }) (Pair { } "tz1RwoEdg4efDQHarsw6aKtMUYvg278Gv1ir")
```

To originate the contract:

```bash
$ tezos-client --wait none originate contract Transferlist \
  transferring 0 from $FRED_ADDRESS running \
  "$(cat contracts/address_transferlist.tz)" \
  --init "$(lorentz-contract-transferlist Transferlist init --issuer "\"$FRED_ADDRESS\"" \
  --transferlists "[]" \
  --users "[]" \
  --admin "\"$FRED_ADDRESS\"" \
  --initialStorageType 'address')" --burn-cap 2.937

Waiting for the node to be bootstrapped before injection...
Current head: BLhwwjTTonUC (timestamp: 2020-07-09T19:24:06-00:00, validation: 2020-07-09T19:24:38-00:00)
Node is bootstrapped, ready for injecting operations.
Estimated gas: 77713 units (will add 100 for safety)
Estimated storage: 2937 bytes added (will add 20 for safety)
Operation successfully injected in the node.
Operation hash is 'ooMkQ2WYZhM9FRTSmg1ijJ5PT4DChzZA8rjPsroM1MR4YjsXxdG'
NOT waiting for the operation to be included.
Use command
  tezos-client wait for ooMkQ2WYZhM9FRTSmg1ijJ5PT4DChzZA8rjPsroM1MR4YjsXxdG to be included --confirmations 30 --branch BKk5pbbgmu4ZnA7NjcSwsZF66h66dc287WQFtRz8fM2NZLfLY75
and/or an external block explorer to make sure that it has been included.
This sequence of operations was run:
  Manager signed operations:
    From: tz1RwoEdg4efDQHarsw6aKtMUYvg278Gv1ir
    Fee to the baker: ꜩ0.010656
    Expected counter: 623955
    Gas limit: 77813
    Storage limit: 2957 bytes
    Balance updates:
      tz1RwoEdg4efDQHarsw6aKtMUYvg278Gv1ir ............. -ꜩ0.010656
      fees(tz1Ke2h7sDdakHJQh8WX4Z372du1KChsksyU,270) ... +ꜩ0.010656
    Origination:
      From: tz1RwoEdg4efDQHarsw6aKtMUYvg278Gv1ir
      Credit: ꜩ0
      Script:
        { ... }
        Initial storage:
          (Pair (Pair "tz1RwoEdg4efDQHarsw6aKtMUYvg278Gv1ir" {})
                (Pair {} "tz1RwoEdg4efDQHarsw6aKtMUYvg278Gv1ir"))
        No delegate for this contract
        This origination was successfully applied
        Originated contracts:
          KT1FHaNcQwQt6Ug92iKuFjFs31bQTSDU92Fi
        Storage size: 2680 bytes
        Updated big_maps:
          New map(9382) of type (big_map nat (pair bool (set nat)))
          New map(9381) of type (big_map address nat)
        Paid storage size diff: 2680 bytes
        Consumed gas: 77713
        Balance updates:
          tz1RwoEdg4efDQHarsw6aKtMUYvg278Gv1ir ... -ꜩ2.68
          tz1RwoEdg4efDQHarsw6aKtMUYvg278Gv1ir ... -ꜩ0.257

New contract KT1FHaNcQwQt6Ug92iKuFjFs31bQTSDU92Fi originated.
Contract memorized as Transferlist.
```

```bash
$ TRANSFERLIST_ADDRESS="KT1FHaNcQwQt6Ug92iKuFjFs31bQTSDU92Fi"
```

# Populating the contract

Let's populate the contract with users and transferlists.

## Adding users

To generate the parameter to add `$ALICE_ADDRESS` to transferlist `0`:

```bash
$ lorentz-contract-transferlist Transferlist UpdateUser \
  --newUser "\"$ALICE_ADDRESS\"" --newUserType 'address' \
  --newUserTransferlistId 0

Right (Right (Left (Left (Right (Pair "tz1R3vJ5TV8Y5pVj8dicBR23Zv8JArusDkYr" (Some 0))))))
```

We can submit the parameter with the following:

```bash
$ tezos-client --wait none transfer 0 from $FRED_ADDRESS to $TRANSFERLIST_ADDRESS \
  --arg "$(lorentz-contract-transferlist Transferlist UpdateUser \
  --newUser "\"$ALICE_ADDRESS\"" --newUserType 'address' \
  --newUserTransferlistId 0)"
```

Since we'll be adding four users, here's a convenient way to add a user:

```bash
$ transferlist_add_user(){ tezos-client --wait none transfer 0 from $FRED_ADDRESS to $TRANSFERLIST_ADDRESS \
  --arg "$(lorentz-contract-transferlist Transferlist UpdateUser \
  --newUser "\"$1\"" --newUserType 'address' \
  --newUserTransferlistId "$2")" --burn-cap 1.0 }
```

## Adding transferlists

To generate the parameter to add a transferlist with:
- `transferlistId`: `0`
- That can transfer to transferlists `0, 2`
- `unrestricted`

```bash
$ lorentz-contract-transferlist Transferlist \
  SetTransferlistOutbound --transferlistId 0 --outboundTransferlist "[0,2]"

Right (Right (Left (Right (Left (Pair 0 (Some (Pair True { 0; 2 })))))))
```

As with users, here's a convenient way to add a transferlist:

```bash
$ add_transferlist(){ tezos-client --wait none transfer 0 from $FRED_ADDRESS to $TRANSFERLIST_ADDRESS \
  --arg "$(lorentz-contract-transferlist Transferlist \
  SetTransferlistOutbound --transferlistId "$1" --outboundTransferlist "$2" $3)" --burn-cap 1.0 }
```

## Batching the setup

Users and transferlists may be added in any order.

To populate the contract with the following setup:

Users:
- `$ALICE_ADDRESS: 0`
- `$BOB_ADDRESS: 0`
- `$CHARLIE_ADDRESS: 1`
- `$DAN_ADDRESS: 2`

Transferlists:
- `0: (unrestricted: True, allowedTransferlists: {0, 2})`
- `1: (unrestricted: True, allowedTransferlists: {1})`
- `2: (unrestricted: False, allowedTransferlists: {1, 2})`

We can run:

```bash
transferlist_add_user $ALICE_ADDRESS 0
transferlist_add_user $BOB_ADDRESS 0
transferlist_add_user $CHARLIE_ADDRESS 1
transferlist_add_user $DAN_ADDRESS 2
add_transferlist 0 '[0,2]'
add_transferlist 1 '[1]'
add_transferlist 2 '[1,2]' --restricted
```

Here are links to the operations:
- [`transferlist_add_user $ALICE_ADDRESS 0 `](https://tzkt.io/oniDs2jTbHBLVs8GZtchLf7pYovfPFdRhqiur5Ro9k6erRXwBzt)
- [`transferlist_add_user $BOB_ADDRESS 0 `](https://tzkt.io/ooX4hXeCMTV41GGwWPPn9Dgqfomq4uh7eyUF8bfSyFgemPCmMQp)
- [`transferlist_add_user $CHARLIE_ADDRESS 1 `](https://tzkt.io/onjyQi9rhZT5LDUXhicn8aoBvKw6Py5AFpxoNtEGNrc5yZc1VPH)
- [`transferlist_add_user $DAN_ADDRESS 2 `](https://tzkt.io/opDD3SdtYrUAvdRUUoK1wGkDZdVDUBgi8DGrFBHneUojJoqZP86)
- [`add_transferlist 0 '[0,2]'`](https://tzkt.io/onvR83Mm5qHh33ks4fEBukunVyMqmWroCV1Z8SsPbLkDyDRusXC)
- [`add_transferlist 1 '[1]'`](https://tzkt.io/onrXeYVmWGoQDyXv76Hi8vQA48LnZikgXY3X3ZQ4n9hkg3Asvrc)
- [`add_transferlist 2 '[1,2]' --restricted`](https://tzkt.io/ooUcGV99euBT1dogLVbTjD3WSpEQ7fnBTfrbtvq7SKJX1CKB39S)


# `assertTransfers`

Succeed if and only if, for each `from` and their associated `to`'s in the list,
either:
- Both
  * `assertReceivers` would succeed for both the `from` and `to` `userId`'s
  * `to`'s `transferlistId` is in the `set` of `from`'s transferlist's set of allowed outbound transferlists
- Or both:
  * `assertReceivers` would succeed for the `to` `userId`
  * The `from` `userId` is the `issuer`'s `userId`

This is equivalent to the following pseudocode:

```python
def assertTransfers(input_list):
  for from, tos in input_list:
    for to in tos:
      if from == issuer:
        assertReceivers [to]
      else:
        assertReceivers [from, to]
        users.get(to) in transferlists.get(users.get(from)).allowedTransferlists
```

```
(list %assertTransfers (pair (userId %from)
                             (list %tos userId)))
```

See [FA2's `transfer`](https://gitlab.com/tzip/tzip/-/blob/master/proposals/tzip-12/tzip-12.md#transfer)
for an example of a similarly batched entrypoint.

## Examples

Using the setup from before, suppose the following call to `assertTransfers` 
were made:

```
assertTransfers 
  { Pair "$ALICE_ADDRESS" { "$BOB_ADDRESS", "$DAN_ADDRESS" }
  , Pair "$BOB_ADDRESS" { "$ALICE_ADDRESS" }
  , Pair "$CHARLIE_ADDRESS" { "$CHARLIE_ADDRESS", "$DAN_ADDRESS" }
  }
```

- `alice -> bob`: `alice` and `bob` are on the same transferlist (`0`),
  which contains itself in its `allowedTransferlists` and is `unrestricted`,
  so this succeeds
- `alice -> dan`: `alice` is on a transferlist (`0`) that contains `dan`'s `transferlistId` (`2`) in its `allowedTransferlists` and is `unrestricted`,
  but it fails because `dan`'s transferlist is restricted
- `bob -> alice`: This succeeds by the same logic as `alice -> bob`: they're on the same `unrestricted` transferlist that contains its own `transferlistId` in its `allowedTransferlists`
- `charlie -> charlie`: This succeeds since `charlie`'s transferlist is unrestricted and contains its own `transferlistId` in its `allowedTransferlists`
- `charlie -> dan`: This fails because `dan`'s transferlist (`2`) is restricted

Thus the above call to `assertTransfers` will fail.

Here's how to generate it:

```bash
$ lorentz-contract-transferlist Transferlist AssertTransfers \
  --userType 'address' --list \
  "{ Pair \"$ALICE_ADDRESS\" { \"$BOB_ADDRESS\"; \"$DAN_ADDRESS\" }; \
  Pair \"$BOB_ADDRESS\" { \"$ALICE_ADDRESS\" }; \
  Pair \"$CHARLIE_ADDRESS\" { \"$CHARLIE_ADDRESS\"; \"$DAN_ADDRESS\" } }"

Left { Pair "tz1R3vJ5TV8Y5pVj8dicBR23Zv8JArusDkYr" { "tz1bDCu64RmcpWahdn9bWrDMi6cu7mXZynHm"; "tz1aHNfXQkBTKUDtkYrZ1iHhAoV3uZYgmkz1" }; Pair "tz1bDCu64RmcpWahdn9bWrDMi6cu7mXZynHm" { "tz1R3vJ5TV8Y5pVj8dicBR23Zv8JArusDkYr" }; Pair "tz1e95DfgHze6hszPbrkdWeVb4vPiDdJWBhF" { "tz1e95DfgHze6hszPbrkdWeVb4vPiDdJWBhF"; "tz1aHNfXQkBTKUDtkYrZ1iHhAoV3uZYgmkz1" } }
```

To submit the `assertTransfers` check:

```bash
$ tezos-client --wait none transfer 0 from $FRED_ADDRESS to $TRANSFERLIST_ADDRESS \
  --arg "$(lorentz-contract-transferlist Transferlist \
  AssertTransfers --userType 'address' --list \
  "{ Pair \"$ALICE_ADDRESS\" { \"$BOB_ADDRESS\"; \"$DAN_ADDRESS\" }; \
  Pair \"$BOB_ADDRESS\" { \"$ALICE_ADDRESS\" }; \
  Pair \"$CHARLIE_ADDRESS\" { \"$CHARLIE_ADDRESS\"; \"$DAN_ADDRESS\" } }")"

Waiting for the node to be bootstrapped before injection...
Current head: BLzZBqGJMh4E (timestamp: 2020-07-09T20:40:36-00:00, validation: 2020-07-09T20:41:01-00:00)
Node is bootstrapped, ready for injecting operations.
This simulation failed:
  Manager signed operations:
    From: tz1RwoEdg4efDQHarsw6aKtMUYvg278Gv1ir
    Fee to the baker: ꜩ0
    Expected counter: 623963
    Gas limit: 1040000
    Storage limit: 60000 bytes
    Transaction:
      Amount: ꜩ0
      From: tz1RwoEdg4efDQHarsw6aKtMUYvg278Gv1ir
      To: KT1FHaNcQwQt6Ug92iKuFjFs31bQTSDU92Fi
      Parameter: (Left { Pair "tz1R3vJ5TV8Y5pVj8dicBR23Zv8JArusDkYr"
                              { "tz1bDCu64RmcpWahdn9bWrDMi6cu7mXZynHm" ;
                                "tz1aHNfXQkBTKUDtkYrZ1iHhAoV3uZYgmkz1" } ;
                         Pair "tz1bDCu64RmcpWahdn9bWrDMi6cu7mXZynHm"
                              { "tz1R3vJ5TV8Y5pVj8dicBR23Zv8JArusDkYr" } ;
                         Pair "tz1e95DfgHze6hszPbrkdWeVb4vPiDdJWBhF"
                              { "tz1e95DfgHze6hszPbrkdWeVb4vPiDdJWBhF" ;
                                "tz1aHNfXQkBTKUDtkYrZ1iHhAoV3uZYgmkz1" } })
      This operation FAILED.

Runtime error in contract KT1FHaNcQwQt6Ug92iKuFjFs31bQTSDU92Fi:
  001: { parameter
  002:     (or (list %assertTransfers (pair address (list address)))
  ...
  070:                                                   IF {} { PUSH string "outbound not transferlisted" ; FAILWITH } } } ;
  ...
  271:          PAIR } }
At line 70 characters 102 to 110,
script reached FAILWITH instruction
with "outbound not transferlisted"
Fatal error:
  transfer simulation failed
```

If we remove the transfers that fail the assertion
(see [TZIP-15](https://gitlab.com/tzip/tzip/-/blob/master/proposals/tzip-15/tzip-15.md#asserttransfers)
for more detail), the assertion succeeds:

```bash
$ tezos-client --wait none transfer 0 from $FRED_ADDRESS to $TRANSFERLIST_ADDRESS \
  --arg "$(lorentz-contract-transferlist Transferlist \
  AssertTransfers --userType 'address' --list \
  "{ Pair \"$ALICE_ADDRESS\" { \"$BOB_ADDRESS\" }; \
  Pair \"$BOB_ADDRESS\" { \"$ALICE_ADDRESS\" }; \
  Pair \"$CHARLIE_ADDRESS\" { \"$CHARLIE_ADDRESS\" } }")" --burn-cap 0.000001

Waiting for the node to be bootstrapped before injection...
Current head: BLjCwGGsznH8 (timestamp: 2020-07-09T20:43:06-00:00, validation: 2020-07-09T20:43:40-00:00)
Node is bootstrapped, ready for injecting operations.
Estimated gas: 72505 units (will add 100 for safety)
Estimated storage: no bytes added
Operation successfully injected in the node.
Operation hash is 'op5a4BjNXh39vqkFgeyRj1m87W9vVmTX5kYFNKwjPv34fL4eo2m'
NOT waiting for the operation to be included.
Use command
  tezos-client wait for op5a4BjNXh39vqkFgeyRj1m87W9vVmTX5kYFNKwjPv34fL4eo2m to be included --confirmations 30 --branch BLJoirBhCYFmQWfmedAdLk6UgBFQAKUY2GuQyDA2Gbb7hinTfqL
and/or an external block explorer to make sure that it has been included.
This sequence of operations was run:
  Manager signed operations:
    From: tz1RwoEdg4efDQHarsw6aKtMUYvg278Gv1ir
    Fee to the baker: ꜩ0.007791
    Expected counter: 623964
    Gas limit: 72605
    Storage limit: 0 bytes
    Balance updates:
      tz1RwoEdg4efDQHarsw6aKtMUYvg278Gv1ir ............. -ꜩ0.007791
      fees(tz1Ke2h7sDdakHJQh8WX4Z372du1KChsksyU,270) ... +ꜩ0.007791
    Transaction:
      Amount: ꜩ0
      From: tz1RwoEdg4efDQHarsw6aKtMUYvg278Gv1ir
      To: KT1FHaNcQwQt6Ug92iKuFjFs31bQTSDU92Fi
      Parameter: (Left { Pair "tz1R3vJ5TV8Y5pVj8dicBR23Zv8JArusDkYr"
                              { "tz1bDCu64RmcpWahdn9bWrDMi6cu7mXZynHm" } ;
                         Pair "tz1bDCu64RmcpWahdn9bWrDMi6cu7mXZynHm"
                              { "tz1R3vJ5TV8Y5pVj8dicBR23Zv8JArusDkYr" } ;
                         Pair "tz1e95DfgHze6hszPbrkdWeVb4vPiDdJWBhF"
                              { "tz1e95DfgHze6hszPbrkdWeVb4vPiDdJWBhF" } })
      This transaction was successfully applied
      Updated storage:
        (Pair (Pair 0x0000452d024b72d5897f14a02dc1a3b8e012c802cc3d 9381)
              (Pair 9382 0x0000452d024b72d5897f14a02dc1a3b8e012c802cc3d))
      Storage size: 3180 bytes
      Consumed gas: 72505
```

