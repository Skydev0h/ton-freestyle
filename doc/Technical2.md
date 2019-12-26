# Technical details of implementation

## Data storage proxy SC

### Data structure

| Variable name   | VarType | Bit length | Description                                    |
| --------------- | ------- | ---------- | ---------------------------------------------- |
| reserved        | `void`  | `6`        | Alignment                                      |
| frt_change_p    | `Bit`   | `1`        | Is FRT change pending now?                     |
| box_tree        | `Dict`  | `1 + Ref`  | Contains all the actual data for this contract |
| seq_no          | `UInt`  | 32         | Sequence number for external messages          |
| owner_key       | `UInt`  | `256`      | Contains public key of the contract owner      |
| protected_frt   | `UInt`  | `24`       | Force Reset Timeout for protected cells        |
| *new_frt_value* | `UInt`  | `24`       | *If FCP* new FRT value to be set after timer   |
| *new_frt_timer* | `UInt`  | `32`       | *If FCP* when allow changing FRT value         |

Box tree is a prefix tree keyed by box names (suffixed with Ø) and containing the defined structure values.

### Box structure

| Variable name | VarType | Bit length | Description                                                  |
| ------------- | ------- | ---------- | ------------------------------------------------------------ |
| ref_data      | `Cell`  | `Ref`      | Data is always referenced because struct is inline to ptree  |
| fr_pending    | `Bit`   | `1`  fuse  | Whether force reset of protected cell is currently pending   |
| allow_get     | `Bit`   | `1`  chown | Allow to obtain value of box with getter method **           |
| allow_inline  | `Bit`   | `1`  chown | Allows injecting this box's value and proxying message       |
| allow_inquiry | `Bit`   | `1`  chown | Allows requesting this box's value with internal message     |
| protected     | `Fuse`  | `1`  fuse  | A **fuse*** that sets this box as protected. Protected boxes have some restrictions for **contract owner** |
| use_quota     | `Bit`   | `1`        | Allows restricting maximum amount of stored data using quota parameters defined below |
| is_frozen     | `Bit`   | `1`        | Specifies that this box is frozen, before changing anything or deleting it, this flag must be cleared |
| has_owner     | `Bit`   | `1`        | Defines that this box has the specified owner                |
| *owner_wc*    | `Int`   | `8`        | *If has_owner* contains the workchain ID of this box owner's smart contract |
| *owner_addr*  | `UInt`  | `256`      | *If has_owner* specifies public key or smart contract address of owner of this box |
| *quota_cells* | `Gram`  | `4-124`    | *If use_quota* specifies max amount of refs in data          |
| *quota_bits*  | `Gram`  | `4-124`    | *If use_quota* specifies max amount of bits in data          |
| *fr_timeout*  | `UInt`  | `32`       | *If force reset is pending* specifies after which moment of time force reset can be carried out. |

*** fuse** is a bit can be set to 1, but cant be reset back to 0. Only **box** owner can reset the **protected** to 0.

** that bit do not actually protect access to data because data is publicly visible on the blockchain, and is more like a safety lever to prevent inadvertent access to data using getter methods

Variables marked with chown or cho can be altered by **box owner**, other variables can be altered only by **contract owner** (with exception of **fuses**).

Quotas are recursive and quota_cells does account for data root cell, zero means disabled.

