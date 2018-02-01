Manually Construct Any Transaction
==================================

General Procedure
-----------------

The general principle for generating, singing and broadcasting an
arbitrary transactions works as follows:

1.  Create an instance of the transaction builder
2.  Add arbitrary operation types
3.  Add the required amount of fees
4.  Sign and broadcast your transaction

The corresponding API calls in the cli_wallet are:

    >>> begin_builder_transaction
    >>> add_operation_to_builder_transaction $HANDLE [opId, {operation}]
    >>> sign_builder_transaction $HANDLE true

The begin\_builder\_transaction call returns a number we call `$HANDLE`
It allows to construct several transactions in parallel and identify
them individually!

The opId and the JSON structure of the operation can be obtained with:

    get_prototype_operation <operation-type>

Example: Transfer
-----------------

A simple *transfer* takes the following form:

    get_prototype_operation transfer_operation
    [
      0,{
        "from": "cyberfounder",
        "to": "nemo1369",
        "amount": {
          "amount": 0,
          "name": "GOLOS",
          "precision": 3
        },
        "extensions": []
      }
    ]

The operation id for the `transfer_operation` is thus `0` (third line)
and the core elements (removing fee) of this operation take the form:

``` {.sourceCode .js}
{
  "from": "cyberfounder",
  "to": "nemo1369",
  "amount": {
    "amount": 0,
    "name": "GOLOS",
    "precision": 3
  }
}
```

We add an operation to a transaction as follows (line breaks inserted
for readability):

    >>> begin_builder_transaction
    0
    >>> add_operation_to_builder_transaction
         0
        [0,{
               "from": "cyberfounder",
                       "to": "nemo1369",
                       "amount": {
                         "amount": 0,
                         "name": "GOLOS",
                         "precision": 3
                       },
           }]

The corresponding `id` can be obtained with `get_account`, and
`get_asset`.

We add a fee payed in GOLOS, sign and broadcast the transaction (if
valid):

    >>> set_fees_on_builder_transaction 0 GOLOS
    >>> sign_builder_transaction 0 true
