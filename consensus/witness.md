## Witness

This document serves as an introduction on how to become an actively block producing witness in in the Golos network.

In Golos, the witnesses' job is to collect transactions, bundle them into a block, sign the block and broadcast it to the network. They essentially are the block producers for the underlying consensus
mechanism.

For each successfully constructed block, a witness is payed in shares that are taken from the limited reserves pool at a rate that is defined by the shareholders by means of approval voting.

Witnesses - are network participants guarantee the data is safely handled. So we will consider starting own witness node.

 Currently active public test network is `https://testnet.golos.io` with WebSocket endpoint on `wss://ws.testnet.golos.io` and replication (seed) endpoint on `seed.testnet.golos.io`. 
 
This article purpose is to describe the witness initialization.

### Download the genesis block (only for main network)

For the main network we need to download the proper genesis block. The genesis block can be found in `share/golosd/snapshot5392323.json`.

### Run a node in the network

It is required first to run the node without block production and connect it to the P2P network with the following command in case of test network connection:

    $ programs/golosd/golosd -s seed.testnet.golos.io --rpc-endpoint 127.0.0.1:8090
        
As for main Golos network the following command would be fine:  

    $ programs/golosd/golosd -s seed.golos.io --rpc-endpoint 127.0.0.1:8090 --snapshot-file=snapshot5392323.json

### Creating a wallet
We now open up the cli_wallet and connect to our plain and stupid witness node:

    $ programs/cli_wallet/cli_wallet -s ws://127.0.0.1:8090

First thing to do is setting up a password for the newly created wallet prior to importing any private keys:

    new >>> set_password <password>
    null
    locked >>> unlock <password>
    null
    unlocked >>>

Wallet creation is now done.

### Basic Account Management
We can import the account name (owner key):

    unlocked >>> import_key <owner wif key>
    true
    unlocked >>> list_my_accounts
    [{
    "name": <accountname>,
    ...
    ]
    unlocked >>> list_account_balances <accountname>
    XXXXXXX GOLOS

In case your account's owner key is different from its active key, make sure you import it into Golos as well.

### Becoming a Witness
To become a witness and be able to produce blocks, you first need to create a witness object that can be voted in with your account name mentioned.

`update_witness "cyberfounder" "https://golos.io/your_witness_election_race_post" GLS58g5rWYS3XFTuGDSxLVwiBiPLoAyCZgn6aB9Ueh8Hj5qwQA3r6 {"account_creation_fee":"3.000 GOLOS","maximum_block_size":131072,"sbd_interest_rate":1000,"asset_creation_fee":"5.000 GBG"} true`

That means we have now a witness object for account name `cyberfounder` with witness election advertisement post at `https://golos.io/your_witness_election_race_post` (you would need to advertise yourself to voted for huh?), your public part of a key you would use to sign a block and some proposed chain parameters.

Our witness is registered, but it can't produce blocks because nobody has voted
it in.  You can see the current list of active witnesses with
`get_active_witnesses`:

    unlocked >>> get_active_witnesses
    {
      "active_witnesses": [
    "1.6.0",
    "1.6.1",
    "1.6.2",
    "1.6.3",
    "1.6.4",
    "1.6.5",
    "1.6.7",
    "1.6.8",
    "1.6.9"
      ],
      ...

Now, we should vote our witness in.  Vote all of the shares in both
`<accountname>` and `cyberfounder` in favor of your new witness.

    unlocked >>> vote_for_witness <accountname> <accountname> true true
    [a transaction in json format]

Get the witness object using `get_witness` and take note of two things. The
`id` is displayed in `get_active_witnesses` when the witness is voted in, and
we will need it on the `golosd` command line to produce blocks.  We'll
also need the public `signing_key` so we can look up the correspoinding private key.

Once we have that, run `get_private_key <public-key>` which lists correspoding private-key.

Warning: `get_private_key` will display your keys unencrypted on the
terminal, don't do this with someone looking over your shoulder.

    unlocked >>> get_witness <accountname>
    {
      [...]
      "signing_key": "GLS7vQ7GmRSJfDHxKdBmWMeDMFENpmHWKn99J457BNApiX1T5TNM8",
      [...]
    }

The `signing_key` is the most important parameter, here. Let's get the private key for that signing key with:

    unlocked >>> get_private_key <signing_key>
    

Now we need to start the witness, so shut down the wallet (ctrl-d),  and shut
down the witness (ctrl-c).  Re-launch the witness, now mentioning the new
witness `<account_name>` and its keypair in case of main Golos network:

    ./golosd --rpc-endpoint=127.0.0.1:8090 --witness="<account_name>" --private-key="5JVFFWRLwz6JoP9kguuRFfytToGU6cLgBVTL9t6NB3D3BQLbUBS" --snapshot-file=snapshot5392323.json -s seed.golos.io
    
As for test network: 
    
    ./golosd --rpc-endpoint=127.0.0.1:8090 --witness="<account_name>" --private-key="5JVFFWRLwz6JoP9kguuRFfytToGU6cLgBVTL9t6NB3D3BQLbUBS" -s seed.testnet.golos.io

Alternatively, you can also add this line into your config.ini:

    witness = cyberfounder
    private-key = 5JVFFWRLwz6JoP9kguuRFfytToGU6cLgBVTL9t6NB3D3BQLbUBS

If you monitor the output of the `golosd`, you should see it generate 
blocks signed by your witness:

    Witness production slot has arrived; generating a block now...
    Generated block #367 with timestamp 2015-07-05T20:46:30 at time 2015-07-05T20:46:30