If [**protected**](#Protection and resetting) fuse is set, contract owner is restricted in performing some actions, however owner can reset the **protected** fuse by initiating [Forced Reset](#Protection and resetting), more details on that later.

---

### The algorithm

First, contract is initialized by contract owner with some parameters preset and box tree empty.

Afterwards, contract owner can issue external messages to create and initialize boxes. Contract owner can afterwards modify those boxes' settings and values.

If box owners are set, they can change some settings of their boxes (allowance flags and resetting protected flag) via internal ~~or external~~ messages (depending of owner configuration of the box). Data used in a specific box can be limited by use of `use_quota` and corresponding `quota_cells` and `quota_bits`.

(*Ability to use external messages was thrown out due to possible problems with gas credit*)

After value is set in box, it can be used, if allowed by flags, in several different ways:

* If `allow_get` is set, value of this smart contract can be read with get methods
* If `allow_inline` is set, the value can be injected into a message and forwarded
* If `allow_inquiry` is set the value can be returned for a direct inquiry request (int. msg.)

Contract owner can also protect some box from himself (if owner is defined) by setting `protected` fuse.

---

### Protection and resetting

In order to guarantee integrity of data for box owner, contract owner can activate protection on the cell. After activating the protection, **contract owner** will be unable to deactivate **allow_** flags (they will behave like fuses), change any **owner-related** variables, activate or decrease **quota**, modify data or delete box.

In order to provide ability for contract owner to reclaim the box and forcefully unset the **protected** flag the owner can initiate force reset procedure. In this case, first attempt of forced reset would activate a timer, which will expire in **protected_frt** (contract global variable) seconds. Afterwards, repeating forced reset attempt would reset **protected** flag. Box owner can object with this action and cancel the timer before it expires and is used by contract owner.

In order to guarantee some integrity, **protected_frt** timer cannot be also arbitrarily changed.  In order to change this variable, at least **protected_frt** (but no less than a day) seconds must be waited before being able to change this value. This also involves a global contract-level timer.

If **protected_frt** is zero, force reset procedure is not allowed. To activate **protected_frt** the owner would have to wait a reasonable time (say **a week minus protected_frt, but at least a day**).

---

### External messages

External messages are permitted from the owner of the contract. They begin with 512-bit signature and are followed by 32-bit operation code, after which the structure depends on concrete operation.

Following operations are supported:

| Name   | Code       | Description                                  |
| ------ | ---------- | -------------------------------------------- |
| `bnew` | `626e6577` | Create a box with set name and configuration |
| `bdel` | `6264656c` | Delete specified unprotected box             |
| `frzz` | `66727a7a` | Freezes specified unprotected box            |
| `thaw` | `74686177` | Unfreezes specified box                      |
| `prot` | `70726f74` | Protects specified box                       |
| `fres` | `66726573` | Initiates or completes force reset procedure |
| `bset` | `62736574` | Changes data of specified unprotected box    |
| `bcfg` | `62636667` | Changes configuration of specified box       |
| `chow` | `63686f77` | Changes contract owner public key            |
| `chfr` | `63686672` | Initiates or completes FRT change procedure  |

`chow` shall be followed by new owner's 256-bit public key.

`chfr` shall be followed by new 24-bit FRT value. It must be called twice - first to initiate FRT change, second (after required interval has passed) to confirm the change.

**All following messages must be immediately followed by a reference to cell with box name!**
**The next text describes what should be in message after operation code and box name ref:**

Both `bnew` and `bcfg` must be followed by [box configuration structure](#Box configuration structure).

`bdel`, `frzz`, `thaw`, `prot`, `fres` have no parameters (only aforementioned box name reference).

`bset` must be followed by a reference to new cell containing new box data.

---

### Internal messages

Internal message should contain 32-bit operation code followed by concrete operation parameters.

Supported operations for internal messages are:

| Name   | Code       | Description                                   |
| ------ | ---------- | --------------------------------------------- |
| `bset` | `62736574` | Changes data of specified owned box           |
| `bcfg` | `62636667` | Changes configuration of specified owned box  |
| `unpr` | `756e7072` | Unprotects owned protected box                |
| `prot` | `70726f74` | Object pending force reset                    |
| `inqu` | `696e7175` | Request specified box data as response        |
| `prox` | `70726f78` | Inject specified box data and forward message |

For `bset`, `bcfg`, and `prot` parameters are the same as for external message (box name reference followed by corresponding options). `unpr` has none options (like `prot` - only box name reference).

**Following two operations are used for inquiring and proxying messages.**

`inqu` shall contain reference to box name cell and response message body, that will be put into response message. Therefore, response message body will be `resp_msg_body` + `ref to cell data`.

`prox` is more complex in it's nature. You need to provide inline destination address (8 bits `workchain id` and 256 bits `address`), box value mode bit (`0` to insert as reference and `1` to insert inline) reference to box name, reference to `prefix` and reference to `suffix` cells.

The sent message will contain all provided grams minus processing and forwarding fees, and will have following body: inline `prefix` + inline or referenced `box value` + inline `suffix`. Providing correct prefix, suffix and inlining mode so that as no build overflow error occurs is responsibility of the sender.

---

### Box configuration structure

The following configuration structure is used in `bnew` and `bset` messages:

| Variable name | VarType | Bit length | Description                                                  |
| ------------- | ------- | ---------- | ------------------------------------------------------------ |
| set_flags     | `Bits`  | `8`        | Specifies flags that should be set in box. Refer to Box structure for list of flags and their order. |
| reset_flags   | `Bits`  | `8`        | Specifies flags that should be reset in box.                 |
| *owner_wc*    | `Int`   | `8`        | *If has_owner is in set_flags*, contains new owner wch id    |
| *owner_addr*  | `UInt`  | `256`      | *If has_owner is in set_flags*, contains new owner address   |
| *quota_cells* | `Gram`  | `4-124`    | *If has_quota is in set_flags*, contains new cells quota     |
| *quota_bits*  | `Gram`  | `4-124`    | *If has_quota is in set_flags*, contains new bits quota      |

It should be noted that in some conditions (**protected** or **frozen** box) some flags may not be allowed to be touched, some values may not be decreased, and some flags may not be touched in any conditions (they are changed using another methods).

---

### Getters

`seqno` getter is usual standard getter that returns current sequence number for external messages.

`get_box` returns information about specified `box_name` (slice) as a tuple corresponding to Box structure.

`get_box_value` returns specified `box_name`'s value as a cell.

`get_box` and `get_box_value` may fail if target box does not have `allow_get` bit set.

