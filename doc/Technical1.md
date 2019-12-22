# Technical details of implementation

## Conditional transfer SC

### Data structure

| Variable name     | Var. type | Bit length | Description                                                  |
| :---------------- | :-------- | :--------- | ------------------------------------------------------------ |
| min_grams         | `Gram`    | `4 - 124`  | Minimum amount of grams collected required to  perform transfer to destination ("release") |
| max_grams         | `Gram`    | `4 - 124`  | Maximum amount of grams that can be collected, overflowing this will bounce message |
| min_accepted      | `Gram`    | `4 - 124`  | Minimum amount of grams accepted in message                  |
| collect_deadline  | `UInt`    | `32`       | If `min_grams` would not be collected by this time, users will be able to reclaim their grams |
| release_locktime  | `UInt`    | `32`       | Funds may not be released until this time                    |
| release_deadline  | `UInt`    | `32`       | If funds are not released by this time automatic action will be performed depending on `auto_release` |
| initialized       | `Bit`     | `1`        | Specifies if contract is initialized                         |
| destroyed         | `Bit`     | `1`        | Specifies if contract is destroyed (funds were released)     |
| can_reclaim       | `Bit`     | `1`        | Indicates whether funds can be now reclaimed (happens when a deadline was not met) |
| continuous_coll   | `Bit`     | `1`        | Specifies whether funds collection continues after `collect_deadline` but before funds release or refund (manual or release deadline) |
| auto_release      | `Bit`     | `1`        | Specifies whether reaching `release_deadline` causes automatic switch to release (1) or refund (0) state |
| has_controller    | `Bit`     | `1`        | Specifies whether there is a controller who decides if to release or refund back grams. |
| investors         | `Dict`    | `1 + Ref`  | An `Dict<UInt264>`  containing balances of users that invested into this contract (wcid:I8 + addr:UI256) |
| beneficiaries     | `Cell?`   | `1 + Ref`  | A linked list of beneficiaries that are to be credited upon release of funds (optional cell = dict) |
| *ext_controller*  | `Bit`     | `1`        | If true, contract is controlled by external messages (no wc, addr is pubkey), or by internal messages otherwise from specified wc:address |
| *controller_wc*   | `Int`     | `8`        | Workchain number of controller contract                      |
| *controller_addr* | `UInt`    | `256`      | Address of controller contract or public key                 |
| beneficiary_wc    | `Int`     | `8`        | Workchain number of ultimate beneficiary                     |
| beneficiary_addr  | `UInt`    | `256`      | Address of ultimate beneficiary contract                     |
| **Max length**    |           | `1013`     | `+ 2 Ref` out of maximum `1023 + 4 Ref`                      |

If variable is zero, corresponding feature is disabled (min / max gram limits, deadlines, locktimes or flags).

**For security reasons no parameters may be changed after creation of the contract.**

Beneficiaries will receive their funds in specified order, after which ultimate beneficiary will receive all remaining funds of the contract. Structure of beneficiary entry is:

| Variable name   | Var. type | Bit length | Description                                                  |
| --------------- | --------- | ---------- | ------------------------------------------------------------ |
| workchain       | `Int`     | `8`        | Beneficiary workchain identifier                             |
| address         | `UInt`    | `256`      | Beneficiary contract address                                 |
| value           | `Gram`    | `4 - 124 ` | How much to credit this beneficiary when funds are released. If the value is zero, percent is used. |
| *percent*       | `UInt`    | `20`       | 1 / 1 000 000th of total sum to be credited, that is: 1 of this value equals 0.0001% of total sum (only if value = 0), |
| **Min length**  | val:12    | `276`      | `828` bits would be used for `3` entries (out of `1023`)     |
| **Perc length** | v+p:24    | `288`      | `864` bits w. b. u. for `3 ` entries if all use % (value = 0) |
| **Lim length**  | val:76    | `340`      | `1020` bits w. b. u. for `3` entries (value <= 2^72 nGram)   |
| **Max length**  | val:124   | `388`      | `776` bits w. b. u. for `2` entries (really?), no more space |

~~Percent may also have values higher than 1 000 000 to represent even smaller parts of sum:~~

|       ~~Min~~ | ~~Max~~       | ~~Result fraction of total~~          |
| ------------: | :------------ | ------------------------------------- |
|         ~~1~~ | ~~999 999~~   | ~~x / 1 000 000~~                     |
| ~~1 000 001~~ | ~~1 009 999~~ | ~~(x - 1 000 000) / 10 000 000~~      |
| ~~1 010 001~~ | ~~1 019 999~~ | ~~(x - 1 010 000) / 100 000 000~~     |
| ~~1 020 001~~ | ~~1 029 999~~ | ~~(x - 1 020 000) / 1 000 000 000~~   |
| ~~1 030 001~~ | ~~1 039 999~~ | ~~(x - 1 030 000) / 10 000 000 000~~  |
| ~~1 040 001~~ | ~~1 048 575~~ | ~~(x - 1 040 000) / 100 000 000 000~~ |

~~These variants allow to represent small percentages with extreme precision (up to 0.000000001‬%).~~

