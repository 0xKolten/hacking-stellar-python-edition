[Home](README.md) - [Chapter 0](0-setup.md) - [Chapter 1](1-accounts.md) - [Chapter 2](2-payments.md) - [Chapter 3](3-assets.md) - [Chapter 4](4-decentralized-exchange.md) - [Chapter 5](5-path-payments.md) - [Conclusion](6-conclusion.md)

## Chapter 0 - Setup

### Downloads

Before we start writing any code and interfacing with the Stellar network, it's important that you have things set up and ready to go.

Since the following chapters only use Python you should make sure you have Python installed (duh). This step is relatively straight forward and you may already have it completed. On the off chance you haven't downloaded Python, you can do so [here](https://www.python.org/).

Most importantly, if you have never written, compiled, or run Python scripts before ***please*** read this blog before getting started—[How to Run Your Python Scripts](https://realpython.com/run-python-scripts/). This will save you from a lot of headaches. 

Next, you should install the [Python Stellar SDK](https://github.com/StellarCN/py-stellar-base). The Python Stellar SDK is a Python library for interfacing with a [Horizon API](https://horizon.stellar.org/)—if you don't know what this is yet no worries, we'll get there.

To install the SDK, you can open up your command line and type the command:

``` json
pip install stellar-sdk==2.0.0

```

Lastly, the Python Stellar SDK's documentation can be found [here](https://stellar-sdk.readthedocs.io/en/latest/). The docs include code examples, descriptions of the objects and classes we'll use, and all the other good stuff you'll need to know. Much of the information presented throughout these chapters is a combination of the [SDK docs](https://stellar-sdk.readthedocs.io/en/latest/) and the [docs from Stellar.org](https://www.stellar.org/developers/guides/get-started/). Having these available to quickly reference will surely help along the way.


### Housekeeping

I mentioned some of this stuff in the prerequisites, but I wanted to mention them again to make sure we start off on the right foot.

If you are new to Stellar, the best way to get a brief introduction is the [30 minute course](https://www.lumenauts.com/courses/stellar-overview-course) put together by [Lumenauts](https://www.lumenauts.com).

Another resource you may find valuable is [Awesome Stellar](https://awesomestellar.com/)—a curated list of Stellar applications, blog posts, educational resources, tools, and more.

Additionally, if it's been a while since you last worked with APIs and JSON responses, now would be a good time to go watch a quick refresher on YouTube. Here's one on [Rest APIs](https://youtu.be/7YcW25PHnAA) and here's one that covers [JSON](https://youtu.be/iiADhChRriM).

Doing all this in preparation for Hacking Stellar should put you in a good place to get started. There are many additional resources shared throughout the chapters that are worth checking out and digging in to.

Without further ado... Let's get started.

→ [Chapter 1 - Accounts](1-accounts.md)
