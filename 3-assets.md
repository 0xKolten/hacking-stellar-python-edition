[Home](README.md) - [Chapter 1](1-accounts.md) - [Chapter 2](2-payments.md) - [Chapter 3](3-assets.md) - [Chapter 4](4-decentralized-exchange.md) - [Chapter 5](5-path-payments.md) - [Conclusion](6-conclusion.md) - [Bonus Chapter 1](bonus-xdr.md) - [Bonus Chapter 2](bonus-streaming.md)

## Chapter 3 - Assets on Stellar

<div align="center"><img width="25%" src="imgs/assets.png"></div>
<br>

For the official documentation on assets - [go here](https://www.stellar.org/developers/guides/concepts/assets.html).

One unique feature of the Stellar network is that it can be used to trade, hold, and transfer any type of asset. If you can tokenize it, it can be issued as credit and used on Stellar (e.g. dollars, cryptocurrencies, security tokens, etc.).

All assets on Stellar have an **asset code** (e.g. USD) and an **issuer account** (who created the token). Lumens are the only asset that *do not* have an issuer account. Assets can also have flags associated with them such as ```AUTHORIZATION REQUIRED``` and ```AUTHORIZATION REVOCABLE```. These flags allow asset issuers to have more control over their issued assets to adhere with compliance, improve customer security, etc.

Before we create an asset there's two concepts we should understand first: **anchors** and **trustlines**.

### Anchors

Anchors in simplest terms are entities that issue assets on Stellar. [Anchors](https://github.com/koltenb/awesome-stellar/blob/master/README.md#stellar-asset-issuers-anchors) can range from individuals to commercial banks. Generally, anchors take deposits from users and issue those deposits on the Stellar network as credit. For example, I can deposit $100 in to [AnchorUSD](https://www.anchorusd.com/) and receive the equivalent amount of [USD tokens/credit](https://stellar.expert/explorer/public/asset/USD-GDUKMGUGDZQK6YHYA5Z6AY2G4XDSZPSZ3SW5UN3ARVMO6QSRDWP5YLEX) on Stellar. When I'm ready to redeem those USD tokens I can withdraw them back in to my bank account through AnchorUSD.

### Trustlines

Continuing the AnchorUSD example: When I deposit $100 to AnchorUSD in exchange for tokens/credit on Stellar, I am ***trusting*** that I will be able to redeem them later for $100. This idea of trust is conceptualized on Stellar through **trustlines**.

In order to hold an asset on Stellar, you must explicitly trust the asset issuer through a trustline. The ```Change Trust``` operation is used to create, update, or delete trustlines. After a trustline is set, it will track the limit for which your account trusts the issuing account and the amount of credit from the issuing account that your account currently holds. Trustlines ensure that you only interact with assets that you have faith in. You don't want bad actors sending counterfit USD to your account that you will be unable to redeem later.

**Note**: Trustlines have a **base reserve fee** of 0.5 lumens each. You will get the fee back after closing the trustline.

### Issuing an Asset

Now that you understand the concepts around Stellar assets, let's create one of our own.

We already have 2 accounts created so we'll use them again:

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

There is a couple of different ways to structure your accounts as an anchor (discussed after the example below), but we'll only be using one account as an **issuer account**.

```Account A``` will be the issuing account in this case.

Creating an asset is actually pretty simple, we can do so with the following script:

``` python
from stellar_base.asset import Asset

def create_asset(asset_code, asset_issuer):
    asset = Asset(asset_code, asset_issuer)
    return asset

if __name__ == '__main__':
    asset = create_asset('KOOL', 'GD7YLRC3YWR3SMVGY3TSQ2UL56D7SG3JDIGYPNYZ2G22HBMOX5S7CLYF')
    print(asset)
```

Using the Asset object provided by the Python Stellar SDK, we provide an asset code and the public key for ```Account A``` to issue "KOOL Tokens." After running the script you should get a short response printed to console:

``` json
<stellar_base.asset.Asset object at 0x02C4A970>
```

Now we need to create a trustline between ```Account B``` and ```Account A``` so that ```Account B``` can receive some KOOL Tokens. We can do this using a script that is very similar to the one we used to send payments:

``` python
from stellar_base.builder import Builder
import json

def create_trust(signing_key, asset_code, asset_issuer):
    builder = Builder(secret=signing_key, horizon_uri='https://horizon-testnet.stellar.org' , network='testnet') \
        .append_change_trust_op(asset_code, asset_issuer)
    # Sign and submit transaction -> print response
    builder.sign()
    response = builder.submit()
    print(json.dumps(response, indent = 2))

if __name__ == '__main__':
    create_trust('SBKNPGV4QOCU6CB4GEUF76LVFJFMBWHY4JVIVYEUB76HYQAG7HXLLD77', 'KOOL', 'GD7YLRC3YWR3SMVGY3TSQ2UL56D7SG3JDIGYPNYZ2G22HBMOX5S7CLYF')
```

Using the Builder object, we can attach a ```Change Trust``` operation via ```.append_change_trust_op(asset_code, asset_issuer)``` and pass in the desired asset code (KOOL) and asset issuer (Account A's public key).

You should get a response like this if all went according to plan:

(Remember to check the transaction link to make sure it was successful)

``` json
{
  "_links": {
    "transaction": {
      "href": "https://horizon-testnet.stellar.org/transactions/14f4d60c8dbee2fb9df234b5385ec94c47e9793cd7ba7c384ca495cbe9c6700d"
    }
  },
  "hash": "14f4d60c8dbee2fb9df234b5385ec94c47e9793cd7ba7c384ca495cbe9c6700d",
  "ledger": 85623,
  "envelope_xdr": "AAAAAATTVdJl/toFVCPprxYai6GY1z5BjPJIh2mIpsZd66lqAAAAZAAAEigAAAABAAAAAAAAAAAAAAABAAAAAAAAAAYAAAABS09PTAAAAAD/hcRbxaO5MqbG5yhqi++H+RtpGg2HtxnRtaOFjr9l8X//////////AAAAAAAAAAFd66lqAAAAQAbYNwk3nKskZokrK5J8VOjPqITP7MSFIinF8O89xjJOS38Ixaaox/rJ0j9eEilSn1s2o3rrGSkMs71F+NFSZAo=",
  "result_xdr": "AAAAAAAAAGQAAAAAAAAAAQAAAAAAAAAGAAAAAAAAAAA=",
  "result_meta_xdr": "AAAAAQAAAAIAAAADAAFOdwAAAAAAAAAABNNV0mX+2gVUI+mvFhqLoZjXPkGM8kiHaYimxl3rqWoAAAAXv6x7nAAAEigAAAAAAAAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAAAAAAAAAAAAAAAABAAFOdwAAAAAAAAAABNNV0mX+2gVUI+mvFhqLoZjXPkGM8kiHaYimxl3rqWoAAAAXv6x7nAAAEigAAAABAAAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAAAAAAAAAAAAAAAABAAAAAwAAAAMAAU53AAAAAAAAAAAE01XSZf7aBVQj6a8WGouhmNc+QYzySIdpiKbGXeupagAAABe/rHucAAASKAAAAAEAAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAEAAU53AAAAAAAAAAAE01XSZf7aBVQj6a8WGouhmNc+QYzySIdpiKbGXeupagAAABe/rHucAAASKAAAAAEAAAABAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAAAAU53AAAAAQAAAAAE01XSZf7aBVQj6a8WGouhmNc+QYzySIdpiKbGXeupagAAAAFLT09MAAAAAP+FxFvFo7kypsbnKGqL74f5G2kaDYe3GdG1o4WOv2XxAAAAAAAAAAB//////////wAAAAEAAAAAAAAAAA=="
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
