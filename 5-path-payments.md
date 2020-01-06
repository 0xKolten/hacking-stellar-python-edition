[Home](README.md) - [Chapter 1](1-accounts.md) - [Chapter 2](2-payments.md) - [Chapter 3](3-assets.md) - [Chapter 4](4-decentralized-exchange.md) - [Chapter 5](5-path-payments.md) - [Conclusion](6-conclusion.md) - [Bonus Chapter 1](bonus-xdr.md) - [Bonus Chapter 2](bonus-streaming.md)

## Chapter 5 - Path Payments on Stellar

<div align="center"><img width="25%" src="imgs/path-payment.png"></div>
<br>

For the official documentation on path payments - [go here](https://www.stellar.org/developers/guides/concepts/list-of-operations.html#path-payment).

Stellar's most unique feature is an operation called a ```Path Payment```. It allows users to send an asset through a path of offers, making it possible for the asset being sent (e.g. EUR) to be different from the asset received (e.g. USD).

To show off how path payments work, I'll make use of the STR token from last chapter. We will send lumens from ```Account A``` and have them converted to STR tokens by the time they arrive at ```Account B```'s address. One cool feature of path payments is that ```Account A``` (or the sender) does not have to create a trustline with the receiving asset since the sender doesn't actually hold the token during the payment.

### Finding a Payment Path

As I mentioned before, a path payment traverses a path of offers (up to 6) until it is converted to the desired asset. In order to make path payments easier to construct, we can execute a path search using the ```/paths``` Horizon endpoint and use the response to prefill the ```Path Payment``` operation. To specify a path search, we use the destination account id, the source account id, the asset and amount that the destination account should receive. Go [here](https://www.stellar.org/developers/horizon/reference/endpoints/path-finding.html#arguments) for the full list of ```/paths``` enpoint arguments.

### Constructing a Path Payment

First, let's see how we can find a path between two assets. In this example I'll be using what is called a [Strict Send Payment Path](https://www.stellar.org/developers/horizon/reference/endpoints/path-finding-strict-send.html). A strict send payment path allows a user to specify the amount of the asset to send. The amount received will vary based on offers in the order books. The alternative is a [Strict Receive Payment Path](https://www.stellar.org/developers/horizon/reference/endpoints/path-finding-strict-receive.html) and is simply the opposite of what we're using. 




→ [Conclusion & More Resources](6-conclusion.md)
