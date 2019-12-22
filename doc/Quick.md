# "Quick" start (deploying and using)

This page outlines required information to deploy and use provided smart contracts.

Lets assume for this document that `fift` interpreter is in system binary path, and FIFT library path is set as `FIFTLIB` in the environment. Otherwise, `-I ../../lib` has to be additionally specified as parameter when invoking `fift`. If it is not in system path full path is to be specified.

[Data storage proxy SC](#Data storage proxy SC) is below the section about conditional transfer SC.

## Conditional transfer SC

### Overview

In order to facilitate usage of the smart contract, there are some scripts to be used:

`ctsc.fc` and `ctsc.fif` are the code of SC itself, they should not be touched.

`ct-create.fif` can be used to prepare query to create a new SC instance.

`ct-ext-ctrl-gen.fif` generates external messages to control the SC.

`ct-int-ctrl-gen.fif` generates payloads for internal messages to control the SC.

`ct-int-ctrl-reclaim.boc` and `ct-int-ctrl-release.boc` are the corresponding payloads.

`ct-test.fif` is a complete test suite for the contract (requires core patch).

More about that later on the text.

### Contract creation

To create the contract you need to use `ct-create.fif` script. Invoking it without any parameters displays the usage help enlisting all possible parameters and describing their values. 

Following table describes the parameters in detail:

| Short | Long parameter str   | Description                                                  |
| ----- | -------------------- | ------------------------------------------------------------ |
| -e    | --ext-ctrl           | Specifies that this contract is to be controlled by external messages, should be followed by key file base name. |
| -i    | --int-ctrl           | Specifies that this contract is to be controlled by internal messages from some contract, which's address goes next. |
| -0    | --min-grams          | Sets minimum amount of grams that have to be collected by contract in order to be able to release funds. |
| -9    | --max-grams          | Sets maximum amount of grams that can be collected by contract, all extra will be bounced back. |
| -m    | --min-accepted       | Sets minimum amount of grams that must be in a single message for it to be accepted by the contract. |
| *-c*  | *--collect-deadline* | Sets deadline for the collection phase of the contract, if min_grams are not collected by this time, refund is done. |
| *-l*  | *--release-locktime* | Sets locktime for controller actions on the contract, controller cannot release or refund until this time. |
| *-d*  | *--release-deadline* | Sets deadline for releasing or refunding collected grams, after this time any message will trigger action governed by presence or absence of -a (--auto-release) flag. |
| -b    | --ult-ben-addr       | Specifies ultimate beneficiary address, that will receive all funds not distributed to other beneficiaries. |
| -n    | --continuous-coll    | *Flag* that permits collecting funds after collect deadline. |
| -a    | --auto-release       | *Flag* that causes release of funds after release deadline, otherwise, if absent, refund would be done therefore. |
| -r    | --relative-time      | *Flag* that causes *unixtime parameters (in italic)* to be counted relative to current time, not to absolute unixtime origin. |
| -P    | --ben-pct            | Defines percent of total collected sum that would be sent to next defined beneficiary (-B / --ben-addr) upon release. |
| -V    | --ben-val            | Defines exact sum in grams that would be sent to next defined beneficiary (-B / -- ben-addr) upon release. |
| -B    | --ben-addr           | Installs a new beneficiary into a beneficiary list. Followed by beneficiary address, percent or value should be preset with either -P / --ben-pct or -V / --ben-val parameter. |
|       | --ok                 | Suppresses warnings and forces creation. Not recommended.    |

It should be noted that *unixtime parameters* are absolute unless -r *flag* is used, then they are calculated from current time (except for 0 which remains 0 - that is, disabled).

-B parameter may appear as many times as needed, -P or -V parameter should appear before it to define how much and what would be sent to that beneficiary.

After creating the query, some amount of Grams (like 5-10 to be safe) should be sent to the initialization address, after which contract creation query should be sent. All next transfers and commands to the contract shall be sent to the normal, bounceable address.

#### Example: Escrow contract

To create an escrow contract controlled with keypair (external messages), you can use command similar to following:

```bash
fift -s ct-create.fif -e escrow -0 495 -9 505 -m 500 -r -c 3600 -d 3600 
     -b Ef8AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADAU -- -1 escrow
```

It should be noted that usually the controller (arbiter) himself should create this contract because creating it requires private key.

This command will store query into `escrow-create.boc` file and private key into `escrow.pk`. Also, contract address will be saved into `escrow.addr`.

The contract requires at least 495 grams for release, but no more than 505. A single message must carry at least 500 grams. For this example the deadlines are set in a hour from the script execution moment, therefore if controller does not confirm release by that time, funds will become reclaimable (because -a flag is not present). 

**Note** that beneficiary in this example is a -1:0...0 contract, so executing this command as is would create a contract that would burn funds (and issue warning upon creation).

#### Example: Automatic crowdfunding contract (positive)

To create an crowdfunding contract that would automatically send money to beneficiary as soon as required target is achieved, you can use something like this:

```bash
fift -s ct-create.fif -0 1000 -m 10 -r -d 1 -a
     -b Ef8AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADAU -- -1 autocrowdfund
```

This contract would automatically transfer all collected funds to the beneficiary as soon as required sum is collected in the contract (with next incoming internal message after it). Minimal accepted value is necessary to prevent potential dust attacks. Contraption of -r -d -1 -a would set release deadline 1 second in future and enable auto release, therefore as soon as required sum is collected it would be automatically released.

#### Example: Automatic timed crowdfunding (with deadlines)

This contract differs in that way, that sum must be collected by some moment, otherwise it would be reclaimed by senders.

```bash
fift -s ct-create.fif -0 1000 -m 10 -r -c 3600 -d 3600 -a 
     -b Ef8AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADAU -- -1 timedcrowdfund
```

This way, if 1000 grams are collected in hour, they will be released to beneficiary after this hour passes. Otherwise, they will be reclaimable by their senders and never be released.

#### Example: Controlled crowdfunding

For this example, we have a crowdfunding that must both reach some goal by set moment, later be accepted by a controller from some contract (maybe even multisig for voting) and this has minimum and maximum clearance time too. Lets get started:

```bash
fift -s ct-create.fif -0 1000 -9 2000 -m 10 -r -c 3600 -l 7200 -d 10800
     -i Ef8zMzMzMzMzMzMzMzMzMzMzMzMzMzMzMzMzMzMzMzMzM0vF -n
     -b Ef8AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADAU -- -1 ctrlcrowdfund
```

This complex example involves many settings used. First of all, it defines minimum collect target as 1000 grams, maximum collected as 2000 grams and minimum deposit value of 10 grams. Afterwards, it defines three time barriers: in 1 hour, collection phase will end, and funds would be refunded if they did not meet required minimum amount, in 2 hours release lock would fall, before that moment controller would not be able to neither release funds or allow reclaiming, and, finally, in 3 hours if action was not taken by controller by this time funds would become reclaimable automatically (since -a flag is not set in this example). For this example, contract can be controlled by a -1:3...3 smart contract, and beneficiary is -1:0...0 one.

#### Example: Multiple beneficiaries

Lets reconsider the first example, and mix in some additional beneficiaries:

```bash
fift -s ct-create.fif -e escrowmul -0 495 -9 505 -m 500 -r -c 3600 -d 3600 
     -V 20 -B Ef8zMzMzMzMzMzMzMzMzMzMzMzMzMzMzMzMzMzMzMzMzM0vF
  -P 10 -B -1:2222222222222222222222222222222222222222222222222222222222222222
     -b Ef8AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADAU -- -1 escrowmul
```

It should be noted that -V ... -B ... and -P ... -B ... pairs can be repeated as much as needed.

This example would cause release phase to be as following:

* Exactly 20 grams would be sent to first beneficiary (-1:3...3)
* 10% of all collected sum (about 50 Grams) would be sent to second one (-1:2...2)
* Finally, all remaining sum would be sent to ultimate beneficiary (-1:0...0)

It should be noted carefully that percentage is calculated from total sum, not remaining one. Therefore, creator should be careful so that all sums fit if only minimum sum is collected.

Contract creation script takes care of that, and warns, if calculated amount to be sent if only minimum allowed amount of grams is collected is not enough to send all funds including percentages and fixed sums.

### Generating external messages

In order to generate external message `ct-ext-ctrl-gen.fif` must be used.

It expects the desired operation, contract address (or @file containing it) and base filename of private key file. It will point out name of created boc file that must be sent to network.

### Using internal messages

The required boc files to be embedded by user's wallet script are already created and called correspondingly `ct-int-ctrl-reclaim.boc` and `ct-int-ctrl-release.boc` . They are readily available to be embedded and used against any deployed SC contract.

If for some reason those boc files need to be regenerated, `ct-int-ctrl-gen.fif` script can be called that will recreate those files.

### Testing

This package contains (yet another) new version of my personal test suite that I created to test my own contracts. However this time it comes with a twist: in order to test c5 control register I had to patch (extend) TON core itself. Therefore, to use that test set to it's full extent, a patched and recompiled fift executable is required (unless somehow my patches end up in ton master). More information on that on the [main page](../README.md). 

If patch cannot be applied, first two code rows can swap commentaries to disable the c5 functionality, but then you would miss out much beauty and usefulness of the tests.

Almost forgot, the test script is `ct-test.fif` and it thoroughly tests external, internal messages, output actions in many different conditions (manipulating balance, time and state of the virtual contract).



---

## Data storage proxy SC

### Overview

In order to facilitate usage of the smart contract, there are some scripts to be used:

`dspsc.fc` and `dspsc.fif` are the code of SC itself, they should not be touched.

WIP

### Contract creation

WIP

### Generating external messages

WIP

### Generating internal messages

WIP

### Testing

WIP