**The functionality above (high precision percentages) is disabled in code due to expected low usefulness, but usage of much opcodes in storage. Can be uncommented if necessary.**

Beware that in case fixed sum + % of total is larger than the collected sum, last beneficiaries may not receive their sums, so calculation of fixed sums and % of total should be carried out carefully.

---

### The algorithm

Contract is initialized with all parameters preset, `can_reclaim` must be 0. 

Afterwards, transfer(s) to this wallet can be performed by any user. The contract will return a small fixed part of such transfer minus gas and forwarding fees and credit remaining to user's investor balance in contract. There are some situations when message will be just bounced back minus fees:

* Contract's balance plus received grams overflows `max_grams` (extra grams will be returned)
* `collect_deadline` have been reached and `continuous_coll` is disabled
* Funds have been released (contract will be `destroy`ed actually after that)
* Message minus fees value is lower than `min_accepted`

In case contract fails, that is condition is not fulfilled in time, or rejection is done by controller, the user can send any (reasonable to pay for processing and forwarding fees) amount of grams to the contract, the contract will look up user's balance and return it with sent grams minus fees. In case user does not have invested balance in the contract, message will be just bounced back.

It may be theoretically possible that for some reason not all funds are distributed when releasing. Releasing funds will set `destroyed` flag, and will try to distribute all funds. Some funds may theoretically bounce (although bounce flag would not be set), therefore if contract survives this (it should not), any message to the `destroyed` contract will cause it sending all it's money to ultimate beneficiary.

---

### Internal messages

Most interaction with the contract is carried out through internal messages. They are used to put ("invest") money in the contract, to reclaim investments when collection or release phases are timed out or is rejected by controller, to release or refund accumulated money by the controller, or to trigger money release if there is no controller but release_locktime is set.

#### Putting and reclaiming grams

For interaction simplicity of ordinary users with the contract, any message received from a non-controller smart contract will be processed as a funding grams transfer or reclaim request, depending on current state of contract.

* If funding is currently allowed, received amount is credited to the sender's investment balance minus a predefined flat fee, that will be returned minus gas and forwarding fees. In case flat fee is insufficient to cover gas and forwarding fees, user can decide to adjust it's value by passing it in the message, then the alternate value will be used.
* If funding is not allowed and there is no due to be reclaimed the message will be bounced back.
* If there is some amount that is due to by the contract to the sender, message will be bounced back, but that amount will be included along it.
* No fuss and no requirement to construct complex messages, funding and reclaims are dead simple

#### Releasing or refunding collected grams

If a special internal message is received from the controlling smart contract or a signed external message by the controlling key, an action may be performed:

If funds can be released (`min_grams` was reached, `collect_deadline` and `release_locktime` have passed, `release_deadline` has not yet arrived, and, obviously, `has_controller` is true) the corresponding message from the controller may:

* initiate refund (`can_reclaim` will be set to 1), 
* or release money to beneficiaries (money will be immediately released according to beneficiary tables, and contract code and data will be replaced with void)

Enough grams should be provided to pay for processing fees and sending extra back.

**In case `auto_release` is true and `release_deadline` is passed, any internal or external message will attempt processing funds release.**

---

### External messages

External message have same body structure as internal ones (as they can originate only from controller), except that they are prefixed with 512-bit signature. They will consume contract's balance for processing of refund initiation or money release. Therefore it is important to have a reasonable amount of grams to be sent to ultimate beneficiary, because from it gas processing fees would be deducted.

Conditions for performing corresponding actions are checked before performing them, after performing reclaim `can_reclaim` flag would be set to 1 which will block any further replays of this message, upon performing release the contract will be deleted altogether. This way, no replay protection is needed.

As for the initialization message, it will check flag `initialized`, and set it to true in the first message.

---

### Message structure

#### Gram investment (putting) message

In general, to put grams into contract no special message structure is needed, a simple transfer would suffice. However, in case beneficiary tree grows enormous, flat fee limit may not be enough to cover gas expenses. Then, investor may choose to send a special message that will override default flat fee, and allow using more grams as gas fee. 

Such message should start with `op = 0x4761732b` (Gas+) followed by gram amount to be reserved and used as processing fee. `query_id` is intentionally omitted, because I see no reason to send 64 bits of useless data that would not be used anyway.

#### Reclaim message

To reclaim grams same simple transfer can be done to contract. In case the contract is in reclaim phase, all provided grams will be used as processing reserve and be returned in response message. 

#### Controlling message

A message by controller can be used to release funds to beneficiaries or to initiate reclaim phase.

To release funds message with `op = 0x52656c65` (Rele) shall be used.
To initiate reclaim message with `op = 0x52636c6d` (Rclm) shall be used.

No `query_id` or any other content should be supplied in message (except sign prefix for external).

---

### Getter methods

Getters can be used to inspect state of contract and obtain useful information from it.

#### get_config

This get method allows to retrieve all information about this contract as a tuple.

Tuple follows the data structure at the top of this document.

#### reclaiming

Shows whether contract is already in reclaiming phase, any message will try to retrieve and return reclaimable amount for this contract, if possible

#### reclaimable

Checks and outputs how much is to be reclaimed by a specific address.