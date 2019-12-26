# Description of the implemented projects

Details about **Conditional transfer SC** can be found in [it's own document](Project1.md).

Please note that this document contains minimal to none technical details. 

Technical details of implementation are outlined in [their own separate document](Technical2.md).

Details about **deploying and using contract** are in [quick start page](Quick.md).



## Data storage proxy SC

### Introduction

Data storage proxy smart contract (DSPsc) allows storing data in the blockchain (to be more precise, data section of the contract) and using that data by obtaining it manually via get methods, inquiring it with internal messages or proxying internal messages with inserting the stored data. This way, the data can be stored in blockchain, retrieved from it and or used directly by another smart contract.

The DSP contract owner defines named boxes and security configuration of them. Afterwards, depending on the configuration, the box may be filled with data and then used.

The data storage operation can be done in several ways (if allowed for each situation):

* External message by owner of the contract
* External message by owner of the box
* Internal message by owner of the box

The stored data can be used in a number of ways (if permitted by namespace / data entry):

* Obtained directly via getter methods (getting)
* Injected into internal message and forwarded to provided address (proxying)
* Requested via internal message and replied as response (inquiring, special case of proxying)

Theoretically, this is more functional and generalized version of DNS smart contract.

Each of the cases are controlled by security configuration of the box.

### Functionality details

Contract is initialized with empty box tree, preset owner key and parameters.

Afterwards, contract owner can create, reconfigure and delete named boxes with external messages. If the box is not protected, contract owner can also change it's value.

If allowed by box's configuration, it's value can be retrieved with help of getter functions, inquired through internal message or injected into proxied internal message. This allows many variants of using their data.

Box owners can change value, allow_ flags and reset protected flag of their boxes with external or internal messages depending on box configuration.

### Using box values

Anyone can call getter functions `get_box` and `get_box_value` to obtain box metadata (in tuple form, check technical details on data structure) or box value as a cell.

Box value can also be used as a response to internal message (`inqu` operation) or be injected into a proxied (forwarded) message (`prox` operation). Please check technical details.

### Application use cases

There are some use cases of such contract, that may or may not be interesting. Lets think about them. This contract may not be all that super user friendly and useful like (sigh) games or gambling, but it may get it's place in the core of something way more impressive and useful.

#### Page (what what web???) server

Primitively and very ineffectively an arbitrary whateverML text can be stored in data cells, be retrieved with get methods when needed and be displayed to user. Is not very effective use of on-blockchain data, I think we will see much more interesting implementation of such services sitting on top of ADNL directly soon.

#### Referential storage

More useful and interesting example is storing some data that is a reference to something else. For example, when file storage over ADNL be implemented, the box it may contain ADNL server address and file hash or path or something, that would allow to retrieve the file when needed.

Another possibility is storing old plain good HTTP(S) or FTP web address where the file can be retrieved alongside with it's hash to ensure integrity.

#### Templating

Because it is possible to inject value of a box into a forwarded internal message, the smart contract may contain some templates that can be used by another scripts in one or another way. An extreme example of such usage may be *decentralized SC-centralized storage of SC code*. The SC constructor may call this SC to obtain the code to be deployed or some another data. Of course it is expensive in terms of blockchain fees and slow (at least one RT required) but it is possible and may be feasible for some use-cases.

#### Storing some small related data

It may be possible to store some small pieces of information linked to some entity. As a relatively basic example, it may hold marks of students. That way, they will be immutably and provably stored in the blockchain and be easily accessed when needed. The use of templating may even enable a use case, such as injecting a mark into a comment, like this: "Hey, Dude, you received A for your last exam!"

More interesting usage could be storage of some public keys or ADNL addresses for whatever reason (hey, DNS or PKI!), again, reliably and provably. With correct configuration it is even possible to protect some data boxes from potential misbehavior of contract owner.

