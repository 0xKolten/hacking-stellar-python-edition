[Home](README.md) - [Chapter 1](1-accounts.md) - [Chapter 2](2-payments.md) - [Chapter 3](3-assets.md) - [Chapter 4](4-decentralized-exchange.md) - [Chapter 5](5-path-payments.md) - [Conclusion](6-conclusion.md) - [Bonus Chapter 1](bonus-xdr.md) - [Bonus Chapter 2](bonus-streaming.md)

## Chapter 1 - Accounts on Stellar

<div align="center"><img width="25%" src="imgs/accounts.png"></div>
<br>

For the full documentation on Accounts - [go here.](https://www.stellar.org/developers/guides/concepts/accounts.html)

To interact with the Stellar network you'll need an account. Your account allows you to send and receive transactions, make offers on the decentralized exchange, and more. Account access is controlled by a public/private key pair.

Accounts are *identified* by a public key. Your public key is what you'll send your friends when you want them to send you money, almost like a username.

Accounts are *accessed* by a private key. In order for you to use your account to send a transaction (e.g. make a payment), you'll need to sign transactions with your private key. Following the username analogy, a private key is similar to a password except it is uniquely tied to your public key. I'd love to dive deeper and explain this myself, but public / private key cryptography is better explained via [a short video](https://youtu.be/GSIDS_lvRv4).

It's important that you keep your private key(s) secret to avoid getting your account(s) compromised. Setting up [multi-sig](https://www.lumenauts.com/guides/how-to-set-up-a-multi-sig-wallet) will also increase your security.

Extra reading: [Security Guide – How To Protect Yourself From Scammers](https://www.stellar.org/blog/stellar-security-guide-protect-scammers/).

Another important distinction to make, is that accounts can exist on the Stellar main net (the one with real money) or they can exist on the Stellar testnet (using fake money). Throughout these chapters the accounts we create will exist on testnet, because that's what it's for.. testing! When creating Stellar applications, it is always best to start on testnet and move to main net later when you're ready for production. A quick guide on testnet ettiquite can be found [here](https://www.stellar.org/developers/guides/concepts/test-net.html).

### Create an Account

Before we can do anything fun, we'll need a Stellar testnet account. Let's generate our first keypair with the following script:

``` python
from stellar_sdk import Keypair

def generate_keypair():
    keypair = Keypair.random()
    print("Public key:", keypair.public_key)
    print("Private key:", keypair.secret)

if __name__ == '__main__':
    generate_keypair()
```

The function ```generate_keypair()``` creates a public/private key pair using the Keypair object provided by the Python Stellar SDK. The individual components of the Keypair object can be accessued using ```keypair.public_key``` and ```keypair.secret```. 

We then print the results to console and you should see something like this:

```
Public key: GD7YLRC3YWR3SMVGY3TSQ2UL56D7SG3JDIGYPNYZ2G22HBMOX5S7CLYF
Private key: SDRRZ2JQBI3RQSHV4Q63YWWJOVFNSXE2R6YSQKGUIPY65UATGUZUX4BB
```

Awesome! We have an account, but before we can use it we need to register it with the testnet and fund it with lumens to satisfy the minimum account balance. (Be sure to **save the keys you generate** as we will be reusing them later.)

**Note:** In order to prevent spam, Stellar requires accounts to maintain a minimum account balance that is calculated using the **base reserve** fee of 0.5 lumens. Minimum Account Balance = (2 + # of entries) * base reserve fee. Each additional entry reserves an additional 0.5 XLM. Entries include: trustlines, offers, signers, and data entries, but more on those later. 

Luckily [Friendbot](https://github.com/stellar/go/tree/master/services/friendbot) has some lumens to spare. So we'll need to ask for some funds by visiting this link: (Replace my public key with the one you generated)

https://friendbot.stellar.org/?addr=GD7YLRC3YWR3SMVGY3TSQ2UL56D7SG3JDIGYPNYZ2G22HBMOX5S7CLYF 

You should get a response that looks like this after visiting the link:

``` json
{
  "_links": {
    "transaction": {
      "href": "https://horizon-testnet.stellar.org/transactions/d73d729aca83fc7205256137d46bf61693dd7d0d75cf29ab2350f3e0c6ce35d7"
    }
  },
  "hash": "d73d729aca83fc7205256137d46bf61693dd7d0d75cf29ab2350f3e0c6ce35d7",
  "ledger": 4586,
  "envelope_xdr": "AAAAAJfwvT2ZL2JpGQ3sbKLRqRt8+4bs0iFEjVDuWyE5i60bAAAAZAAAAMoAAAAKAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAQAAAAEAAAAAEH3Rayw4M0iCLoEe96rPFNGYim8AVHJU0z4ebYZW4JwAAAAAAAAAAP+FxFvFo7kypsbnKGqL74f5G2kaDYe3GdG1o4WOv2XxAAAAF0h26AAAAAAAAAAAAjmLrRsAAABAfsPMIoEzviwifAXzNO2gANpUKjKD8hl9tnhCho+DDqSe3Y37awEfOw4/7Gh7eFzN9RG6K4ucC9LvBp8TXvnZC4ZW4JwAAABAH4YJK8pZNN6w6aTDBSWwcLiHHz1TEBxD/RDxJTDfME80hXnkJnneSZmP2d06X20Tj472p95cOEk18JIvdovaDQ==",
  "result_xdr": "AAAAAAAAAGQAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAA=",
  "result_meta_xdr": "AAAAAQAAAAIAAAADAAAR6gAAAAAAAAAAl/C9PZkvYmkZDexsotGpG3z7huzSIUSNUO5bITmLrRsAAAAAPDNcmAAAAMoAAAAJAAAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAAAAAAAAAAAAAAAABAAAR6gAAAAAAAAAAl/C9PZkvYmkZDexsotGpG3z7huzSIUSNUO5bITmLrRsAAAAAPDNcmAAAAMoAAAAKAAAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAAAAAAAAAAAAAAAABAAAAAwAAAAMAABHoAAAAAAAAAAAQfdFrLDgzSIIugR73qs8U0ZiKbwBUclTTPh5thlbgnAFe4GqH5kSYAAAAwQAAAAwAAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAEAABHqAAAAAAAAAAAQfdFrLDgzSIIugR73qs8U0ZiKbwBUclTTPh5thlbgnAFe4FM/b1yYAAAAwQAAAAwAAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAAAABHqAAAAAAAAAAD/hcRbxaO5MqbG5yhqi++H+RtpGg2HtxnRtaOFjr9l8QAAABdIdugAAAAR6gAAAAAAAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAA=="
}
```

Now we have a testnet account that's funded with 10,000 lumens from Friendbot.

**Note**: You could also add a get request to your ```generate_keypair()``` function to do this programmatically and save time in the future. A quick and dirty version of the script would look like: 

``` python
from stellar_sdk import Keypair
import requests

def generate_keypair():
    friendbot = 'https://friendbot.stellar.org/?addr='
    keypair = Keypair.random()
    requests.get(friendbot + keypair.public_key)
    print("Public key:", keypair.public_key)
    print("Private key:", keypair.secret)

if __name__ == '__main__':
    generate_keypair()
``` 

Simply import the requests library, include the friendbot URL, and send the get request after generating a keypair. 

### Getting Account Information

I know I told you that your testnet account has 10,000 lumens, but you don't have to take my word for it. Let's see how we can interact with the Horizon testnet API to get more details about accounts.

One way is to send a get request to the Horizon account endpoint (https://horizon-testnet.stellar.org/accounts/{account}):

``` python
import requests
import json

def show_address_data(public_key):
    r = requests.get('https://horizon-testnet.stellar.org/accounts/' + public_key)
    account_data = r.json()
    print(json.dumps(account_data, indent = 2))

if __name__ == '__main__':
    show_address_data('GD7YLRC3YWR3SMVGY3TSQ2UL56D7SG3JDIGYPNYZ2G22HBMOX5S7CLYF')
```

Running your script should return a response showing all account fields that looks like:
``` json
{
  "_links": {
    "self": {
      "href": "https://horizon-testnet.stellar.org/accounts/GD7YLRC3YWR3SMVGY3TSQ2UL56D7SG3JDIGYPNYZ2G22HBMOX5S7CLYF"
    },
  -- SNIP --
  "id": "GD7YLRC3YWR3SMVGY3TSQ2UL56D7SG3JDIGYPNYZ2G22HBMOX5S7CLYF",
  "account_id": "GD7YLRC3YWR3SMVGY3TSQ2UL56D7SG3JDIGYPNYZ2G22HBMOX5S7CLYF",
  "sequence": "19696720019456",
  "subentry_count": 0,
  "last_modified_ledger": 4586,
  "thresholds": {
    "low_threshold": 0,
    "med_threshold": 0,
    "high_threshold": 0
  },
  "flags": {
    "auth_required": false,
    "auth_revocable": false,
    "auth_immutable": false
  },
  "balances": [
    {
      "balance": "10000.0000000",
      "buying_liabilities": "0.0000000",
      "selling_liabilities": "0.0000000",
      "asset_type": "native"
    }
  ],
  "signers": [
    {
      "weight": 1,
      "key": "GD7YLRC3YWR3SMVGY3TSQ2UL56D7SG3JDIGYPNYZ2G22HBMOX5S7CLYF",
      "type": "ed25519_public_key"
    }
  ],
  "data": {}
}
```

Read the following documentation for a detailed description of each account field: https://www.stellar.org/developers/guides/concepts/accounts.html#account-fields

Another way to get information about our account is to use the Address object provided by the Python Stellar SDK.

Adjust your script to look like this:

``` python
from stellar_base.address import Address

def show_address_data(public_key):
    address = Address(address = public_key, secret = None, network='testnet')
    address.get()

    print("Public key:", address.id)
    print("Last modified ledger:", address.last_modified_ledger)
    print("Lumen Balance:", address.balances[-1]['balance'])

if __name__ == '__main__':
    show_address_data('GD7YLRC3YWR3SMVGY3TSQ2UL56D7SG3JDIGYPNYZ2G22HBMOX5S7CLYF')
```

With the above code, we use the Address object provided by the Python Stellar SDK to get our public key ```address.id```, the last modified ledger modified by our account ```address.last_modified_ledger```, and our lumen balance ```address.balances[-1]['balance']```.

**Note:** Stellar addresses can contain many different balances for many different assets. The balances attribute will return an array of all assets the account holds and their respective balances. Lumens are always located last in the array and can be accessed consistently using ```address.balances[-1]```.

Running the script should print the following information to console:

```
Public key: GD7YLRC3YWR3SMVGY3TSQ2UL56D7SG3JDIGYPNYZ2G22HBMOX5S7CLYF
Last modified ledger: 4586
Lumen Balance: 10000.0000000
```

The Address object exists as a helper class for Horizon operations on an account. It can be valuable when working with Stellar addresses in your Python scripts or programs. The full documentation can be found [here](https://stellar-base.readthedocs.io/en/latest/api.html#address).

Now that we have 10,000 lumens in our Testnet account, we should learn how to send our first payment.

→ [Chapter 2 - Payments](2-payments.md)
