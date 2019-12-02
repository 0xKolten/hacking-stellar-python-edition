[Home](README.md) - [Chapter 1](1-accounts.md) - [Chapter 2](2-payments.md) - [Chapter 3](3-assets.md) - [Chapter 4](4-decentralized-exchange.md) - [Chapter 5](5-path-payments.md) - [Conclusion](6-conclusion.md) - [Bonus Chapter 1](bonus-xdr.md) - [Bonus Chapter 2](bonus-streaming.md)

## Chapter 5 - Path Payments on Stellar

<div align="center"><img width="25%" src="imgs/path-payment.png"></div>
<br>

For the official documentation on path payments - [go here](https://www.stellar.org/developers/guides/concepts/list-of-operations.html#path-payment).

Stellar's most unique feature is an operation called a ```Path Payment```. It allows users to send an asset through a path of offers, making it possible for the asset being sent (e.g. EUR) to be different from the asset received (e.g. USD).

### Finding a Payment Path

As I mentioned before, a path payment traverses a path of offers (up to 6) until it is converted to the desired asset. In order to make path payments easier to construct, we can execute a path search using the ```/paths``` Horizon endpoint and use the response to prefill the ```Path Payment``` operation. To specify a path search, we use the destination account id, the source account id, the asset and amount that the destination account should receive. Go [here](https://www.stellar.org/developers/horizon/reference/endpoints/path-finding.html#arguments) for the full list of ```/paths``` enpoint arguments.

### Constructing a Path Payment

To show off how path payments work, create a new account (```Account C```) and fund it with friendbot.

```
Account C
Public key: GBTAVRW74F2SJTT3USFHB7ZCRVNGWNXUYJASS3BOAZX4L6DOM7QLCARF
```

Then open up a trustline to a random testnet token like we did in [Chapter 4](4-decentralized-exchange.md). I'm going to trust a testnet token called POAT, issuing account: ```GAECL2FYQAMR2YFVCMOBBAIOOZGEAER6HART2MW7JWGNRDN53Q3S2WOB```, using the ```create_trust()``` script from [Chapter 3](3-assets.md).

Next, we'll write a script that finds a path from ```Account A```'s assets -> POAT and use that information to build a path payment from ```Account A``` to ```Account C```.

**Note**: ```Account A``` does not have to create a trustline with the POAT issuer as the account doesn't actually hold the token during the payment.

``` python
from stellar_base.builder import Builder # For next half of path payment function
import requests
import json

def path_payment(sending_key, receiving_key, asset_issuer, asset_code, amount):
    # Create Horizon URL
    url = 'https://horizon-testnet.stellar.org/paths?destination_account={}&source_account={}&destination_asset_type=credit_alphanum4&destination_asset_code={}&destination_asset_issuer={}&destination_amount={}'

    #Get JSON response
    r = requests.get(url.format(receiving_key, sending_key, asset_code, asset_issuer, amount))
    path_search = r.json()
    print(json.dumps(path_search, indent=2))

if __name__ == '__main__':
    poat_issuer = 'GAECL2FYQAMR2YFVCMOBBAIOOZGEAER6HART2MW7JWGNRDN53Q3S2WOB'
    receiving_key = 'GBTAVRW74F2SJTT3USFHB7ZCRVNGWNXUYJASS3BOAZX4L6DOM7QLCARF'
    sending_key = 'GD7YLRC3YWR3SMVGY3TSQ2UL56D7SG3JDIGYPNYZ2G22HBMOX5S7CLYF'
    path_payment(sending_key, receiving_key, poat_issuer, 'POAT', '5')
```

After running your script you should get a response similar to this:

``` json
{
  "_embedded": {
    "records": [
      {
        "source_asset_type": "credit_alphanum4",
        "source_asset_code": "FEE",
        "source_asset_issuer": "GDZZDS3QVBP2UMO2FFNXUJLDAQHFNFUQSJSROI3HWVCPKZ7YIFMTXLLY",
        "source_amount": "0.6399969",
        "destination_asset_type": "credit_alphanum4",
        "destination_asset_code": "POAT",
        "destination_asset_issuer": "GAECL2FYQAMR2YFVCMOBBAIOOZGEAER6HART2MW7JWGNRDN53Q3S2WOB",
        "destination_amount": "5.0000000",
        "path": [
          {
            "asset_type": "native"
          }
        ]
      },
      {
        "source_asset_type": "native",
        "source_amount": "3.7759815",
        "destination_asset_type": "credit_alphanum4",
        "destination_asset_code": "POAT",
        "destination_asset_issuer": "GAECL2FYQAMR2YFVCMOBBAIOOZGEAER6HART2MW7JWGNRDN53Q3S2WOB",
        "destination_amount": "5.0000000",
        "path": []
      }
    ]
  }
}
```

The path search from Horizon returned two possible paths to send an asset from ```Account A``` and convert it to POAT for ```Account C```. From FEE -> lumens -> POAT or from lumens -> POAT.

To make things fun we'll use the first path and go from FEE -> lumens -> POAT. We'll add some more functionality to our script to grab information from the get request and build our transaction:

``` python
from stellar_base.builder import Builder
import requests
import json

def path_payment(sending_key, receiving_key, asset_issuer, asset_code, amount):
    # Create Horizon URL
    url = 'https://horizon-testnet.stellar.org/paths?destination_account={}&source_account={}&destination_asset_type=credit_alphanum4&destination_asset_code={}&destination_asset_issuer={}&destination_amount={}'

    #Get JSON response
    r = requests.get(url.format(receiving_key, sending_key, asset_code, asset_issuer, amount))
    path_search = r.json()

    # Print path we are going to use to console
    print("\nPath we'll use:\n\n", json.dumps(path_search['_embedded']['records'][0], indent=2), "\n")

    # Get relevant info about send asset and payment path
    send_asset_code = path_search['_embedded']['records'][0]['source_asset_code']
    send_asset_issuer = path_search['_embedded']['records'][0]['source_asset_issuer']
    send_amount = path_search['_embedded']['records'][0]['source_amount']
    payment_path = [('XLM', None)]

    # Build transaction based on info
    # Don't forget to include Account A's secret key to sign the transaction
    builder = Builder(secret='SDRRZ2JQBI3RQSHV4Q63YWWJOVFNSXE2R6YSQKGUIPY65UATGUZUX4BB', horizon_uri='https://horizon-testnet.stellar.org' , network='testnet') \
        .append_path_payment_op(destination=receiving_key, send_code=send_asset_code, send_issuer=send_asset_issuer, send_max=send_amount, \
        dest_code=asset_code, dest_issuer=asset_issuer, dest_amount=amount, path=payment_path)
    # Sign and submit transaction -> print response
    builder.sign()
    response = builder.submit()
    print("\n", json.dumps(response, indent = 2))

if __name__ == '__main__':
    poat_issuer = 'GAECL2FYQAMR2YFVCMOBBAIOOZGEAER6HART2MW7JWGNRDN53Q3S2WOB'
    receiving_key = 'GBTAVRW74F2SJTT3USFHB7ZCRVNGWNXUYJASS3BOAZX4L6DOM7QLCARF'
    sending_key = 'GD7YLRC3YWR3SMVGY3TSQ2UL56D7SG3JDIGYPNYZ2G22HBMOX5S7CLYF'
    path_payment(sending_key, receiving_key, poat_issuer, 'POAT', '5')
```

First we grab information about the source asset (the asset we are sending). To do this, we use the variables ```send_asset_code``` to store the asset code "FEE", then we use ```send_asset_issuer``` to store the issuing address of the FEE token.

After that, we specified how much FEE token we would like to send in order to satisfy the amount of POAT we want ```Account C``` to receive.

**Note**: Path payments are specified using ```send max```, which is the maximum amount of ```send asset``` to send. Using a path search allows you easily specify a precise amount of ```send asset``` based on how much of the destination asset you need the second account to receive.

We also get information about the path asset(s) between FEE and POAT. In this case the path asset is the native asset (aka lumens).

Lastly we specified the full path our payment will go on. We do this by creating a list of tuples (asset_code, asset_issuer) for each asset *in* the path.

After gathering all the information, you simply append a path payment operation to the Builder object with all of the necessary fields and you're good to go!

If you wrote your script like mine, you should get the following response after running it:

```
Path we'll use:

 {
  "source_asset_type": "credit_alphanum4",
  "source_asset_code": "FEE",
  "source_asset_issuer": "GDZZDS3QVBP2UMO2FFNXUJLDAQHFNFUQSJSROI3HWVCPKZ7YIFMTXLLY",
  "source_amount": "0.6399969",
  "destination_asset_type": "credit_alphanum4",
  "destination_asset_code": "POAT",
  "destination_asset_issuer": "GAECL2FYQAMR2YFVCMOBBAIOOZGEAER6HART2MW7JWGNRDN53Q3S2WOB",
  "destination_amount": "5.0000000",
  "path": [
    {
      "asset_type": "native"
    }
  ]
}

```
``` json
 {
  "_links": {
    "transaction": {
      "href": "https://horizon-testnet.stellar.org/transactions/18f820253a22d16260d3d9c966de3d1b7f36fe48540968c84087ebdff09cbd25"
    }
  },
  "hash": "18f820253a22d16260d3d9c966de3d1b7f36fe48540968c84087ebdff09cbd25",
  "ledger": 134871,
  "envelope_xdr": "AAAAAP+FxFvFo7kypsbnKGqL74f5G2kaDYe3GdG1o4WOv2XxAAAAZAAAEeoAAAAIAAAAAAAAAAAAAAABAAAAAAAAAAIAAAABRkVFAAAAAADzkctwqF+qMdopW3olYwQOVpaQkmUXI2e1RPVn+EFZOwAAAAAAYafhAAAAAGYKxt/hdSTOe6SKcP8ijVprNvTCQSlsLgZvxfhuZ+CxAAAAAVBPQVQAAAAACCXouIAZHWC1ExwQgQ52TEASPjgjPTLfTYzYjb3cNy0AAAAAAvrwgAAAAAEAAAAAAAAAAAAAAAGOv2XxAAAAQFi6+NDJv14zIBcJ42txiGD30jpWdqCwSivRpkt7tH9uScbZzBu2Rze9TogT3S3rp/dtI/QnJagmKfOB7iTSYAI=",
  "result_xdr": "AAAAAAAAAGQAAAAAAAAAAQAAAAAAAAACAAAAAAAAAAIAAAAABNNV0mX+2gVUI+mvFhqLoZjXPkGM8kiHaYimxl3rqWoAAAAAAACiIAAAAAAAAAAAAkArRwAAAAFGRUUAAAAAAPORy3CoX6ox2ilbeiVjBA5WlpCSZRcjZ7VE9Wf4QVk7AAAAAABhp+EAAAAAcCiRhV1T/6agGQdDBvR1JTvQIVyKg+ASXRuYzO1OxXYAAAAAAACe2gAAAAFQT0FUAAAAAAgl6LiAGR1gtRMcEIEOdkxAEj44Iz0y302M2I293DctAAAAAAL68IAAAAAAAAAAAAJAK0cAAAAAZgrG3+F1JM57pIpw/yKNWms29MJBKWwuBm/F+G5n4LEAAAABUE9BVAAAAAAIJei4gBkdYLUTHBCBDnZMQBI+OCM9Mt9NjNiNvdw3LQAAAAAC+vCAAAAAAA==",
  "result_meta_xdr": "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"
}
```

After checking if the transaction was successful, we can repurpose ```show_address_data()``` one last time and check ```Account C```'s POAT balance:

```
Public key: GBTAVRW74F2SJTT3USFHB7ZCRVNGWNXUYJASS3BOAZX4L6DOM7QLCARF
Last modified ledger: 134871
POAT Balance: 5.0000000
```

And it worked 🤑

We can take an even closer look at the transaction by pulling it up on StellarExpert: https://stellar.expert/explorer/testnet/tx/18f820253a22d16260d3d9c966de3d1b7f36fe48540968c84087ebdff09cbd25

If you expand the operation, you'll see the process it went through:
1. Account credited - 5 POAT transferred to GBTA…CARF.
2. Account debited - 0.6399969 FEE transferred from GD7Y…CLYF.
3. Trade - GD7Y…CLYF bought 3.7759815 XLM for 0.6399969 FEE (offer 41504).
4. Trade - GACN…U2YG bought 0.6399969 FEE for 3.7759815 XLM (offer 41504).
5. Trade - GD7Y…CLYF bought 5 POAT for 3.7759815 XLM (offer 40666).
6. Trade - GBYC…MKW2 bought 3.7759815 XLM for 5 POAT (offer 40666).

What's even more cool is that these steps are considered 1 operation and costs a whopping 0.00001 lumen fee!

→ [Conclusion & More Resources](6-conclusion.md)
