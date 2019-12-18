[Home](README.md) - [Chapter 1](1-accounts.md) - [Chapter 2](2-payments.md) - [Chapter 3](3-assets.md) - [Chapter 4](4-decentralized-exchange.md) - [Chapter 5](5-path-payments.md) - [Conclusion](6-conclusion.md) - [Bonus Chapter 1](bonus-xdr.md) - [Bonus Chapter 2](bonus-streaming.md)

## Chapter 3 - Assets on Stellar

<div align="center"><img width="25%" src="imgs/assets.png"></div>
<br>

For the official documentation on assets - [go here](https://www.stellar.org/developers/guides/concepts/assets.html).

One unique feature of the Stellar network is that it can be used to trade, hold, and transfer any type of asset. If you can tokenize it or represent it as a number in a database, it can be issued as credit and used on Stellar (e.g. dollars, cryptocurrencies, security tokens, etc.).

All assets on Stellar have an **asset code** (e.g. USD) and an **issuer account** (who created the token). Lumens are the only asset that *do not* have an issuer account. Assets can also have flags associated with them such as ```AUTHORIZATION REQUIRED``` and ```AUTHORIZATION REVOCABLE```. These flags allow asset issuers to have more control over their issued assets to adhere with compliance, improve customer security, etc.

Before we create an asset there's two concepts we should understand first: **anchors** and **trustlines**.

### Anchors (Asset Issuers)

In simplest terms, anchors are entities that issue assets on Stellar. [Anchors](https://github.com/koltenb/awesome-stellar/blob/master/README.md#stellar-asset-issuers-anchors) can range from individuals to commercial banks. Generally, anchors take deposits from users and issue those deposits on the Stellar network as credit. For example, I can deposit $100 in to [AnchorUSD](https://www.anchorusd.com/) and receive the equivalent amount of [USD tokens/credit](https://stellar.expert/explorer/public/asset/USD-GDUKMGUGDZQK6YHYA5Z6AY2G4XDSZPSZ3SW5UN3ARVMO6QSRDWP5YLEX) on Stellar. When I'm ready to redeem those USD tokens I can withdraw them back in to my bank account through AnchorUSD.

### Trustlines

Continuing the AnchorUSD example: When I deposit $100 to AnchorUSD in exchange for tokens/credit on Stellar, I am ***trusting*** that I will be able to redeem them later for $100. This idea of trust is conceptualized and made explicit on Stellar through **trustlines**.

In order to hold an asset on Stellar, you must explicitly trust the asset issuer through a trustline. The ```Change Trust``` operation is used to create, update, or delete trustlines. After a trustline is set, it will track the limit for which your account trusts the issuing account and the amount of credit from the issuing account that your account currently holds. Trustlines ensure that you only interact with assets that you have faith in. You don't want bad actors sending counterfit USD, or any other asset, to your account that you will be unable to redeem later.

**Note**: Since they persist on the ledger, trustlines have a **base reserve fee** of 0.5 lumens each. You will get the fee back after closing the trustline.

### Issuing an Asset

Now that you understand the concepts around Stellar assets, let's create one of our own.

We already have 2 accounts created so we'll use them again. ```Account A``` and ```Account B``` will be repurposed to create a typical anchor set up. In this set up will have an *issuing account* and a *distributing account*. 

```
Account A
Public key: GD7YLRC3YWR3SMVGY3TSQ2UL56D7SG3JDIGYPNYZ2G22HBMOX5S7CLYF
Private key: SDRRZ2JQBI3RQSHV4Q63YWWJOVFNSXE2R6YSQKGUIPY65UATGUZUX4BB
```
```
Account B
Public key: GACNGVOSMX7NUBKUEPU26FQ2ROQZRVZ6IGGPESEHNGEKNRS55OUWU2YG
Private key: SBKNPGV4QOCU6CB4GEUF76LVFJFMBWHY4JVIVYEUB76HYQAG7HXLLD77
```

The main reasons for creating two seperate accounts are to seperate the issuance and distribution logic, and to provide transparency in to the outstanding asset supply. 

The first thing we will need to do is set a trustline for the asset we would like to create from the *distrubtor account* to the *issuing account*. The script will look very similar to the one we used to send a payment. Instead of appending a payment operation we must append a change trust operation.  

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

def trust_asset(signing_key, asset_code, asset_issuer):
    # Talk to testnet horizon instance
    server = Server(horizon_url="https://horizon-testnet.stellar.org")

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

    Account B
    Public key: GCF4PQGCOFR245LDPRMBDGMD7VQMOGF2KQZJAWICB6JJ337NDPUQR66E
    Private key: SCO4JEHDN2ZLIDZMZC62UR6Y5NICMTMRKBPG3JMBKI5AHVXJA46MY2VG
    """

    trust_asset('SCO4JEHDN2ZLIDZMZC62UR6Y5NICMTMRKBPG3JMBKI5AHVXJA46MY2VG', 'HACK', 'GBG7D5ZZJLAKPDBAGSVS3O3TMIV2O3HOIOXE2OSGGCYNRATOICDRTIAR')
```

All of this should look familar from the last chapter. The only thing we changed is the function name, its parameters, and appended a new type of operation. In order to fulfill the needs of ```.append_change_trust_op(asset_code, asset_issuer)``` our function only needs to take the source account's *private key*, *asset code*, and the asset issuer's *public key*. 

In the above example, I created a token called ```HACK``` and specified that the distribution account (Account B) *trusts* the issuing account (Account A) to issue that asset. As long as the trustline persists, the issuing account will be able to send the distribution account ```HACK``` tokens. 

When you run the script, you should get a response like this: 

``` json
Transaction result: txSUCCESS
{
  "_links": {
    "transaction": {
      "href": "https://horizon-testnet.stellar.org/transactions/ee930d3269e316e9b1529059c97b6e6582e84b978d1cb44290b7fc7bbd79fa19"
    }
  },
  "hash": "ee930d3269e316e9b1529059c97b6e6582e84b978d1cb44290b7fc7bbd79fa19",
  "ledger": 780718,
  "envelope_xdr": "AAAAAIvHwMJxY651Y3xYEZmD/WDHGLpUMpBZAg+Sne/tG+kIAAAAZAAIhT0AAAABAAAAAAAAAAAAAAABAAAAAAAAAAYAAAABSEFDSwAAAABN8fc5SsCnjCA0qy27c2Irp2zuQ65NOkYwsNiCbkCHGX//////////AAAAAAAAAAHtG+kIAAAAQJNRQ7ENvP73jtnsywKJMUc4cfhlBaeCejVRzzu70DFdJF2iDHhui75PnjiPfB+FI8BU9h3lPrK2cOUXzTE6ZQY=",
  "result_xdr": "AAAAAAAAAGQAAAAAAAAAAQAAAAAAAAAGAAAAAAAAAAA=",
  "result_meta_xdr": "AAAAAQAAAAIAAAADAAvprgAAAAAAAAAAi8fAwnFjrnVjfFgRmYP9YMcYulQykFkCD5Kd7+0b6QgAAAAXhBGxnAAIhT0AAAAAAAAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAAAAAAAAAAAAAAAABAAvprgAAAAAAAAAAi8fAwnFjrnVjfFgRmYP9YMcYulQykFkCD5Kd7+0b6QgAAAAXhBGxnAAIhT0AAAABAAAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAAAAAAAAAAAAAAAABAAAAAwAAAAMAC+muAAAAAAAAAACLx8DCcWOudWN8WBGZg/1gxxi6VDKQWQIPkp3v7RvpCAAAABeEEbGcAAiFPQAAAAEAAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAEAC+muAAAAAAAAAACLx8DCcWOudWN8WBGZg/1gxxi6VDKQWQIPkp3v7RvpCAAAABeEEbGcAAiFPQAAAAEAAAABAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAAAC+muAAAAAQAAAACLx8DCcWOudWN8WBGZg/1gxxi6VDKQWQIPkp3v7RvpCAAAAAFIQUNLAAAAAE3x9zlKwKeMIDSrLbtzYiunbO5Drk06RjCw2IJuQIcZAAAAAAAAAAB//////////wAAAAEAAAAAAAAAAA=="
}
```

Lastly, we'll repurpose our payment script to send KOOL Tokens, instead of lumens, from ```Account A``` to ```Account B```:

**Note**: Since we aren't sending lumens, we must specify the asset code and who the asset issuer is when adding the payment operation.

``` python
from stellar_base.builder import Builder
import json

def send_asset(signing_key, receiving_key, amount):
    builder = Builder(secret=signing_key, horizon_uri='https://horizon-testnet.stellar.org' , network='testnet') \
        .append_payment_op(destination=receiving_key, asset_code='KOOL', amount=amount, asset_issuer='GD7YLRC3YWR3SMVGY3TSQ2UL56D7SG3JDIGYPNYZ2G22HBMOX5S7CLYF') \
    # Sign and submit transaction -> print response
    builder.sign()
    response = builder.submit()
    print(json.dumps(response, indent = 2))

if __name__ == '__main__':
    send_asset('SDRRZ2JQBI3RQSHV4Q63YWWJOVFNSXE2R6YSQKGUIPY65UATGUZUX4BB', 'GACNGVOSMX7NUBKUEPU26FQ2ROQZRVZ6IGGPESEHNGEKNRS55OUWU2YG', '100')
```

Finally, we can adjust the script we used to read account balances to include the KOOL Token balance. Since ```Account B``` should only have two assets (lumens and KOOL Tokens), we can grab the first balance on the list to get the KOOL Token balance:

``` python
from stellar_base.address import Address

def show_address_data(public_key):
    address = Address(address = public_key, secret = None, network='testnet')
    address.get()

    print("Public key:", address.id)
    print("Last modified ledger:", address.last_modified_ledger)
    print("Lumen Balance:", address.balances[-1]['balance'])
    print("KOOL Balance:", address.balances[0]['balance'])

if __name__ == '__main__':
    show_address_data('GD7YLRC3YWR3SMVGY3TSQ2UL56D7SG3JDIGYPNYZ2G22HBMOX5S7CLYF')
```

Unless you specified a different amount, ```Account B``` should return a balance of 100 KOOL Tokens:

```
Public key: GACNGVOSMX7NUBKUEPU26FQ2ROQZRVZ6IGGPESEHNGEKNRS55OUWU2YG
Last modified ledger: 85623
Lumen Balance: 10199.9999900
KOOL Balance: 100.0000000
```

Congrats, you're 100 KOOL richer! 💰

### Assets Continued

One thing you might notice if you try to read the balance of ```Account A``` is that there is no balance for KOOL Tokens. In most cases, anchors like to create two accounts - where one serves as an issuer account and one as a distributor account. Distributor accounts are often used to seperate the logic between issuance and distribution and to create a limited supply and/or provide transparency about supply to customers and Stellar users.

There are some best practices that we didn't adhere to for simplicity. If you're genuinely interested in issuing a Stellar asset I recommend going through a few pieces of documentation:
- [Custom Assets](https://www.stellar.org/developers/guides/walkthroughs/custom-assets.html)
- [Issuing Assets](https://www.stellar.org/developers/guides/issuing-assets.html)
- [Assets](https://www.stellar.org/developers/guides/concepts/assets.html)

If you're interested in having more control over who can hold your asset read: [Controlling Asset Holders](https://www.stellar.org/developers/guides/concepts/assets.html#controlling-asset-holders)

Let's move on and learn about the Stellar decentralized exchange.

→ [Chapter 4 - Decentralized Exchange](4-decentralized-exchange.md)
