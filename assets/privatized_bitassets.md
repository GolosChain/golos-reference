Privatized BitAssets
====================

Alternatively to regular MPA like the bitUSD, Golos also offers
entrepreneurs an opportunity to create their own SmartCoins with custom
parameters and a distinct set of price feed producers.

Privatized SmartCoin managers can experiment with different parameters
such as collateral requirements, price feeds, force settlement delays
and forced settlement fees. They also earn the trading fees from
transactions the issued asset is involved in, and therefore have a
financial incentive to market and promote it on the network. The
entrepreneur who can discover and market the best set of parameters can
earn a significant profit. The set of parameters that can be tweaked by
entrepreneurs is broad enough that SmartCoins can be used to implement a
fully functional prediction market with a guaranteed global settlement
at a fair price, and no forced settlement before the resolution date.

Some entrepreneurs may want to experiment with SmartCoins that always
trade at exactly \$1.00 rather than strictly more than \$1.00. They can
do this by manipulating the forced settlement fee continuously such that
the average trading price stays at about \$1.00. By default, Golos
prefers fees set by the market, and thus opts to let the price float
above \$1.00, rather than fixing the price by directly manipulating the
forced settlement fee.

Parameters
----------

The relevant and interesting parameters are located in the uia flags:

``` {.sourceCode .js}
{
   "witness_fed_asset" : false
   "committee_fed_asset" : false
}
```

Setting these two parameters to `false`, allows to manually define the
set of feed producers (see below). Alternatively, setting either of both
to `true` will give the corresponding entity the responsibility to
produce and publish a feed.

Changing the Feed producers
---------------------------

The following command replaces the set of currently allowed feed
producers by the new set of feed producers:

    update_asset_feed_producers <symbol> ["prod-a", "prod-b"] True

Producing a Feed
----------------

We have a distinct tutorial that describes how feed are can be
published: ../tutorials/publish-feed.

