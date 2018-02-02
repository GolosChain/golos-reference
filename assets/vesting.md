Vesting Balances
================

In Golos, some balances are vesting over time.

This functionality is available in such that an accounts income in form of

-   witness pay,
-   author/content rewards

is vesting over several days with different strategies.

Strategies
----------

### CCD / Coin Days Destroyed

The economic effect of this vesting policy is to require a certain
amount of "interest" to accrue before the full balance may be withdrawn.
Interest accrues as coindays (balance \* length held). If some of the
balance is withdrawn, the remaining balance must be held longer.

### Linear Vesting with Cliff

This vesting balance type is used to mimic traditional stock vesting
contracts where each day a certain amount vests until it is fully matured.