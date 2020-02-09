# TON freestyle contest entry

[*(Blockchain competition number 2 - Official requirements and documentation)*](doc/Official.md)

This package contains a participation entry for second blockchain contest of TON for creating own freestyle smart contracts. 

Official description of objectives, rules and requirements is done on [the referenced page](doc/Official.md). 

As per project ideas themselves, **two** smart contract projects were chosen to be implemented:

* [**Conditional transfer smart contract**](doc/Project1.md)
* [**Data storage proxy smart contract**](doc/Project2.md)

More details about those ideas should be read on their respective pages linked above.

Update on tests: the changes have been actually **added to the master TON branch** by the TON maintainers, so no patch is needed to utilize test suite to full extent.

Quick-start instructions about using those contracts can be found [here](doc/Quick.md).

---

**NEW!** Refactored scripts using the *not yet implemented* functionality have been added with `-fcext` suffix! Please check them out! More information in the aforementioned *quick-start instructions*.

---

Technical details about implementations are on their corresponding pages.

Summarizing proposed changes and fixed in [this pull request](https://github.com/ton-blockchain/ton/pull/220) (and others) from [this fork](https://github.com/Skydev0h/ton):

| Type of change    | Name                                                         | Status         | Description                                                  |
| ----------------- | ------------------------------------------------------------ | -------------- | ------------------------------------------------------------ |
| fift enhance      | [(gas)runvmctxact(q)](https://github.com/ton-blockchain/ton/pull/220/commits/7572b6cbec1ed255a10fcb6031635e47a865e10c) | **accepted**   | Added as generalized runvm function that accepts bit flags, fift compatibility layer includes the proposed functions |
| runvm fix         | [commited states](https://github.com/ton-blockchain/ton/pull/220/commits/03857adfa55354368a064be0361ec093c854bde3) | **accepted**   | Fix implemented in runvm code                                |
| fift enhance      | [dict slice key ops](https://github.com/ton-blockchain/ton/pull/220/commits/22f32e13bef22be6ffc94f9dd481a0ca7df62790) | **accepted**   | Implemented as generalized dictionary operations with -1 signed flag indicating slice key |
| vec tool fix      | [-c readline disable](https://github.com/ton-blockchain/ton/pull/220/commits/4a1ea66bdbc84abd313f6419b8e0490bc9d08b54) | **accepted**   | Fixed in a slightly different way                            |
| func enhance      | [-u flags (warnings)](https://github.com/ton-blockchain/ton/pull/229) | *not yet...*   | Adding a `-u` and `-uu` flags that warn about unused variables and calls in func |
| func enhance      | [integer constants](https://github.com/ton-blockchain/ton/pull/227) | *not yet...*   | Adding a `const` keyword that allows to define global constants that get their value injected in-place in script instead of const itself |
| func enhance      | [include keyword](https://github.com/ton-blockchain/ton/pull/228) | *not yet...*   | Added `include` keyword that allows to include another func source file. |
| fift asm enh      | [unused procs opt](https://github.com/ton-blockchain/ton/pull/230) | ***accepted*** | Implemented in a little different way (although more logical) that I cant understand yet |
| fift enhance      | [error backtrace](https://github.com/ton-blockchain/ton/pull/231) | *not yet...*   | Added a `-t` flag that displays backtrace of error and if `-tt` is used stack dumps |
| func enhance      | [implicit functions](https://github.com/ton-blockchain/ton/pull/236) | *not yet...*   | Added `implicit` functions that do not need to be followed by args and even `()` tuple. |
| **ve bugfix**     | **[ve memory leak](https://github.com/ton-blockchain/ton/issues/235#issuecomment-581017463)** | *in prog...*   | Analyzing and tackling the memory leak in validator engine, at least one possible fix suggested in issue (very important issue) |
| *sys SC atk* C1P3 | [elector vote atk](https://github.com/ton-blockchain/ton/issues/240) | *reported*     | Possible attack on elector contract from validator with potential dire consequences |
| *ve bugfix* C1P3  | [ve dbgout atk](https://github.com/ton-blockchain/ton/issues/246) | *predated*     | Predated, but worth mentioning nevertheless                  |

There were some other ideas, that were trashed for one or another reason, due to low subjective usefulness, high complexity or unprepared infrastructure, you can get acquainted with them [here](doc/Unchosen.md), if interested.

Good luck to all participants and Happy New Year!



---



#### Mandatory page ~~wine~~ cellar

```
/------------------------------------------------------------------------\
| Created for: Telegram (Open Network) Blockchain Contest #2 (Freestyle) |
>------------------------------------------------------------------------<
| December 2019, second half (Happy New Year!)                           |
\------------------------------------------------------------------------/
```

No author information this time, because search engines may index too much.