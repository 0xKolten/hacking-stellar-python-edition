[Home](README.md) - [Chapter 1](1-accounts.md) - [Chapter 2](2-payments.md) - [Chapter 3](3-assets.md) - [Chapter 4](4-decentralized-exchange.md) - [Chapter 5](5-path-payments.md) - [Conclusion](6-conclusion.md) - [Bonus Chapter 1](bonus-xdr.md) - [Bonus Chapter 2](bonus-streaming.md)

## Chapter 2 - Payments on Stellar

<div align="center"><img width="40%" src="imgs/send-money-anywhere.png"></div>
<br>

For the official documentation on payments - [go here](https://www.stellar.org/developers/guides/concepts/list-of-operations.html#payment).

As a Stellar user, you have the option to execute many types of [operations](https://www.stellar.org/developers/guides/concepts/operations.html). Payments are one type of operation and are simply the sending of a specific amount of an asset to a destination.

**Note**: On Stellar, transactions contain a *list of operations* and other relevant information such as fees, signatures, etc. In short, transactions are simply commands that alter the state of the ledger. Each operation you add to a transaction costs a **base fee** of 100 stroops (0.00001 lumens), so the total transaction fee is the number of operations multiplied by the base fee. For the official documentation on transactions - [go here](https://www.stellar.org/developers/guides/concepts/transactions.html).

### Sending a Payment

Before sending our first payment, we need to generate a new address to serve as our payment destination. Run the ```generate_keypair()``` function we created in the previous chapter to get a new address and fund it using Friendbot.

You should now have keys for two accounts, both with 10,000 lumens to send:

```
Account A
Public key: GBG7D5ZZJLAKPDBAGSVS3O3TMIV2O3HOIOXE2OSGGCYNRATOICDRTIAR
Private key: SBK4EAZIWXELREKEXP4WB6DCCMJH7SGTEQE2BJALA32VQQ4ADFAWJGOV
```
```
Account B
Public key: GCF4PQGCOFR245LDPRMBDGMD7VQMOGF2KQZJAWICB6JJ337NDPUQR66E
Private key: SCO4JEHDN2ZLIDZMZC62UR6Y5NICMTMRKBPG3JMBKI5AHVXJA46MY2VG
```

To send our first payment from ```Account A``` to ```Account B```, we'll write a script that builds a transaction for our payment operation and checks if that transaction was successful - I'll explain how it all works in the next section:

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

def send_payment(signing_key, receiving_key, amount, asset='XLM'):
    # Derive Keypair object and public key from the signing key (source account)
    source_keypair = Keypair.from_secret(signing_key)
    source_public_key = source_keypair.public_key

    # Account receiving funds
    receiving_key = receiving_key

    # Talk to testnet horizon instance
    server = Server(horizon_url="https://horizon-testnet.stellar.org")

    # Fetch the current sequence number for the source account from Horizon.
    source_account = server.load_account(source_public_key)

    # Build transaction
    transaction = (
        TransactionBuilder(
            source_account=source_account,
            network_passphrase=Network.TESTNET_NETWORK_PASSPHRASE,
            base_fee=100,
        )
        .append_payment_op(receiving_key, amount, asset)
        .build()
    )

    transaction.sign(source_keypair)

    # Submit the transaction to Horizon and check if successful
    response = server.submit_transaction(transaction)
    result_xdr = response.get('result_xdr')
    tx_success(result_xdr)
    print(json.dumps(response, indent=2))

if __name__ == '__main__':
    # Account A Signing Key: SBK4EAZIWXELREKEXP4WB6DCCMJH7SGTEQE2BJALA32VQQ4ADFAWJGOV
    # Account B Public Key: GCF4PQGCOFR245LDPRMBDGMD7VQMOGF2KQZJAWICB6JJ337NDPUQR66E
    send_payment('SBCQT2KDQNBP3H4ONGLCY2QRD2EXDMVG7REODUIPWRBTENAIGCHTBD6V', 'GCCNQCV26F4DZVOOKQBNZBW7JNXDRNHNCSVGYQOXKX27RWEAXWCMH3IZ', '100')
 ```

Run the script and if everything went according to plan, you should've gotten a response like this:
``` json
Transaction result: txSUCCESS
{
  "_links": {
    "transaction": {
      "href": "https://horizon-testnet.stellar.org/transactions/ab2391d09a662b3d94ee274f54015542906613034142ad16dc35a69377836b5a"
    }
  },
  "hash": "ab2391d09a662b3d94ee274f54015542906613034142ad16dc35a69377836b5a",
  "ledger": 654261,
  "envelope_xdr": "AAAAAE3x9zlKwKeMIDSrLbtzYiunbO5Drk06RjCw2IJuQIcZAAAAZAAIScQAAAABAAAAAAAAAAAAAAABAAAAAAAAAAEAAAAAi8fAwnFjrnVjfFgRmYP9YMcYulQykFkCD5Kd7+0b6QgAAAAAAAAAADuaygAAAAAAAAAAAW5AhxkAAABA59dY2rInjjirjX43z8rStBUejRpUzxnELfZe4kiU55N0ms8TUxaG6iapCRXVFxR3PDbEqlUwzXvR/E31rnsQBw==",
  "result_xdr": "AAAAAAAAAGQAAAAAAAAAAQAAAAAAAAABAAAAAAAAAAA=",
  "result_meta_xdr": "AAAAAQAAAAIAAAADAAn7tQAAAAAAAAAATfH3OUrAp4wgNKstu3NiK6ds7kOuTTpGMLDYgm5AhxkAAAAXSHbnnAAIScQAAAAAAAAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAAAAAAAAAAAAAAAABAAn7tQAAAAAAAAAATfH3OUrAp4wgNKstu3NiK6ds7kOuTTpGMLDYgm5AhxkAAAAXSHbnnAAIScQAAAABAAAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAAAAAAAAAAAAAAAABAAAABAAAAAMACIU9AAAAAAAAAACLx8DCcWOudWN8WBGZg/1gxxi6VDKQWQIPkp3v7RvpCAAAABdIdugAAAiFPQAAAAAAAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAEACfu1AAAAAAAAAACLx8DCcWOudWN8WBGZg/1gxxi6VDKQWQIPkp3v7RvpCAAAABeEEbIAAAiFPQAAAAAAAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAMACfu1AAAAAAAAAABN8fc5SsCnjCA0qy27c2Irp2zuQ65NOkYwsNiCbkCHGQAAABdIduecAAhJxAAAAAEAAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAEACfu1AAAAAAAAAABN8fc5SsCnjCA0qy27c2Irp2zuQ65NOkYwsNiCbkCHGQAAABcM3B2cAAhJxAAAAAEAAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAA=="
}
```

Now we can use the ```show_balance()``` script that we created last chapter and take a look at both account balances:

```
Account A
Lumen Balance: 9899.9999900 XLM
```
```
Account B
Lumen Balance: 10100.0000000 XLM
```

Perfect, it worked 😎 Let's break down what we did.

### Unpacking the Payment

To understand what's going on we first need to understand the ```send_payment()``` function part of the script:

``` python
def send_payment(signing_key, receiving_key, amount, asset='XLM'):
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
        .append_payment_op(receiving_key, amount, asset)
        .build()
    )

    transaction.sign(source_keypair)

    # Submit the transaction to Horizon and check if successful
    response = server.submit_transaction(transaction)
    result_xdr = response.get('result_xdr')
    tx_success(result_xdr)
    print(json.dumps(response, indent=2))
```

The first thing to highlight are the 4 variables that ```send_payment()``` takes as input: 
- ```signing_key```
- ```receiving_key```
- ```amount```
- ```asset```

These four inputs are all we need to make a simple lumen payment. The ```signing_key``` allows the sender, or source account, to sign off on the transaction with their private key. The ```receiving_key``` is the receiving user's public key. The ```amount```, is the amount of an asset we are sending, and ```asset``` specifies that we are sending lumens. 

The first part of the function creates a server object and tells it to communicate with the public Horizon testnet instance. From the SDK's documentation: *Server handles the network connection to a Horizon instance and exposes an interface for requests to that instance.*

The next part of function takes the ```signing_key``` and uses it to derive the source account's keypair. We do this so we can get the source accounts [sequence number](https://www.stellar.org/developers/guides/concepts/transactions.html#sequence-number) in order to build the transaction. Sequence numbers are associated with the source account sending a transaction and follow a strict ordering rule in order to prevent [double-spending](https://en.wikipedia.org/wiki/Double-spending). 

At this point we are ready to build the transaction. The transaction is built around the ```source_account```, ```network_passphrase``` (which network is being used), and the ```base_fee``` - which is 100 stroops or 0.00001 XLM. Quick note on the fee, this is subject to change so it is generally best to check what the fee should be vs assuming the minimum. You can do this by using ```base_fee = server.fetch_base_fee()``` to get the latest [base fee](https://www.stellar.org/developers/guides/concepts/fees.html#base-fee) from the ledger. 

From there we append a payment operation and specify the receiving address, amount, and asset we are sending. Using this notation we can append other types of operations as well. Then we call ```.build()``` to build the ```TransactionEnvelope``` and increment the source account's sequence number by 1. From there the ```TransactionEnvelope``` is signed with the source account's signing key and it gets submitted to Horizon and the response is printed to console. 

The response includes: 
- ```_links```
- ```hash```
- ```ledger```
- ```envelope_xdr```
- ```result_xdr```
- ```result_meta_xdr```

One of the most important parts of this response is the ```result_xdr```. We can pass the ```result_xdr``` to the ```tx_success``` function written in the script to verify that the transaction was actually successful. This important because even failed transactions can be included in the ledger. 

I cover XDRs in more depth in Chapter 7, but what's important to know for now is that XDRs are used to encode data and pass that data around the Stellar network. The ```tx_success()``` function decodes the ```result_xdr``` to let us know wether or not our transaction was successful. 

Now that you have some experience creating and sending payments, try sending different amounts back and forth between your accounts and getting familar with the responses from Horizon.

Next we'll learn about Stellar assets and create one of our own.

→ [Chapter 3 - Assets](3-assets.md)
