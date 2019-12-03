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
Public key: GBG7D5ZZJLAKPDBAGSVS3O3TMIV2O3HOIOXE2OSGGCYNRATOICDRTIAR
Private key: SBK4EAZIWXELREKEXP4WB6DCCMJH7SGTEQE2BJALA32VQQ4ADFAWJGOV
```

Awesome! We have an account, but before we can use it we need to register it with the testnet and fund it with lumens to satisfy the minimum account balance. (Be sure to **save the keys you generate** as we will be reusing them later.)

**Note:** In order to prevent spam, Stellar requires accounts to maintain a minimum account balance that is calculated using the **base reserve** fee of 0.5 lumens. Minimum Account Balance = (2 + # of entries) * base reserve fee. Each additional entry reserves an additional 0.5 XLM. Entries include: trustlines, offers, signers, and data entries, but more on those later. 

Luckily [Friendbot](https://github.com/stellar/go/tree/master/services/friendbot) has some lumens to spare. So we'll need to ask for some funds by visiting this link: (Replace my public key with the one you generated)

https://friendbot.stellar.org/?addr=GBG7D5ZZJLAKPDBAGSVS3O3TMIV2O3HOIOXE2OSGGCYNRATOICDRTIAR 

You should see a response that looks like this after visiting the link:

``` json
{
  "_links": {
    "transaction": {
      "href": "https://horizon-testnet.stellar.org/transactions/8ca89139404fdfa8abd6da7fc53b1f3e37dd4835b846972d915bf0d7bda1d5e4"
    }
  },
  "hash": "8ca89139404fdfa8abd6da7fc53b1f3e37dd4835b846972d915bf0d7bda1d5e4",
  "ledger": 543172,
  "envelope_xdr": "AAAAAMBOk1wMhFkn4483FUoFZikZhhP8pl4kr40Xr6U25RcpAAGGoAAEaUYAAABmAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAQAAAAEAAAAAEH3Rayw4M0iCLoEe96rPFNGYim8AVHJU0z4ebYZW4JwAAAAAAAAAAE3x9zlKwKeMIDSrLbtzYiunbO5Drk06RjCw2IJuQIcZAAAAF0h26AAAAAAAAAAAAjblFykAAABALoS0nD5vchJx5LDD5ROigAS0384ahwlBGuCBmnJx9lLuK4BT998prRx4xD/ZvEr/Qx+Oc35p1PH8Lm0uLaLBC4ZW4JwAAABAxlj2tG4AkrEUm4OWa4vz1obJE0s0jYYa2COtRix1m6IInFX1PWfjBHL1zs4qUX3+8BFhf2nVvqzY0I6IsBKaCA==",
  "result_xdr": "AAAAAAAAAGQAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAA=",
  "result_meta_xdr": "AAAAAQAAAAIAAAADAAhJxAAAAAAAAAAAwE6TXAyEWSfjjzcVSgVmKRmGE/ymXiSvjRevpTblFykAAAAAPCJuOAAEaUYAAABlAAAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAAAAAAAAAAAAAAAABAAhJxAAAAAAAAAAAwE6TXAyEWSfjjzcVSgVmKRmGE/ymXiSvjRevpTblFykAAAAAPCJuOAAEaUYAAABmAAAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAAAAAAAAAAAAAAAABAAAAAwAAAAMACEm4AAAAAAAAAAAQfdFrLDgzSIIugR73qs8U0ZiKbwBUclTTPh5thlbgnAExXY9m6u8hAAAAZgAAAGcAAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAEACEnEAAAAAAAAAAAQfdFrLDgzSIIugR73qs8U0ZiKbwBUclTTPh5thlbgnAExXXgedAchAAAAZgAAAGcAAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAAACEnEAAAAAAAAAABN8fc5SsCnjCA0qy27c2Irp2zuQ65NOkYwsNiCbkCHGQAAABdIdugAAAhJxAAAAAAAAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAA=="
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
