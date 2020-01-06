[Home](README.md) - [Chapter 1](1-accounts.md) - [Chapter 2](2-payments.md) - [Chapter 3](3-assets.md) - [Chapter 4](4-decentralized-exchange.md) - [Chapter 5](5-path-payments.md) - [Conclusion](6-conclusion.md) - [Bonus Chapter 1](bonus-xdr.md) - [Bonus Chapter 2](bonus-streaming.md)

## Chapter 5 - Path Payments on Stellar

<div align="center"><img width="25%" src="imgs/path-payment.png"></div>
<br>

For the official documentation on path payments - [go here](https://www.stellar.org/developers/guides/concepts/list-of-operations.html#path-payment).

Stellar's most unique feature is an operation called a ```Path Payment```. It allows users to send an asset through a path of offers, making it possible for the asset being sent (e.g. EUR) to be different from the asset received (e.g. USD).

To show off how path payments work, I'll make use of the STR token from last chapter. We will send lumens from ```Account B``` and have them converted to STR tokens by the time they arrive at ```Account A```'s address. 

### Finding a Payment Path

As I mentioned before, a path payment traverses a path of offers (up to 6) until it is converted to the desired asset. In order to make path payments easier to construct, we can execute a path search using the ```/paths``` Horizon endpoint and use the response to prefill the ```Path Payment``` operation. To specify a path search, we use the destination account id, the source account id, the asset and amount that the destination account should receive. Go [here](https://www.stellar.org/developers/horizon/reference/endpoints/path-finding.html#arguments) for the full list of ```/paths``` enpoint arguments.

Let's see how we can find a path between two assets. In this example I'll be using what is called a [Strict Receive Payment Path](https://www.stellar.org/developers/horizon/reference/endpoints/path-finding-strict-receive.html). A strict receive payment path allows a user to specify the amount of the asset being receieved. The amount sent will vary based on offers in the order books. The alternative is a [Strict Send Payment Path](https://www.stellar.org/developers/horizon/reference/endpoints/path-finding-strict-send.html) and is simply the opposite of what we're using. 

``` python 
import requests
import json

def path_payment(sending_key, asset_code, asset_issuer, amount, receiving_key):
    # Create Horizon URL
    url = 'https://horizon-testnet.stellar.org/paths/strict-receive?source_account={}&destination_asset_type=credit_alphanum4&destination_asset_code={}&destination_asset_issuer={}&destination_amount={}&destination_account={}'

    # Get JSON response from Horizon
    r = requests.get(url.format(sending_key, asset_code, asset_issuer, amount, receiving_key))
    path_search = r.json()
    print(json.dumps(path_search, indent=2))

if __name__ == '__main__':
    str_issuer = 'GBEYFNS6KJRFEI22X5OBUFKQ5LK7Z2FZVFMAXBINC2SOCKA25AS62PUN'
    receiving_key = 'GBG7D5ZZJLAKPDBAGSVS3O3TMIV2O3HOIOXE2OSGGCYNRATOICDRTIAR'
    sending_key = 'GCF4PQGCOFR245LDPRMBDGMD7VQMOGF2KQZJAWICB6JJ337NDPUQR66E'
    path_payment(sending_key, 'STR', str_issuer, '5', receiving_key)
```

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

### Constructing a Path Payment



→ [Conclusion & More Resources](6-conclusion.md)
