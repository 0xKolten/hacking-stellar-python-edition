[Home](README.md) - [Chapter 1](1-accounts.md) - [Chapter 2](2-payments.md) - [Chapter 3](3-assets.md) - [Chapter 4](4-decentralized-exchange.md) - [Chapter 5](5-path-payments.md) - [Chapter 6](6-xdr.md) - [Conclusion](7-conclusion.md)

## Chapter 6 - External Data Representation (XDR)

For the official documentation on XDRs - [go here](https://www.stellar.org/developers/guides/concepts/xdr.html).

### External Data Representation (XDR)

In short, XDR is a standard for the description and encoding of data. On Stellar, the ledger, transactions, results, history, and messages passed between computers running stellar-core are encoded using XDR. The benefits of XDR are that it's very compact, data compacted using XDR can be reliably and predictibly stored, and XDR definitions include rich descriptions of data types and structures.

As you've probably noticed in some of the earlier chapters, XDRs are included in many of the repsonses you get from Horizon. They look something like this:

``` json
"envelope_xdr": "AAAAAE3x9zlKwKeMIDSrLbtzYiunbO5Drk06RjCw2IJuQIcZAAAAZAAIScQAAAABAAAAAAAAAAAAAAABAAAAAAAAAAEAAAAAi8fAwnFjrnVjfFgRmYP9YMcYulQykFkCD5Kd7+0b6QgAAAAAAAAAADuaygAAAAAAAAAAAW5AhxkAAABA59dY2rInjjirjX43z8rStBUejRpUzxnELfZe4kiU55N0ms8TUxaG6iapCRXVFxR3PDbEqlUwzXvR/E31rnsQBw==",
"result_xdr": "AAAAAAAAAGQAAAAAAAAAAQAAAAAAAAABAAAAAAAAAAA=",
"result_meta_xdr": "AAAAAQAAAAIAAAADAAn7tQAAAAAAAAAATfH3OUrAp4wgNKstu3NiK6ds7kOuTTpGMLDYgm5AhxkAAAAXSHbnnAAIScQAAAAAAAAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAAAAAAAAAAAAAAAABAAn7tQAAAAAAAAAATfH3OUrAp4wgNKstu3NiK6ds7kOuTTpGMLDYgm5AhxkAAAAXSHbnnAAIScQAAAABAAAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAAAAAAAAAAAAAAAABAAAABAAAAAMACIU9AAAAAAAAAACLx8DCcWOudWN8WBGZg/1gxxi6VDKQWQIPkp3v7RvpCAAAABdIdugAAAiFPQAAAAAAAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAEACfu1AAAAAAAAAACLx8DCcWOudWN8WBGZg/1gxxi6VDKQWQIPkp3v7RvpCAAAABeEEbIAAAiFPQAAAAAAAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAMACfu1AAAAAAAAAABN8fc5SsCnjCA0qy27c2Irp2zuQ65NOkYwsNiCbkCHGQAAABdIduecAAhJxAAAAAEAAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAEACfu1AAAAAAAAAABN8fc5SsCnjCA0qy27c2Irp2zuQ65NOkYwsNiCbkCHGQAAABcM3B2cAAhJxAAAAAEAAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAA=="
```

The ```envelope_xdr``` is a base64 encoded string of the raw ```TransactionEnvelope``` and contains the transaction information and the signature generated using the secret key of the account that sent the payment. The ```result_xdr``` is a base64 encoded string of the raw ```TransactionResult``` and contains information about whether or not the transaction was successful. The ```result_meta_xdr``` is a base64 encoded string of the raw ```TransactionMeta``` that contains meta information about the transaction.

### Getting Transaction Results 

Throughout the chapters I've been using a function called ```tx_success(result_xdr)``` (thanks to [Overcat](https://github.com/overcat) for helping me with it). In short, this function lets us know whether or not a transaction was successful. After a transaction is submitted to Horizon the response will include a ```result_xdr``` containing the transaction result returned by Stellar-core. 

This function takes the ```result_xdr``` and decodes it using ```xdr_decoded = base64.b64decode(result_xdr.encode())```. Then the function needs to unpack the data from the XDR and does so via ```xdr_obj = unpacker.unpack_TransactionResult()```. After it is unpacked, we can access the transaction result code in the XDR object using the [dot operator] and print it to console: ```print('Transaction result:', TransactionResultCode.get(xdr_obj.result.code))```

All together it looks like this: 

``` python 
def tx_success(result_xdr):
    xdr_decoded = base64.b64decode(result_xdr.encode())
    unpacker = Xdr.StellarXDRUnpacker(xdr_decoded)
    xdr_obj = unpacker.unpack_TransactionResult()
    print('Transaction result:', TransactionResultCode.get(xdr_obj.result.code))
```

### Loading XDRs to Python Objects

So you're seeing these XDR objects everywhere and you're wondering how you can load them into a Python object and access data from them. Doing so is pretty straightforward and in this example I'll be using the ```envelope_xdr```.

``` json
"envelope_xdr": "AAAAAE3x9zlKwKeMIDSrLbtzYiunbO5Drk06RjCw2IJuQIcZAAAAZAAIScQAAAABAAAAAAAAAAAAAAABAAAAAAAAAAEAAAAAi8fAwnFjrnVjfFgRmYP9YMcYulQykFkCD5Kd7+0b6QgAAAAAAAAAADuaygAAAAAAAAAAAW5AhxkAAABA59dY2rInjjirjX43z8rStBUejRpUzxnELfZe4kiU55N0ms8TUxaG6iapCRXVFxR3PDbEqlUwzXvR/E31rnsQBw=="
```

To load it in to a Python object you can do the following:

``` python
from stellar_sdk.transaction_envelope import TransactionEnvelope

def envelope_data(envelope_xdr):
    password = 'Test SDF Network ; September 2015'

    tx_envelope = TransactionEnvelope.from_xdr(envelope_xdr, password)

if __name__ == '__main__':
    envelope_xdr = "AAAAAE3x9zlKwKeMIDSrLbtzYiunbO5Drk06RjCw2IJuQIcZAAAAZAAIScQAAAABAAAAAAAAAAAAAAABAAAAAAAAAAEAAAAAi8fAwnFjrnVjfFgRmYP9YMcYulQykFkCD5Kd7+0b6QgAAAAAAAAAADuaygAAAAAAAAAAAW5AhxkAAABA59dY2rInjjirjX43z8rStBUejRpUzxnELfZe4kiU55N0ms8TUxaG6iapCRXVFxR3PDbEqlUwzXvR/E31rnsQBw=="
    envelope_data(envelope_xdr)
```

From there, you can access to the different attributes of the object like so:

``` python
from stellar_sdk.transaction_envelope import TransactionEnvelope

def envelope_data(envelope_xdr):
    password = 'Test SDF Network ; September 2015'

    tx_envelope = TransactionEnvelope.from_xdr(envelope_xdr, password)

    print("Source account:", tx_envelope.transaction.source.public_key)
    print("Sequence number:", tx_envelope.transaction.sequence)
    print("Fee paid:", tx_envelope.transaction.fee)

if __name__ == '__main__':
    envelope_xdr = "AAAAAE3x9zlKwKeMIDSrLbtzYiunbO5Drk06RjCw2IJuQIcZAAAAZAAIScQAAAABAAAAAAAAAAAAAAABAAAAAAAAAAEAAAAAi8fAwnFjrnVjfFgRmYP9YMcYulQykFkCD5Kd7+0b6QgAAAAAAAAAADuaygAAAAAAAAAAAW5AhxkAAABA59dY2rInjjirjX43z8rStBUejRpUzxnELfZe4kiU55N0ms8TUxaG6iapCRXVFxR3PDbEqlUwzXvR/E31rnsQBw=="
    envelope_data(envelope_xdr)
```

If you run this script as is you should get a similar result to this: 

```
Source account: GBG7D5ZZJLAKPDBAGSVS3O3TMIV2O3HOIOXE2OSGGCYNRATOICDRTIAR
Sequence number: 2332905976102913
Feed paid: 100
```

The feed paid is denominated in stroops. I'd rather have it denominated in XLM so let's flesh this script out a little more: 

```python
from stellar_sdk.transaction_envelope import TransactionEnvelope

def stroops_to_xlm(stroops):
    s = float(stroops)
    xlm = s / 10000000
    return "{:.5f}".format(xlm)

def envelope_data(envelope_xdr):
    password = 'Test SDF Network ; September 2015'

    tx_envelope = TransactionEnvelope.from_xdr(envelope_xdr, password)

    print("Source account:", tx_envelope.transaction.source.public_key)
    print("Sequence number:", tx_envelope.transaction.sequence)
    print("Fee paid:", stroops_to_xlm(tx_envelope.transaction.fee), "XLM")

if __name__ == '__main__':
    envelope_xdr = "AAAAAE3x9zlKwKeMIDSrLbtzYiunbO5Drk06RjCw2IJuQIcZAAAAZAAIScQAAAABAAAAAAAAAAAAAAABAAAAAAAAAAEAAAAAi8fAwnFjrnVjfFgRmYP9YMcYulQykFkCD5Kd7+0b6QgAAAAAAAAAADuaygAAAAAAAAAAAW5AhxkAAABA59dY2rInjjirjX43z8rStBUejRpUzxnELfZe4kiU55N0ms8TUxaG6iapCRXVFxR3PDbEqlUwzXvR/E31rnsQBw=="
    envelope_data(envelope_xdr)
```

The ```stroops_to_xlm()``` function takes the fee attribute from the ```tx_envelope```, converts it to a float, then converts to lumens by dividing the amount of stroops by 10 million and finally returns it in a pretty format. 

Now when you run it you should get this: 

``` json 
Source account: GBG7D5ZZJLAKPDBAGSVS3O3TMIV2O3HOIOXE2OSGGCYNRATOICDRTIAR
Sequence number: 2332905976102913
Fee paid: 0.00001 XLM
```

Now that you know the basics of XDRs you should have all of the foundational knowledge you need to dive deeper in to the Stellar network!

→ [Conclusion](7-conclusion.md)
