[Home](README.md) - [Chapter 1](1-accounts.md) - [Chapter 2](2-payments.md) - [Chapter 3](3-assets.md) - [Chapter 4](4-decentralized-exchange.md) - [Chapter 5](5-path-payments.md) - [Chapter 6](6-xdr.md) - [Conclusion](7-conclusion.md)

## Chapter 1 - Accounts on Stellar

<div align="center"><img width="25%" src="imgs/accounts.png"></div>
<br>

For the full documentation on Accounts - [go here.](https://www.stellar.org/developers/guides/concepts/accounts.html)

To interact with the Stellar network you'll need an account. Your account allows you to send and receive transactions, make offers on the decentralized exchange, and more. Account access is controlled by a public/private key pair.

Accounts are *identified* by a public key. Your public key is what you'll send your friends when you want them to send you money, almost like a username. 

Accounts are *accessed* by a private key. In order for you to prove ownership of your account and use it to send a transaction (e.g. make a payment), you'll need to sign transactions with your private key. Following the username analogy, a private key is similar to a password except it is uniquely tied to your public key. I'd love to dive deeper and explain this myself, but public / private key cryptography is better explained via [this short video](https://youtu.be/GSIDS_lvRv4).

It's important that you keep your private key(s) secret to avoid getting your account(s) compromised. Setting up [multi-sig](https://www.lumenauts.com/guides/how-to-set-up-a-multi-sig-wallet) will also increase your security but I'll save that for another time.

Extra reading: [Security Guide – How To Protect Yourself From Scammers](https://www.stellar.org/blog/stellar-security-guide-protect-scammers/).

Another important distinction to make, is that accounts can exist on the Stellar mainnet (the one with real money) or they can exist on the Stellar testnet (using fake money). Throughout these chapters the accounts we create will exist on **testnet**, because that's what it's for.. testing! When creating Stellar applications, it is always best to start on testnet and move to mainnet later when you're ready for production. A quick guide on testnet ettiquite can be found [here](https://www.stellar.org/developers/guides/concepts/test-net.html).

### Create an Account

Before we can do anything fun, we'll need a Stellar account. Let's generate our first keypair with the following script (```keypair.py```): 

``` python
from stellar_sdk import Keypair

def generate_keypair():
    keypair = Keypair.random()
    print("Public key:", keypair.public_key)
    print("Private key:", keypair.secret)

if __name__ == '__main__':
    generate_keypair()
```

The function ```generate_keypair()``` creates a public/private key pair using the Keypair object provided by the Python Stellar SDK. The individual components of the Keypair object can be accessed using ```keypair.public_key``` and ```keypair.secret```. 

After running the script you should see the results printed to console like this:

```
Public key: GBG7D5ZZJLAKPDBAGSVS3O3TMIV2O3HOIOXE2OSGGCYNRATOICDRTIAR
Private key: SBK4EAZIWXELREKEXP4WB6DCCMJH7SGTEQE2BJALA32VQQ4ADFAWJGOV
```

Be sure to **save the keys you generate** somewhere in a ```.txt``` file. We **will** be reusing them later.

Awesome! We have an account, but before we can use it we need to 'register' it with the testnet and fund it with lumens to satisfy the minimum account balance. 

What are lumens? The lumen, often abbreviated XLM, is the protocol token of the Stellar network aka the *native* currency of the network. Anyone that wants to hold or move money on Stellar must also hold lumens. They are generally used to pay fees, but are also a functional currency on the network. 

**Note:** In order to prevent spam, Stellar requires accounts to maintain a minimum account balance that is calculated using the **base reserve** fee of 0.5 lumens. Minimum Account Balance = (2 + # of entries) * base reserve fee. Each additional entry reserves an additional 0.5 XLM. Entries include: trustlines, offers, signers, and data entries, but more on those later. 

Luckily we have a friend that lives on testnet called [Friendbot](https://github.com/stellar/go/tree/master/services/friendbot) who has some lumens to spare. So we'll need to ask for some funds by visiting this link: (Replace my public key with the one you generated)

https://friendbot.stellar.org/?addr=GBG7D5ZZJLAKPDBAGSVS3O3TMIV2O3HOIOXE2OSGGCYNRATOICDRTIAR 

You should see a response that looks like this after visiting the link with your address plugged in:

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
    friendbot = 'https://friendbot.stellar.org'

    keypair = Keypair.random()
    requests.get(friendbot, params={'addr': keypair.public_key})

    print("Public key:", keypair.public_key)
    print("Private key:", keypair.secret)

if __name__ == '__main__':
    generate_keypair()
``` 

Simply import the requests library, include the friendbot URL, and send the get request after generating a keypair. 

### A Brief Horizon Overview 

Before going further, I should quickly introduce the Horizon API because we'll be interfacing with it quiet a bit. Horizon is an API server that allows you to submit transactions to the Stellar network, get account details, stream data from the network, and more. The most commonly used Horizon instances are https://horizon.stellar.org/ and https://horizon-testnet.stellar.org/ (the one we'll be using). These two are run and maintained by the Stellar Development Foundation, but other public facing Horizon APIs exist such as https://stellar-horizon.satoshipay.io/. 

Horizon has multiple endpoints that we can get information from such as: 
- https://horizon-testnet.stellar.org/accounts/{account}
- https://horizon-testnet.stellar.org/assets{?asset_code,asset_issuer,cursor,limit,order}
- https://horizon-testnet.stellar.org/ledgers/{sequence}
- Find the complete overview [here](https://www.stellar.org/developers/horizon/reference/index.html). 

We'll be using a few of these throughout the following chapters. 

### Getting Account Information

I know I told you that your testnet account has 10,000 lumens, but you don't have to take my word for it. Let's see how we can interact with the [Horizon testnet API](https://horizon-testnet.stellar.org/) to get more details about accounts and check out our account balance.

Using the SDK we can send a get request to the Horizon account endpoint (https://horizon-testnet.stellar.org/accounts/{account}) and grab the balance information that we are looking for like so (```balance.py```:

``` python
from stellar_sdk import Server

def show_balance(public_key):
    server = Server(horizon_url='https://horizon-testnet.stellar.org')
    address = server.accounts().account_id(public_key).call()
    print('Lumen Balance:', address['balances'][-1]['balance'], 'XLM')

if __name__ == '__main__':
    show_balance('GBG7D5ZZJLAKPDBAGSVS3O3TMIV2O3HOIOXE2OSGGCYNRATOICDRTIAR')
```

This script creates a server object that connects us to the public facing Horizon testnet API and allows us to interface with it. Using the server object we can create a new ```AccountsCallBuilder``` object via ```server.accounts()``` and use that object to get data about a specific account via ```server.accounts().account_id(public_key)```. From there, using ```.call()``` sends a get request to Horizon and returns a JSON response that you can see by visiting https://horizon-testnet.stellar.org/accounts/GBG7D5ZZJLAKPDBAGSVS3O3TMIV2O3HOIOXE2OSGGCYNRATOICDRTIAR - feel free to replace my public key with your own. The script then stores that response in a variable ```address```. 

To print the results in a nice format we can grab balance data directly from the JSON using ```address['balances'][-1]['balance']```. The way this works is that it isolates the list of balances attached to the account, grabs the last balance on the list (which is always lumens), and then grabs the number balance associated with that item in the list. 

Running this script should print the following: 

``` json 
Lumen Balance: 10000.0000000 XLM
``` 

**Note:** It is important to remember that Stellar accounts can have many balances associated with them. When presenting balances to a user or checking your own balances, you should generally go through this entire list presenting the name of the asset and associated balance. If you are only interested in lumens, these are always placed at the end of the list of balances making for easy access. 

To learn more about accounts, read the following documentation for a detailed description of each account field: https://www.stellar.org/developers/guides/concepts/accounts.html#account-fields

Now that we confirmed that we have 10,000 lumens in our testnet account, we should learn how to send our first payment.

→ [Chapter 2 - Payments](2-payments.md)
