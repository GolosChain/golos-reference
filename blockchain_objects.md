Objects
-------

In contrast to most cryptocurrency wallets, Golos has a different
model to represent the blockchain, its transactions and accounts. This chapter
wants to given an introduction to the concepts of *[objects](https://github.com/GolosChain/chainbase/blob/8b83ea5b9513cdfad3380a9b043ce1e93429207e/include/chainbase/chainbase.hpp#L116)* as they are used for [storage](storage_engine.md) management. Furthermore, we will briefly introduce the API and
show how to subscribe to object changes (such as new blocks or incoming
deposits). Afterwards, we will show how exchange may monitor their accounts and
credit incoming funds to their corresponding users.

Graphene blockchains do not contain addresses, but in this particular framework modification for Golos objects identified by a unique *id*, a *space* and a *version* in the form:

    version.space.id

Some examples:

    1.0.15   # version / consensus / id: 15
    1.0.105  # version / consensus / id: 105
    
    1.10.1    # version / account_by_key objects / key lookup object 
    
Default space for every in-consensus object is 0.

A programmatic description of all the objects used in this particular Golos implementation can be found in the [sources](https://github.com/GolosChain/golos/blob/golos-v0.17.0/libraries/chain/include/golos/chain/steem_object_types.hpp). 

Particular objects storage details can be found in the [Chainbase](https://github.com/GolosChain/chainbase/blob/8b83ea5b9513cdfad3380a9b043ce1e93429207e/include/chainbase/chainbase.hpp).