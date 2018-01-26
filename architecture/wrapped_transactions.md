Wrapped transactions
--------------------

- This is `theoreticalbts` idea for an interesting feature

In traditional exchanges, unfilled orders are free -- market fees are only charged on matched orders.  We have to charge a minimal
amount per unfilled order as anti-spam measure.  However, we can imagine an e(x)change provider (X)avier who hosts orders on
an external server.  When Alice wants to place an order, she creates an order transaction with no fee, then uploads the order to
Xavier's server; Xavier publishes it (and Xavier will need to implement alternative anti-spam measures to protect his server from
abuse).

When Bob wants to match Alice's order, he provides the fee.

Here's my idea for how to implement this without substantially re-working the fee structure.  We create a special "community" account
(TODO:  better name) with a special flag which signals that *no authority* is needed to withdraw funds from it.  Alice signs her tx
paying the fee from the community account, now the only reason her tx is invalid is because the community account has no funds.  Now
Bob can create a *wrapping transaction* containing his matching order, funding for the community account, and Alice's tx.  The wrapped
tx is signed by Bob.  Crucially, doing it this way means no one can insert a tx taking the money from the community account in between
Bob's operation funding the community account and Alice's transaction paying it.

Can we do this with proposed tx's?  We have to think very carefully about the exact semantics of proposed tx's.