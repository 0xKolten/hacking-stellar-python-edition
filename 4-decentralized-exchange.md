[Home](README.md) - [Chapter 1](1-accounts.md) - [Chapter 2](2-payments.md) - [Chapter 3](3-assets.md) - [Chapter 4](4-decentralized-exchange.md) - [Chapter 5](5-path-payments.md) - [Chapter 6](6-xdr.md) - [Conclusion](7-conclusion.md)

## Chapter 4 - Stellar Decentralized Exchange

<div align="center"><img width="20%" src="imgs/decentralized-exchange.png"></div>
<br>

For the official documentation on the Stellar decentralized exchange (SDEX) - [go here](https://www.stellar.org/developers/guides/concepts/exchange.html).

Not only can you issue, send, and receive assets on Stellar - you can trade them on the Stellar [decentralized exchange](https://youtu.be/2L8-lrmzeWk).

Along with account balances, the Stellar ledger also stores buy and sell offers made by accounts. Accounts can make offers using the ```Manage Buy Offer``` or ```Manage Sell Offer``` operations. When a user makes an offer, it is checked against the existing order book for that pair. If it doesn't cross an existing order, it is added to the orderbook until the offer is taken or cancelled. To understand some of the concepts, we'll explore the SDEX orderbook then submit an order of our own.

**Note:** For some of the scripts below we will be using the [requests](https://requests.readthedocs.io/en/master/) library. 

### The Orderbook

For this section we don't need the Python SDK - we can just interface with the Horizon API directly. Horizon has an ```/order_book``` endpoint that can be used to retrieve data about specific trading pairs. All of the arguments can be found [here](https://www.stellar.org/developers/horizon/reference/endpoints/orderbook-details.html). In this example, we'll write a script that gets 5 bids and 5 asks for the USD (AnchorUSD) / XLM pair by providing the following arguments:

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

To expirement with creating an offer and to save some time, I headed over to [testnet.interstellar.exchange](https://testnet.interstellar.exchange/app/#/markets/guest) (a UI for the testnet SDEX) to find an asset to trade. In this case I'll be trading my lumens for a token called "STR," but you may choose any others that you find on the market.

By looking at the order book, I noticed I could buy about 933 STR for 1.988 XLM each. Sounds like an okay deal so I'll take it. 

**Note**: Remember that in order to hold STR tokens we have to trust the asset issuer provided by Interstellar.Exchange: ```GBEYFNS6KJRFEI22X5OBUFKQ5LK7Z2FZVFMAXBINC2SOCKA25AS62PUN```. In this example we'll use both a ```Change Trust``` operation and a ```Manage Buy Offer``` operation in our transaction. This concept of including multiple operations in one transaction is called *batching*. 

Since ```Account B``` already has HACK tokens from last chapter, I'll use ```Account A``` to hold the STR tokens:

```
Public key: GBG7D5ZZJLAKPDBAGSVS3O3TMIV2O3HOIOXE2OSGGCYNRATOICDRTIAR
Private key: SBK4EAZIWXELREKEXP4WB6DCCMJH7SGTEQE2BJALA32VQQ4ADFAWJGOV
```

Your script will look something like this. Don't worry, I'll explain how it works below: 

``` python
from stellar_sdk import Server, Keypair, TransactionBuilder, Network
from stellar_sdk.xdr import Xdr
from stellar_sdk.xdr.StellarXDR_const import TransactionResultCode
import json
import base64

def tx_success(result_xdr):
    xdr_decoded = base64.b64decode(result_xdr.encode())
    unpacker = Xdr.StellarXDRUnpacker(xdr_decoded)
    xdr_obj = unpacker.unpack_TransactionResult()
    print('Transaction result:', TransactionResultCode.get(xdr_obj.result.code))

def trust_buy_asset(signing_key, asset_code, asset_issuer, amount, price):
    # Talk to testnet horizon instance
    server = Server(horizon_url='https://horizon-testnet.stellar.org')

    # Derive Keypair object and public key from the signing key (source account)
    source_keypair = Keypair.from_secret(signing_key)
    source_public_key = source_keypair.public_key

    # Fetch the current sequence number for the source account from Horizon.
    source_account = server.load_account(source_public_key)

    # Build transaction
    transaction = (
        TransactionBuilder(
            source_account=source_account,
            network_passphrase=Network.TESTNET_NETWORK_PASSPHRASE,
            base_fee=100,
        )
        .append_change_trust_op(asset_code, asset_issuer)
        .append_manage_buy_offer_op('XLM', None, asset_code, asset_issuer, amount, price)
        .build()
    )

    transaction.sign(source_keypair)

    # Submit the transaction to Horizon and check if successful
    response = server.submit_transaction(transaction)
    result_xdr = response.get('result_xdr')
    tx_success(result_xdr)
    print(json.dumps(response, indent=2))

if __name__ == '__main__':
    """
    Account A
    Public key: GBG7D5ZZJLAKPDBAGSVS3O3TMIV2O3HOIOXE2OSGGCYNRATOICDRTIAR
    Private key: SBK4EAZIWXELREKEXP4WB6DCCMJH7SGTEQE2BJALA32VQQ4ADFAWJGOV
    """

    trust_buy_asset('SBK4EAZIWXELREKEXP4WB6DCCMJH7SGTEQE2BJALA32VQQ4ADFAWJGOV', 'STR', 'GBEYFNS6KJRFEI22X5OBUFKQ5LK7Z2FZVFMAXBINC2SOCKA25AS62PUN', '933', '1.988')
```

The ```trust_buy_asset()``` function takes a a few arguments that we'll need: the source account's ```signing_key```, STR's ```asset_code```, the ```asset_issuer``` address of STR, the ```amount``` of STR we want to buy, and the ```price``` we are willing to pay (in lumens). 

As before, we'll use the transaction builder provided by the Python SDK and append the two operations we need included in our transaction. ```.append_change_trust_op()``` should look familiar, fill in the relevant info to set a trustline for the STR token (or token of your choice - just like we did with the HACK token. 

Using ```.append_manage_buy_offer_op()```, we can specify that we are selling lumens, buying STR tokens, looking to buy 933 STR tokens, and the price (in lumens) we are willing to pay per STR token. You'll notice that after specifying that we selling lumens, we pass in ```None```. This is because lumens *do not* have an issuing address like other assets. 

**Remember:** It is only necessary to trust assets once. For the sake of simplicity I included a change trust operation in the script below even though this is not the most optimal set up. 

After submitting the transaction and verifying that it was successful, we can read the balance of ```Account A``` and find that we are now the owner of 933 STR tokens: 

```
STR balance: 933
XLM balance: 8,045.19596 
```

**Note**: There are many other ways to expirement and explore with the Stellar decentralized exchange that are not covered here. I highly encourage anyone interested in the SDEX to dive in to the documentation and APIs to get fully acclimated. You can also look around on SDEX UIs such as [StellarX](https://www.stellarx.com/markets) or [StellarPort](https://stellarport.io/exchange/) to see the SDEX in action.   

→ [Chapter 5 - Path Payments](5-path-payments.md)
