[Home](README.md) - [Chapter 0](0-setup.md) - [Chapter 1](1-accounts.md) - [Chapter 2](2-payments.md) - [Chapter 3](3-assets.md) - [Chapter 4](4-decentralized-exchange.md) - [Chapter 5](5-path-payments.md) - [Conclusion](6-conclusion.md)

## Chapter 3 - Assets on Stellar

<div align="center"><img width="25%" src="imgs/assets.png"></div>
<br>

For the official documentation on assets - [go here](https://www.stellar.org/developers/guides/concepts/assets.html).

One unique feature of the Stellar network is that it can be used to trade, hold, and transfer any type of asset. If you can represent it as a number in a database, it can be issued as a token and used on Stellar (e.g. dollars, cryptocurrencies, security tokens, etc.).

All assets on Stellar have an **asset code** (e.g. USD) and an **issuer account** (who created the token). Lumens are the only asset that *do not* have an issuer account. Assets can also have flags associated with them such as ```AUTHORIZATION REQUIRED``` and ```AUTHORIZATION REVOCABLE```. These flags allow asset issuers to have more control over their issued assets to adhere with compliance, improve customer security, etc.

Before creating an asset there's two concepts that need explaining: **anchors** and **trustlines**.

### Anchors (Asset Issuers)

In simplest terms, anchors are entities that issue assets on Stellar. [Anchors](https://github.com/koltenb/awesome-stellar/blob/master/README.md#stellar-asset-issuers-anchors) can range from individuals to commercial banks. Generally, anchors take deposits from users and issue those deposits on the Stellar network as a token (or a credit). For example, I can deposit $100 in to [AnchorUSD](https://www.anchorusd.com/) and receive the equivalent amount of [USD tokens/credit](https://stellar.expert/explorer/public/asset/USD-GDUKMGUGDZQK6YHYA5Z6AY2G4XDSZPSZ3SW5UN3ARVMO6QSRDWP5YLEX) on Stellar. When I'm ready to redeem those USD tokens I can withdraw them back in to my bank account through AnchorUSD.

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

Next, we should repurpose the payment script created earlier so that it can send more assets besides lumens. This will allow us to send ```HACK``` tokens from the issuing account to the distributing account. 

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

def send_payment(signing_key, receiving_key, amount, asset='XLM', asset_issuer=None):
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
        .append_payment_op(receiving_key, amount, asset, asset_issuer)
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
    Account A (Issuer)
    Public key: GBG7D5ZZJLAKPDBAGSVS3O3TMIV2O3HOIOXE2OSGGCYNRATOICDRTIAR
    Private key: SBK4EAZIWXELREKEXP4WB6DCCMJH7SGTEQE2BJALA32VQQ4ADFAWJGOV

    Account B (Distributor)
    Public key: GCF4PQGCOFR245LDPRMBDGMD7VQMOGF2KQZJAWICB6JJ337NDPUQR66E
    Private key: SCO4JEHDN2ZLIDZMZC62UR6Y5NICMTMRKBPG3JMBKI5AHVXJA46MY2VG
    """

    send_payment('SBK4EAZIWXELREKEXP4WB6DCCMJH7SGTEQE2BJALA32VQQ4ADFAWJGOV', 'GCF4PQGCOFR245LDPRMBDGMD7VQMOGF2KQZJAWICB6JJ337NDPUQR66E', '10000', 'HACK', 'GBG7D5ZZJLAKPDBAGSVS3O3TMIV2O3HOIOXE2OSGGCYNRATOICDRTIAR')
```

In the new version of the send payment script only two things need changing: 

```python
def send_payment(signing_key, receiving_key, amount, asset='XLM', asset_issuer=None)
```

This change adds an asset issuer to the ```send_payment()``` function. This is necessary because all assets on the network, besides lumens, have a unique issuing address. To ensure that you are sending the correct asset, the asset code *and* its associated asset issuer must be provided. Otherwise various things can cause the transaction to fail. I set the ```assets``` default to ```XLM``` and the ```asset_issuer``` default to ```None``` so this script can still be used to send a basic lumen payment. 

The next change is: 

```python
.append_payment_op(receiving_key, amount, asset, asset_issuer)
```

Adding the ```asset_issuer``` to the payment operation is the last big step and is necessary for the reasons listed above. 

Finally, don't forget to change your function call at the bottom of the script: 

```python 
send_payment('SBK4EAZIWXELREKEXP4WB6DCCMJH7SGTEQE2BJALA32VQQ4ADFAWJGOV', 'GCF4PQGCOFR245LDPRMBDGMD7VQMOGF2KQZJAWICB6JJ337NDPUQR66E', '10000', 'HACK', 'GBG7D5ZZJLAKPDBAGSVS3O3TMIV2O3HOIOXE2OSGGCYNRATOICDRTIAR')
```

This change specifies that we are sending 10,000 HACK tokens from the issuing account to the distributing account. 

After running the script you should get a similar response to what we've seen before and should verify that the transacation was a success (```Transaction result: txSUCCESS```). 

Once the transaction is successful, you can repurpose the script we made earlier to check *all* of an account's balances: 

``` python
from stellar_sdk import Server

def show_balance(public_key):
    server = Server(horizon_url='https://horizon-testnet.stellar.org')
    address = server.accounts().account_id(public_key).call()
    for balance in address['balances']:
        if 'asset_code' in balance:
            print(balance['asset_code'], "balance:", balance['balance'])
        else:
            print('XLM balance:', address['balances'][-1]['balance'])

if __name__ == '__main__':
    show_balance('GCF4PQGCOFR245LDPRMBDGMD7VQMOGF2KQZJAWICB6JJ337NDPUQR66E')
```

The big change in this script compared to the previous version is that it loops through all of the available balances that an account has. In the loop, we check to see if any balances include an ```asset_code```, if so, it prints the associated balance. After going through the custom assets, it then prints the lumen balance because lumen balances do not have an associated asset code. Easy stuff! 

### Assets Continued

One thing you might notice if you try to read the balance of ```Account A``` is that there is no balance for ```HACK``` tokens. This is because issuing accounts don't maintain a balance on Stellar for the assets they are issuing. Since anyone can extend a trustline to anyone else for some arbitrary token, balances and outstanding supply aren't created until the trustline is *acted upon*. In order for the token to actually exist, the issuer has to send that token to someone. 

If you're interested in issuing a Stellar asset I recommend going through a few pieces of documentation:
- [Custom Assets](https://www.stellar.org/developers/guides/walkthroughs/custom-assets.html)
- [Issuing Assets](https://www.stellar.org/developers/guides/issuing-assets.html)
- [Assets](https://www.stellar.org/developers/guides/concepts/assets.html)

If you're interested in having more control over who can hold your asset read: [Controlling Asset Holders](https://www.stellar.org/developers/guides/concepts/assets.html#controlling-asset-holders)

Let's move on and learn about the Stellar decentralized exchange.

→ [Chapter 4 - Decentralized Exchange](4-decentralized-exchange.md)
