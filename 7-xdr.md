[Home](README.md) - [Chapter 1](1-accounts.md) - [Chapter 2](2-payments.md) - [Chapter 3](3-assets.md) - [Chapter 4](4-decentralized-exchange.md) - [Chapter 5](5-path-payments.md) - [Conclusion](6-conclusion.md) - [Bonus Chapter 1](bonus-xdr.md) - [Bonus Chapter 2](bonus-streaming.md)

## Bonus Chapter - External Data Representation (XDR)

For the official documentation on XDRs - [go here](https://www.stellar.org/developers/guides/concepts/xdr.html).

### External Data Representation (XDR)

In short, XDR is a standard for the description and encoding of data. On Stellar, the ledger, transactions, results, history, and messages passed between computers running stellar-core are encoded using XDR. The benefits of XDR are that it's very compact, data compacted using XDR can be reliably and predictibly stored, and XDR definitions include rich descriptions of data types and structures.

As you've probably noticed in some of the earlier chapters, XDRs are included in many of the repsonses you get from Horizon. They look something like this:

``` json
"envelope_xdr": "AAAAAP+FxFvFo7kypsbnKGqL74f5G2kaDYe3GdG1o4WOv2XxAAAAyAAAEeoAAAABAAAAAAAAAAEAAAATSGVyZSdzIHNvbWUgbHVtZW5zIQAAAAACAAAAAAAAAAEAAAAABNNV0mX+2gVUI+mvFhqLoZjXPkGM8kiHaYimxl3rqWoAAAAAAAAAADuaygAAAAAAAAAABQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABAAAAC2ZlZC5uZXR3b3JrAAAAAAAAAAAAAAAAAY6/ZfEAAABApvgdnq5Tq80aR9FymeQBJAWXgHhMtcpjX59PC3ARtKNaHg5jlePBATov3KLtI9n3f8vOVPbYZ4yOU0xY8WFpAA==",
"result_xdr": "AAAAAAAAAMgAAAAAAAAAAgAAAAAAAAABAAAAAAAAAAAAAAAFAAAAAAAAAAA=",
"result_meta_xdr": "AAAAAQAAAAIAAAADAAASRwAAAAAAAAAA/4XEW8WjuTKmxucoaovvh/kbaRoNh7cZ0bWjhY6/ZfEAAAAXSHbnOAAAEeoAAAAAAAAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAAAAAAAAAAAAAAAABAAASRwAAAAAAAAAA/4XEW8WjuTKmxucoaovvh/kbaRoNh7cZ0bWjhY6/ZfEAAAAXSHbnOAAAEeoAAAABAAAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAAAAAAAAAAAAAAAACAAAABAAAAAMAABIoAAAAAAAAAAAE01XSZf7aBVQj6a8WGouhmNc+QYzySIdpiKbGXeupagAAABdIdugAAAASKAAAAAAAAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAEAABJHAAAAAAAAAAAE01XSZf7aBVQj6a8WGouhmNc+QYzySIdpiKbGXeupagAAABeEEbIAAAASKAAAAAAAAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAMAABJHAAAAAAAAAAD/hcRbxaO5MqbG5yhqi++H+RtpGg2HtxnRtaOFjr9l8QAAABdIduc4AAAR6gAAAAEAAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAEAABJHAAAAAAAAAAD/hcRbxaO5MqbG5yhqi++H+RtpGg2HtxnRtaOFjr9l8QAAABcM3B04AAAR6gAAAAEAAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAIAAAADAAASRwAAAAAAAAAA/4XEW8WjuTKmxucoaovvh/kbaRoNh7cZ0bWjhY6/ZfEAAAAXDNwdOAAAEeoAAAABAAAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAAAAAAAAAAAAAAAABAAASRwAAAAAAAAAA/4XEW8WjuTKmxucoaovvh/kbaRoNh7cZ0bWjhY6/ZfEAAAAXDNwdOAAAEeoAAAABAAAAAAAAAAAAAAAAAAAAC2ZlZC5uZXR3b3JrAAEAAAAAAAAAAAAAAAAAAAA="
```

The ```envelope_xdr``` is a base64 encoded string of the raw ```TransactionEnvelope``` and contains the transaction information and the signature generated using the secret key of the account that sent the payment. The ```result_xdr``` is a base64 encoded string of the raw ```TransactionResult``` and contains information about whether or not the transaction was successful. The ```result_meta_xdr``` is a base64 encoded string of the raw ```TransactionMeta``` that contains meta information about the transaction.

### Loading XDRs to Python Objects

So you're seeing these XDR objects everywhere and you're wondering how you can load them into a Python object and access data from them. Doing so is pretty straightforward and in this example I'll be using the ```envelope_xdr``` from above.

``` json
"envelope_xdr": "AAAAAP+FxFvFo7kypsbnKGqL74f5G2kaDYe3GdG1o4WOv2XxAAAAyAAAEeoAAAABAAAAAAAAAAEAAAATSGVyZSdzIHNvbWUgbHVtZW5zIQAAAAACAAAAAAAAAAEAAAAABNNV0mX+2gVUI+mvFhqLoZjXPkGM8kiHaYimxl3rqWoAAAAAAAAAADuaygAAAAAAAAAABQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABAAAAC2ZlZC5uZXR3b3JrAAAAAAAAAAAAAAAAAY6/ZfEAAABApvgdnq5Tq80aR9FymeQBJAWXgHhMtcpjX59PC3ARtKNaHg5jlePBATov3KLtI9n3f8vOVPbYZ4yOU0xY8WFpAA=="
```

To load it in to a Python object you can do the following:

``` python
from stellar_base.transaction_envelope import TransactionEnvelope

envelop_xdr = "AAAAAP+FxFvFo7kypsbnKGqL74f5G2kaDYe3GdG1o4WOv2XxAAAAyAAAEeoAAAABAAAAAAAAAAEAAAATSGVyZSdzIHNvbWUgbHVtZW5zIQAAAAACAAAAAAAAAAEAAAAABNNV0mX+2gVUI+mvFhqLoZjXPkGM8kiHaYimxl3rqWoAAAAAAAAAADuaygAAAAAAAAAABQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABAAAAC2ZlZC5uZXR3b3JrAAAAAAAAAAAAAAAAAY6/ZfEAAABApvgdnq5Tq80aR9FymeQBJAWXgHhMtcpjX59PC3ARtKNaHg5jlePBATov3KLtI9n3f8vOVPbYZ4yOU0xY8WFpAA=="

tx_envelope = TransactionEnvelope.from_xdr(envelope_xdr)
```

From there, you can access to the different attributes of the object like so:

``` python
from stellar_base.transaction_envelope import TransactionEnvelope

envelope_xdr = "AAAAAP+FxFvFo7kypsbnKGqL74f5G2kaDYe3GdG1o4WOv2XxAAAAyAAAEeoAAAABAAAAAAAAAAEAAAATSGVyZSdzIHNvbWUgbHVtZW5zIQAAAAACAAAAAAAAAAEAAAAABNNV0mX+2gVUI+mvFhqLoZjXPkGM8kiHaYimxl3rqWoAAAAAAAAAADuaygAAAAAAAAAABQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABAAAAC2ZlZC5uZXR3b3JrAAAAAAAAAAAAAAAAAY6/ZfEAAABApvgdnq5Tq80aR9FymeQBJAWXgHhMtcpjX59PC3ARtKNaHg5jlePBATov3KLtI9n3f8vOVPbYZ4yOU0xY8WFpAA=="

tx_envelope = TransactionEnvelope.from_xdr(envelope_xdr)

print("Source account:", tx_envelope.tx.source)
print("Sequence number:", tx_envelope.tx.sequence)
print("Fee:", tx_envelope.tx.fee)
```
```
Source account: b'GD7YLRC3YWR3SMVGY3TSQ2UL56D7SG3JDIGYPNYZ2G22HBMOX5S7CLYF'
Sequence number: 19696720019457
Fee: 200
```
