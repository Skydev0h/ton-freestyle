# TON freestyle contest entry

[*(Blockchain competition number 2 - Official requirements and documentation)*](doc/Official.md)

This package contains a participation entry for second blockchain contest of TON for creating own freestyle smart contracts. 

Official description of objectives, rules and requirements is done on [the referenced page](doc/Official.md). 

As per project ideas themselves, **two** smart contract projects were chosen to be implemented:

* [**Conditional transfer smart contract**](doc/Project1.md)
* [**Data storage proxy smart contract**](doc/Project2.md)

More details about those ideas should be read on their respective pages linked above.

Update on tests: the changes have been actually **added to the master TON branch** by the TON maintainers, so no patch is needed to utilize test suite to full extent.

Summarizing proposed changes and fixed in [this pull request](https://github.com/ton-blockchain/ton/pull/220) from [this fork](https://github.com/Skydev0h/ton):

| Type of change | Name                | Status       | Description                                                  |
| -------------- | ------------------- | ------------ | ------------------------------------------------------------ |
| fift enhance   | (gas)runvmctxact(q) | **accepted** | Added as generalized runvm function that accepts bit flags, fift compatibility layer includes the proposed functions |
| runvm fix      | commited states     | **accepted** | Fix implemented in runvm code                                |
| fift enhance   | dict slice key ops  | **accepted** | Implemented as generalized dictionary operations with -1 signed flag indicating slice key |
| vec tool fix   | -c readline disable | **accepted** | Fixed in a slightly different way                            |
| func enhance   | -d flag (warnings)  | *not yet...* | Adding a -d flag that warns about any optimized out calls in func |

Quick-start instructions about using those contracts can be found [here](doc/Quick.md).

Technical details about implementations are on their corresponding pages.

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