Architecture
------------

Graphene is designed to allow multiple *applications* to connect to the network.  An *application* consists of a p2p node
which receives blocks and optionally broadcasts new transactions on behalf of the application's users.  Applications contain
core logic which consists of *core chain state* and *core indexes*, which is the minimal amount of data necessary to reach
*core consensus*, shared global agreement among all nodes participating in the p2p network about *core features*.

Core virtual property
---------------------

*Virtual property* consists of the things that can be owned.  Traditional cryptocurrencies, such as Bitcoin, have a
single type of virtual property:  A digital token which is transferrible, fungible (e.g. all Bitcoins are identical),
and (for practical purposes) divisible.

Graphene has two different types of digital property:  *Base* property, and *derived* property.
(Economists would call base property *fiat*, but this term is already used in the cryptocurrency space as a retronym for
government-issued currency, used to differentiate it from cryptocurrency.)

The base property in Graphene core consists of:

- Accounts
- Core asset
- UIA's

Derived property in Graphene core consists of:

- BitAssets
- Bonds
- Options
- Escrows
- Other types of derived property

Applications which leverage the Graphene blockchain for consensus may implement their own base and/or derived virtual property.
(TODO:  explain how application-level virtual property may interact with core property.)

Derived property is created by *core smart contracts* which are hard-coded in Graphene.  Smart contracts can be as diverse as
real-world contracts, but in Graphene, all core smart contracts are collateralized, two-party contracts.  One side of the contract
must post *collateral* which is used to perform settlement.  The party which posts collateral is referred to as the *long side*.

| Contract type | Long name     | Long fungible? | Long transferrable? | Short name       | Short fungible? | Short transferrable? |
| ------------- | ------------- | -------------- | ------------------- | ---------------- | --------------- | -------------------- |
| BitAsset      | Long holder   | Yes            | Yes                 | Short holder     | No              | No                   |
| Bond          | Lender        | No             | Yes                 | Borrower         | No              | Yes                  |
| Option        | Option holder | ?              | Yes                 | Option writer    | ?               | Yes                  |

A word about the rationale for short BitAssets not being transferrable:  There is little technical obstacle to making short positions transferrable.
However, if Alice is a weakly collateralized short seeking to exit, and Bob is a new short seeking to enter at high leverage, Alice and Bob both
have incentive for her to sell her position to him.  It is desirable to force Alice to cover and force Bob to short at the minimum initial leverage,
to re-capitalize shorts over time.

Differences from BitShares
--------------------------

- Shorting mechanics.  Shorts have maintenance margin requirement and user-settable stop-loss.  TODO:  Document exactly how this works.

- Incremental order matching.  Orders match incrementally when entered.  TODO:  Document exactly how this works.

- Traditional price splitting.  If orders match at a spread, they split the surplus 50-50.  TODO:  Document more exactly, including example.

- Fill-or-kill orders.  TODO:  How do these work?

- BitAssets 3.0.  TODO:  Turn forum posts, etc. into detailed spec including examples.

- TaPoS.  TODO:  Explain this.

- Short expiration time.  TODO:  Explain this.

- Multisig uses authority system.

Account porting gateways
------------------------

- This is `theoreticalbts` idea for an interesting feature

This can also be used for third-party "account porting gateways".  E.g. let's say we have `email` account and want to
set up a way where anyone with an email address can claim the email address in BitShares form.  So for example
`email/user_at_example_dot_com` would be given to someone who proves they control `user@example.com`.  The holder of
the `email` GOLOS account, and *not* core witnesses / validation, is responsible for checking this proof (which may
require arbitrary off-blockchain actions that cannot be validated in a non-decentralized way, like sending confirmation
emails).  The validation consists of signature checking ("user `email` confirmed pubkey `p` owns `email/user_at_example_dot_com`")
combined with the `email` account's policy ("we require successful response to registration mail before giving out names")
combined with trust in `email` account (by sending money to account `email/user_at_example_dot_com`, user is trusting `email`
account honestly associated `user_at_example_dot_com` with the correct person).

