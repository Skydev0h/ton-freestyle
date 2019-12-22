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

