[Home](README.md) - [Chapter 1](1-accounts.md) - [Chapter 2](2-payments.md) - [Chapter 3](3-assets.md) - [Chapter 4](4-decentralized-exchange.md) - [Chapter 5](5-path-payments.md) - [Chapter 6](6-xdr.md) - [Conclusion](7-conclusion.md)

## Chapter 5 - Path Payments on Stellar

<div align="center"><img width="25%" src="imgs/path-payment.png"></div>
<br>

For the official documentation on path payments - [go here](https://www.stellar.org/developers/guides/concepts/list-of-operations.html#path-payment).

Stellar's most unique feature is an operation called a ```Path Payment```. It allows users to send an asset through a path of offers, making it possible for the asset being sent (e.g. EUR) to be different from the asset received (e.g. USD).

To show off how path payments work, I'll make use of the STR token from last chapter. We will send lumens from ```Account B``` and have them converted to STR tokens by the time they arrive at ```Account A```'s address. 

### Finding a Payment Path

As I mentioned before, a path payment traverses a path of offers (up to 6) until it is converted to the desired asset. In order to make path payments easier to construct, we can execute a path search using the ```/paths``` Horizon endpoint and use the response to prefill the ```Path Payment``` operation. 

Let's see how we can find a path between two assets. In this example I'll be using what is called a [Strict Receive Payment Path](https://www.stellar.org/developers/horizon/reference/endpoints/path-finding-strict-receive.html). A strict receive payment path allows a user to specify the amount of the asset being receieved. The amount sent will vary based on offers in the order books. I don't do it in this example, but you can also specify a maximum amount that you are willing to send in order to protect your payment from bad orders. The alternative is a [Strict Send Payment Path](https://www.stellar.org/developers/horizon/reference/endpoints/path-finding-strict-send.html) and is simply the opposite of what we're using. 

You can start by writing a script as so: 

``` python 
from stellar_sdk import Server, Keypair, TransactionBuilder, Network
from stellar_sdk.xdr import Xdr
from stellar_sdk.xdr.StellarXDR_const import TransactionResultCode
import requests
import json
import base64

def path_payment(signing_key, asset_code, asset_issuer, amount, receiving_key):

    # Talk to testnet horizon instance
    server = Server(horizon_url="https://horizon-testnet.stellar.org")

    # Derive Keypair object and public key from the signing key (source account)
    source_keypair = Keypair.from_secret(signing_key)
    source_public_key = source_keypair.public_key

    # PATH SEARCH
    # Create Horizon URL
    url = 'https://horizon-testnet.stellar.org/paths/strict-receive?source_account={}&destination_asset_type=credit_alphanum4&destination_asset_code={}&destination_asset_issuer={}&destination_amount={}&destination_account={}'

    # Get JSON response from Horizon
    r = requests.get(url.format(source_public_key, asset_code, asset_issuer, amount, receiving_key))
    path_search = r.json()
    print(json.dumps(path_search, indent=2))

if __name__ == '__main__':
    # STR Issuing Address
    str_issuer = 'GBEYFNS6KJRFEI22X5OBUFKQ5LK7Z2FZVFMAXBINC2SOCKA25AS62PUN'

    # Account A Public Key
    receiving_key = 'GBG7D5ZZJLAKPDBAGSVS3O3TMIV2O3HOIOXE2OSGGCYNRATOICDRTIAR'

    # Account B Private Key
    signing_key = 'SCO4JEHDN2ZLIDZMZC62UR6Y5NICMTMRKBPG3JMBKI5AHVXJA46MY2VG'

    # Run path payment :)
    path_payment(signing_key, 'STR', str_issuer, '5', receiving_key)
```

The information needed to execute a path search for a Strict Receive Path Payment includes: 
- The destination asset issuer (STR issuer) 
- The receiving address (```Account A```'s public key) 
- The sending address (```Account B```'s public key)
- The receiving amount (5 STR) 

**Note:** When the script is finished we will be submitting a transaction from ```Account B```. So instead of just inputting ```Account B```'s public key I input it's secret key then derive the public key for the path search. Got it? Sweet. 

After collecting the relevant info, we use it to fill in the Horizon URL we are getting a response from: 

``` python
url = 'https://horizon-testnet.stellar.org/paths/strict-receive?source_account={}&destination_asset_type=credit_alphanum4&destination_asset_code={}&destination_asset_issuer={}&destination_amount={}&destination_account={}'
```

The empty brackets ```{}``` in the link is where the arguments will get included in the link via ```url.format(sending_key, asset_code, asset_issuer, amount, receiving_key)``` later in the script. For simplicity I chose to fill in the ```destination_asset_type=credit_alphanum4``` argument manually. ```credit_alphanum4``` is used to specify assets that are *not* luments. If lumens are the destination asset this would be changed to ```native```. 

From there, the script sends a ```GET``` request to the provided URL and should print a response like this when you run it: 

``` json
{
  "_embedded": {
    "records": [
      {
        "source_asset_type": "native",
        "source_amount": "9.9600000",
        "destination_asset_type": "credit_alphanum4",
        "destination_asset_code": "STR",
        "destination_asset_issuer": "GBEYFNS6KJRFEI22X5OBUFKQ5LK7Z2FZVFMAXBINC2SOCKA25AS62PUN",
        "destination_amount": "5.0000000",
        "path": []
      }
    ]
  }
}
```

When running path searches, you'll get a response of all of the available paths. In this example, there are orders available on the Stellar DEX that will allow ```Account B```'s lumens to be converted directly to the STR token on their journey to ```Account A```. So the ```path``` list will be empty. 

### Constructing a Path Payment

In order to actually build and submit a path payment our script will have to be a bit more robust. Instead of just printing the path search response to console, I'll instead use it to grab some relevant information. For my version of the script, I'm particularly interested in these two arguments from the response we got earlier: 

```json
"source_amount": "9.9600000",
```
``` json
"path": []
```

The ```source_amount``` argument is how much XLM we'll have to send in order for ```Account A``` to receive 5 STR tokens. We need this to fill out the path payment operation later. The ```path``` argument is a list of assets in between the source asset and the receiving asset. This list is empty so that means its a direct conversion. Since we need this info, remove the print statement from earlier and instead grab that info and store it in two variables: 

``` python
send_max = path_search['_embedded']['records'][0]['source_amount']
path = path_search['_embedded']['records'][0]['path']
```

From there I will build out the script similarly to the other scripts through the book, including the logic for ```tx_success``` and building out our transaction. 

With all of the new code added, it will look something like this: 
```python
from stellar_sdk import Server, Keypair, TransactionBuilder, Network
from stellar_sdk.xdr import Xdr
from stellar_sdk.xdr.StellarXDR_const import TransactionResultCode
import requests
import json
import base64

def tx_success(result_xdr):
    xdr_decoded = base64.b64decode(result_xdr.encode())
    unpacker = Xdr.StellarXDRUnpacker(xdr_decoded)
    xdr_obj = unpacker.unpack_TransactionResult()
    print('Transaction result:', TransactionResultCode.get(xdr_obj.result.code))

def path_payment(signing_key, asset_code, asset_issuer, amount, receiving_key):

    # Talk to testnet horizon instance
    server = Server(horizon_url="https://horizon-testnet.stellar.org")

    # Derive Keypair object and public key from the signing key (source account)
    source_keypair = Keypair.from_secret(signing_key)
    source_public_key = source_keypair.public_key

    # PATH SEARCH
    # Create Horizon URL
    url = 'https://horizon-testnet.stellar.org/paths/strict-receive?source_account={}&destination_asset_type=credit_alphanum4&destination_asset_code={}&destination_asset_issuer={}&destination_amount={}&destination_account={}'

    # Get JSON response from Horizon
    r = requests.get(url.format(source_public_key, asset_code, asset_issuer, amount, receiving_key))
    path_search = r.json()
    send_max = path_search['_embedded']['records'][0]['source_amount']
    path = path_search['_embedded']['records'][0]['path']

    # LET'S CREATE THAT TRANSACTION
    # Fetch the current sequence number for the source account from Horizon.
    source_account = server.load_account(source_public_key)

    # Build transaction
    transaction = (
        TransactionBuilder(
            source_account=source_account,
            network_passphrase=Network.TESTNET_NETWORK_PASSPHRASE,
            base_fee=100,
        )
        .append_path_payment_strict_receive_op(destination=receiving_key, send_code='XLM', send_issuer=None, send_max=send_max, \
        dest_code=asset_code, dest_issuer=asset_issuer, dest_amount=amount, path=path, source=None)
        .build()
    )

    transaction.sign(source_keypair)

    # Submit the transaction to Horizon and check if successful
    response = server.submit_transaction(transaction)
    result_xdr = response.get('result_xdr')
    tx_success(result_xdr)
    print(json.dumps(response, indent=2))

if __name__ == '__main__':
    # STR Issuing Address
    str_issuer = 'GBEYFNS6KJRFEI22X5OBUFKQ5LK7Z2FZVFMAXBINC2SOCKA25AS62PUN'
    
    # Account A Public Key
    receiving_key = 'GBG7D5ZZJLAKPDBAGSVS3O3TMIV2O3HOIOXE2OSGGCYNRATOICDRTIAR'
    
    # Account B Private Key
    signing_key = 'SCO4JEHDN2ZLIDZMZC62UR6Y5NICMTMRKBPG3JMBKI5AHVXJA46MY2VG'
    
    # Run path payment :) 
    path_payment(signing_key, 'STR', str_issuer, '5', receiving_key)
```

It might seem like there's a lot going on in this script but it's really just a matter of grabbing the info from our path search and using it to fill in the path payment operation when building the transaction. The most important part is making sure your operation is filled out correctly: 

```python 
.append_path_payment_strict_receive_op(destination=receiving_key, send_code='XLM', send_issuer=None, send_max=send_max, \
dest_code=asset_code, dest_issuer=asset_issuer, dest_amount=amount, path=path, source=None)
```

**Quick tip:** That slash allows you to continue a piece of code on the next line. 

After running it you should be able to check the balance and notice ```Account A``` is 5 STR richer while ```Account B``` is down 9.96 XLM! Almost like magic. 

The explanation for this magic trick goes as follows: 

First, ```Account A``` gets credited with 5 STR tokens and ```Account B``` is debited 9.96 XLM. This is *basically* like a (really) short term, no interest loan. Next, ```Account B``` has to pay back that 'loan.' It does this buy filling an existing market order on the SDEX. In this case, someone had an order available that allowed us to buy 5 STR for 9.96 XLM. Sooo, the 9.96 XLM that was debited from ```Account B``` is given to the selling account to fulfill the order - making the world whole again. 

While this path payment example was pretty straight forward I encourage you to try one that is more intricate. Try finding a longer path between some assets and see what happens!

→ [Chapter 6 - XDRs](6-xdr.md)
