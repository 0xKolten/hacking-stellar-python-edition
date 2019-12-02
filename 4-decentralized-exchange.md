[Home](README.md) - [Chapter 1](1-accounts.md) - [Chapter 2](2-payments.md) - [Chapter 3](3-assets.md) - [Chapter 4](4-decentralized-exchange.md) - [Chapter 5](5-path-payments.md) - [Conclusion](6-conclusion.md) - [Bonus Chapter 1](bonus-xdr.md) - [Bonus Chapter 2](bonus-streaming.md)

## Chapter 4 - Stellar Decentralized Exchange

<div align="center"><img width="20%" src="imgs/decentralized-exchange.png"></div>
<br>

For the official documentation on the Stellar decentralized exchange (SDEX) - [go here](https://www.stellar.org/developers/guides/concepts/exchange.html).

Not only can you issue, send, and receive assets on Stellar - you can trade them on the Stellar decentralized exchange.

Along with account balances, the Stellar ledger also stores buy and sell offers made by accounts. Accounts can make offers using the ```Manage Buy Offer``` or ```Manage Sell Offer``` operations. When a user makes an offer, it is checked against the existing order book for that pair. If it doesn't cross an existing order, it is added to the orderbook until the offer is taken or cancelled. To understand some of the concepts, we'll explore the SDEX orderbook then submit an order of our own.

### The Orderbook

Horizon has an ```/order_book``` endpoint that can be used to retrieve data about specific trading pairs. All of the arguments can be found [here](https://www.stellar.org/developers/horizon/reference/endpoints/orderbook-details.html). In this example, we'll write a script that gets 5 bids and 5 asks for the USD (AnchorUSD) / XLM pair by providing the following arguments:

``` selling_asset_type=credit_alphanum4 ```

``` selling_asset_code=USD```

``` selling_asset_issuer=GDUKMGUGDZQK6YHYA5Z6AY2G4XDSZPSZ3SW5UN3ARVMO6QSRDWP5YLEX ```

``` buying_asset_type=native ```

``` limit=5 ```

``` python
import requests

def get_orderbook(selling_asset, selling_asset_issuer):
    # Create Horizon URL
    url = 'https://horizon.stellar.org/order_book?selling_asset_type=credit_alphanum4&selling_asset_code={}&selling_asset_issuer={}&buying_asset_type=native&limit=5'

    # Get JSON response
    r = requests.get(url.format(selling_asset, selling_asset_issuer))
    orderbook = r.json()

    # Iterate through bids
    for bid in orderbook['bids']:
        print("Bid -", "Price:", bid['price'], "XLM per USD")

    print('------------------------------------')

    # Iterate through asks
    for ask in orderbook['asks']:
        print("Ask -", "Price:", ask['price'], "XLM per USD")

if __name__ == '__main__':
    anchor_usd_address = 'GDUKMGUGDZQK6YHYA5Z6AY2G4XDSZPSZ3SW5UN3ARVMO6QSRDWP5YLEX'
    get_orderbook('USD', anchor_usd_address)

```

Based on current prices, your output should be similar to this:  
```
Bid - Price: 12.6614974 XLM per USD
Bid - Price: 12.6542233 XLM per USD
Bid - Price: 12.6381987 XLM per USD
Bid - Price: 12.6362663 XLM per USD
Bid - Price: 12.5612045 XLM per USD
------------------------------------
Ask - Price: 12.6995070 XLM per USD
Ask - Price: 12.7121870 XLM per USD
Ask - Price: 12.7375606 XLM per USD
Ask - Price: 12.7395001 XLM per USD
Ask - Price: 12.8136815 XLM per USD
```

**Note**: Try altering the script to grab information about other pairs you are interested in.

### The Ticker API

Another way to get data from the SDEX is to use the [Ticker API](https://medium.com/stellar-developers-blog/a-new-ticker-for-the-stellar-community-4ba7961e0759). It offers two endpoints: ```/assets.json``` and ```/markets.json```. The assets endpoint provides information about all assets available on the SDEX. The markets endpoint displays 24-hour, 7-day and orderbook information about markets that were active during these periods.

Here's an example script that gets some market data for the aggregated USD / XLM pair:

**Note**: When using market data from the ticker, assets from different issuers that have the same code are aggregated.

``` python
import requests

def get_market(name):
    # Get market data from Stellar ticker
    r = requests.get('https://ticker.stellar.org/markets.json')
    market = r.json()
    # Look for pair -> print information
    for pair in market['pairs']:
        if pair['name'] == name:
            print("Price:", round(pair['price'], 7), "XLM per USD")
            print("24hr Trade Count:", pair['trade_count'])
            print("24hr Price Change:", round(pair['change'], 7), "XLM")

if __name__ == '__main__':
    get_market('XLM_USD')
```

This script will return price data, a 24hr trade count, and the 24hr price change:
```
Price: 12.6229794 XLM per USD
24hr Trade Count: 141
24hr Price Change: 0.6819694 XLM
```

For more on the Ticker API check out the [documentation](https://github.com/stellar/go/blob/master/services/ticker/docs/API.md).

I've also written a similar tutorial on Medium called [Explore Stellar Addresses and the Stellar DEX using Python](https://medium.com/@kolten/explore-stellar-addresses-and-the-stellar-dex-using-python-e72611822b48), where I walk you through reading an account's lumen balance and converting it to USD using SDEX market data.  

### Creating an Offer

To expirement with creating an offer, I headed over to [testnet.interstellar.exchange](https://testnet.interstellar.exchange/app/#/markets/guest) (a UI for the testnet SDEX) to find an asset to trade. In this case I'll be trading my lumens for a token called "FEE," but you may choose any others that you find.

By looking at the order book, I noticed a user selling FEE tokens for 5.9 lumens each. For simplicity, I'll take the other side of the trade and buy 10 FEE tokens.

**Note**: Remember that in order to hold FEE tokens we have to trust the asset issuer provided by Interstellar.Exchange: ```GDZZDS3QVBP2UMO2FFNXUJLDAQHFNFUQSJSROI3HWVCPKZ7YIFMTXLLY```. In this example we'll use both a ```Change Trust``` operation and a ```Manage Buy Offer``` operation in our transaction.

Since ```Account B``` already has KOOL tokens from last chapter, I'll use ```Account A``` to hold the FEE tokens instead:

```
Public key: GD7YLRC3YWR3SMVGY3TSQ2UL56D7SG3JDIGYPNYZ2G22HBMOX5S7CLYF
Private key: SDRRZ2JQBI3RQSHV4Q63YWWJOVFNSXE2R6YSQKGUIPY65UATGUZUX4BB
```

As before, we'll use the Builder object provided by the Python Stellar SDK and append the two operations we need included in our transaction. ```.append_change_trust_op()``` should look familiar, fill in the relevant info to set a trustline for the FEE token. Using ```.append_manage_buy_offer_op()```, we can specify that we are selling lumens, buying FEE tokens, looking to buy 10 FEE tokens, and the price (in lumens) we are willing to pay per FEE token.

``` python
from stellar_base.builder import Builder
import json

def trust_and_buy(signing_key, asset_code, asset_issuer):
    builder = Builder(secret=signing_key, horizon_uri='https://horizon-testnet.stellar.org' , network='testnet') \
        .append_change_trust_op(asset_code, asset_issuer) \
        .append_manage_buy_offer_op(selling_code='XLM', selling_issuer=None, buying_code=asset_code, buying_issuer=asset_issuer, amount='10', price='5.9')
    # Sign and submit transaction -> print response
    builder.sign()
    response = builder.submit()
    print(json.dumps(response, indent = 2))

if __name__ == '__main__':
    trust_and_buy('SDRRZ2JQBI3RQSHV4Q63YWWJOVFNSXE2R6YSQKGUIPY65UATGUZUX4BB', 'FEE', 'GDZZDS3QVBP2UMO2FFNXUJLDAQHFNFUQSJSROI3HWVCPKZ7YIFMTXLLY')
```

After submitting the transaction and verifying that it was successful, we can repurpose our script that reads account balances and check ```Account A```'s:

```
Public key: GD7YLRC3YWR3SMVGY3TSQ2UL56D7SG3JDIGYPNYZ2G22HBMOX5S7CLYF
Last modified ledger: 101402
Lumen Balance: 9740.9999400
FEE Balance: 10.0000000
```

If your order was filled, you should now be the owner of 10 FEE tokens!

**Note**: There are many other ways to expirement and explore with the Stellar decentralized exchange that are not covered here. I highly encourage anyone interested in the SDEX to dive in to the documentation and APIs to get fully acclimated. You can also look around on SDEX UIs such as [StellarX](https://www.stellarx.com/markets) or [StellarPort](https://stellarport.io/exchange/) to see the SDEX in action.   

→ [Chapter 5 - Path Payments](5-path-payments.md)