This method can also be applied to any website which has a userbase that has a login API (Github / Twitter / LinkedIn / etc.),
or even merely the ability for members to post content (e.g. in forum profile) -- if you give a user a challenge and they
successfully post it in their profile or other publication area, they've successfully confirmed access to that account.

With wrapped transactions, the account porting gateway can create a no-fee tx assigning the name to the user, the user then
adds the fee to actually register the account.  This allows the account porting gateway to avoid having to solve the
*economic* problem of determining which registrations will result in profitable CLV, and focus solely on the *technical*
problem of verifying the owner of an existing name in a third-party system like email, DNS, Google accounts, etc., while
still claiming referrer fees.  Admittedly there's no "free lunch," the UX is a little more rocky because the user
needs to provide their own funding.

Account revocation
------------------

- This is `theoreticalbts` idea for an interesting feature

There needs to be an "I lost my email address and private keys" button which allows `email` to revoke the name `email/user_at_example_dot_com`.
However, the underlying account should still exist, it just needs to be unlinked from the name (this way if `user` later finds
their private keys, they still have access to funds).  The name should be unable to be reassigned until a long enough delay which
at least allows transactions with TaPoS before the revocation block become invalid (otherwise if Alice sends to `user_at_example_dot_com`
and Eve controls `email` and a single witness, Eve can have her witness censor Alice's tx inclusion in the block, instead including a tx
assigning `email/user_at_example_dot_com` to herself, and taking the funds when the tx appears in a later block).

NB, this attack may be more difficult in practice, because clients do lookup of name-to-account-ID mapping locally.  It still makes sense
to have a revocation period -- this way if you've heard from someone in the last 30 days that their address is `email/user_at_example_dot_com`,
then you know you'll either send the funds to the right person or get an error.  Also, wallets should warn if one of your contacts
has been revoked and reassigned.

It also complicates account history, as the name displayed will be determined by the mapping for the name at the time the tx was performed.

Assertion ops
-------------

- This is `theoreticalbts` idea for an interesting feature

An *assertion op* is an operation which invalidates a transaction unless the *asserted condition* holds.
So far we have:

- TaPoS assertion.  Asserts that a particular block hash exists in the history.  All transactions have this
assertion, it prevents transactions from migrating to forks where ID's have different objects.

These may need to be included:

- Data object assertion.  We may be required to check a data object which has an ID, owner and custom content.
We may assert a data object exists with the given ID and owner, and some function of the content is true.

Examples of data object assertion functions:  Assert that the content follows a certain byte pattern (including gaps),
assert the content is a Merkle / Patricia proof that x is in S, assert some hash of the content is a given value.
These can be combined to construct an AXCT smart contract, and the latter is also useful for name-blinding schemes.

- Date assertions.  Not-valid-before or not-valid-after.  Every tx has expiration (not-valid-after) in the near future,
and can be not-valid-before some point in the recent past via TaPoS.  However it may be useful to extend these windows,
e.g. create a transaction not-valid-before some time in the future (this is `nLockTime` in Bitcoin).  Or create a transaction
not-valid-after some time in the *far* future.  A mechanism for more flexible date assertions than the default
TaPoS / expiration should be provided.  It is acceptable to require the use of proposed transactions for this mechanism.

- Authority assertions.  An *authority assertion* is an op which adds the given authority to the list of authority
required to make the transaction valid.  Authority assertion is useful to make platform actions and app actions
atomic.  E.g. Alice and Bob want to create a transaction to trade her AppCoin for Bob's BitUSD.  The tx includes a
custom op interpreted by the app layer as transfer of Alice's AppCoins to Bob, which the app doesn't honor without
Alice's signature; and a regular (platform) transfer op sending Bob's BitUSD to Alice.  Without an authority assertion,
Alice's signature also needs to be a custom op, which also means Alice has to sign first.  (If Alice used a regular
signature, Eve would be able to play the role of Alice, strip her own signature and apply Bob's signature, taking
Bob's BitUSD without compensating him with her AppCoins.)  Authority assertions mean that app signatures don't have
to be wrapped in this way, and allows tx to be signed in any order.