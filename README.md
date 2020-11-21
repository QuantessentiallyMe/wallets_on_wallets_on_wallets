# wallets_on_wallets_on_wallets

![HD Wallet](https://weteachblockchain.org/assets/img/courses/bitcoin-for-developers/KWallets-02.jpg)

## Synopsis
_________________________________________________

The ideas of decentralized currency and the blockchain are truly taking the world by storm. There are numerous coins appearing every day, there may even be a Git coin. With all of this cyber money comes the need for protection. We store said funds in wallets and with these wallets we form public keys and private keys. But we have to do this for each transaction and especially with different currencies. Suppose you want to do a transaction with someone and they have Git coin but you have an Ethereum wallet? This creates many different generations of public and private keys which becomes quite cumbersome. 

Let me now introduce the HD Wallet. An HD Wallet or Hierarchal Deterministic wallet solves this problem by automatically generating a hierarchical tree-like structure of private/public addresses (or keys). They derive addresses from a single master seed (hence the name hierarchical). All HD wallets use a variant of the standard 12-word master seed key, and each time this seed is extended at the end by a counter value which makes it possible to automatically derive an unlimited number of new addresses. [You can look here for more information.](https://www.investopedia.com/terms/h/hd-wallet-hierarchical-deterministic-wallet.asp)

In this repository I will be exploring HD Wallets by using the command line tool hd-wallet-derive and creating a wallet that will manage Ethereum and Bitcoin. All files and instruction will be found here below from setup to transaction. Enjoy!

### Setup
_____________________________________________________

1. We will begin by creating a project directory. Here I will call said directory wallet and enter into said directory.

2. Once in the directory we will be cloning the "hd-wallet-derive" tool and installing everything necessary by using instructions found in its README.
    -- a. git clone https://github.com/dan-da/hd-wallet-derive
    
    -- b. cd hd-wallet-derive
    
    -- c. php -r "readfile('https://getcomposer.org/installer');" | php
    
    -- d. php composer.phar install



3.  We then create a symlink, which is just a reference to another file or directory, called derive for the hd-wallet-derive/hd-wallet-derive.php script into the top level project directory like so: ln -s hd-wallet-derive/hd-wallet-derive.php derive
This will allow us to call ./derive instead of ./hd-wallet-derive/hd-wallet-derive.php. Just to make things easier. 


4. To test that you can run the ./derive script properly, use one of the examples on the repo's README.md (the hd-wallet-derive repo).


5. Next create a file called wallet.py, this will be the universal wallet script.

![alt text](https://github.com/QuantessentiallyMe/wallets_on_wallets_on_wallets/blob/main/screenshots/cloning%20derive%20wallet.png)
![alt text](https://github.com/QuantessentiallyMe/wallets_on_wallets_on_wallets/blob/main/screenshots/cloning%20derive%20wallet%202.png)
![alt text](https://github.com/QuantessentiallyMe/wallets_on_wallets_on_wallets/blob/main/screenshots/cloning%20derive%20wallet%203.png)
![alt text](https://github.com/QuantessentiallyMe/wallets_on_wallets_on_wallets/blob/main/screenshots/cloning%20derive%20wallet%204.png)

6. You will also need bit and web3.py. If you do not have these two things already, follow my instructions here:

```
#Bit
pip install bit

#web3.py
pip install web3

#For Windows users, hopefully you have Anaconda and you should do these things in the Anaconda Prompt.
```

#### Setup Constants


1. We then create a seperate Python file called constants.py and set the following constants:

BTC = 'btc'
ETH = 'eth'
BTCTEST = 'btc-test'

2. Now we will go back to the wallet.py file and import these constants with the code:

```from constants import *```


3. We will now use these constants anytime we reference these strings in our code.

![alt text](https://github.com/QuantessentiallyMe/wallets_on_wallets_on_wallets/blob/main/screenshots/running%20wallet.py%20starter.png)

![alt text](https://github.com/QuantessentiallyMe/wallets_on_wallets_on_wallets/blob/main/screenshots/deriving%20wallet%20keys.png)

![alt text](https://github.com/QuantessentiallyMe/wallets_on_wallets_on_wallets/blob/main/screenshots/running%20wallet.py%202.png)


#### Generate A Mnemonic


1. You will need a 12 word mnemonic to seed from, one can be created with hd-wallet-derive. Generate one however you like. Here is a link where you can generate one just by clicking "Generate", [Awesome Link](https://iancoleman.io/bip39/).


2. We will set this mnemonic as an environment variable one can also include the new mnemonic as a fallback using like this:

```mnemonic = os.getenv('MNEMONIC', 'insert mnemonic here')```


#### Deriving The Wallet Keys


1. Now we will use the subprocess library to call the ./derive script from Python. Make sure to properly wait for the process.


2. We will pass the following flags into the shell command as variables:

-- Mnemonic (--mnemonic) must be set from an environment variable, or default to a test mnemonic
-- Coin (--coin)
-- Numderive (--numderive) to set number of child keys generated

3. We then set the --format=json flag, then parse the output into a JSON object using a function I call "derive_wallets". The wallet.py code should be looking something like this:

``` 
mnemonic = os.getenv('MNEMONIC')

def derive_wallets(coins):
    command = f'./derive -g --mnemonic="{mnemonic}" --coin={coins} --numderive=3 --cols=path,address,privkey,pubkey --format=json'
    p = subprocess.Popen(command, stdout=subprocess.PIPE, shell=True)
    output, err = p.communicate()
    p_status = p.wait()
    keys = json.loads(output)
    return keys   
```
6. Next we will create a dictionary called coins that derives ETH and BTCTEST wallets with this function like so:

```
coins= { ETH: derive_wallets(ETH), BTCTEST: derive_wallets(BTCTEST)}

print(coins)
```

When done properly, the final object should look something like this (there are only 3 children each in this image):

![alt text](https://github.com/QuantessentiallyMe/wallets_on_wallets_on_wallets/blob/main/screenshots/derive%20function.png)



#### Linking The Transaction Signing Libraries

Now, we need to use bit and web3.py to leverage the keys we've got in the coins object.

You will need to create three more functions:

1. Create the priv_key_to_account function. We will use this function to convert the privkey string in a child key to an account object
that bit or web3.py can use to transact.
```
def priv_key_to_account(coin, priv_key):
    if coin == ETH:
        return Account.privateKeyToAccount(priv_key)
    elif coin== BTCTEST:
        return PrivateKeyTestnet(priv_key)
```

2. Create the create_tx function. We will use this to create the raw, unsigned transaction that contains all metadata needed to transact.
```
def create_tx(coin, account, to, amount):
    if coin == ETH:
        gasEstimate=w3.eth.estimateGas(
        {"from":account.address,"to":recipient,"value":amount}
        )
        return {
            "from":account.address,
            "to":recipient,
            "value":amount,
            "gasPrice":w3.eth.gasPrice,
            "gas":gasEstimate,
            "nonce":w3.eth.getTransactionCount(account.address),
            }
    elif coin == BTCTEST:
        return PrivateKeyTestnet.prepare_transaction(account.address, [(to, amount, BTC)])
```
Here in the function you see that you have to check the coin and return different things for the different coins. 

3. Create the sent_tx function. We will us this to call create_tx, sign the transaction, then send it to the designated network.
```
def sent_tx(coin, account, to, amount):
    tx= create_tx(coin, account, to, amount)
    signed_tx = account.sign_transaction(tx)
    if coin == ETH:
        return w3.eth.sendRawTransaction(signed_tx.rawTransaction)
    elif coin == BTCTEST:
        return  NetworkAPI.broadcast_tx_testnet(signed)
```
You may notice these are the exact same parameters as create_tx. sent_tx will call create_tx, so it needs
all of this information available.
You will need to check the coin, then create a raw_tx object by calling create_tx. Then, you will need to sign
the raw_tx using bit or web3.py (hint: the account objects have a sign transaction function within).
Once you've signed the transaction, you will need to send it to the designated blockchain network.

For ETH, return w3.eth.sendRawTransaction(signed.rawTransaction)

For BTCTEST, return NetworkAPI.broadcast_tx_testnet(signed)

Below you will find screenshots of how everything should look. 
![alt text](https://github.com/QuantessentiallyMe/wallets_on_wallets_on_wallets/blob/main/screenshots/prev%20key%20function.png)
![alt text](https://github.com/QuantessentiallyMe/wallets_on_wallets_on_wallets/blob/main/screenshots/sent%20tx%20function.png)


### Send Some Transactions!
--------------------------------------------------------------------------------
Now, you should be able to fund these wallets using testnet faucets. Open up a new terminal window inside of wallet,
then run `python`. Within the Python shell, run `from wallet import *`. You will now be able to access the functions interactively.
You'll need to set the account with "priv_key_to_account" and use "send_tx" to send transactions.

#### Bitcoin Testnet Transaction


1. First we fund a BTCTEST address [using this testnet faucet.](https://testnet-faucet.mempool.co/)

2. Then we use a [block explorer](https://tbtc.bitaps.com/) to watch transactions on the address.

Here are some example screenshots: 
![alt text](https://github.com/QuantessentiallyMe/wallets_on_wallets_on_wallets/blob/main/screenshots/sending%20bitcoin%20testnet%20coins.png)

![alt text](https://github.com/QuantessentiallyMe/wallets_on_wallets_on_wallets/blob/main/screenshots/received%20BTCTEST%20coin.png)

#### Local PoA Ethereum Transaction

***NOTE: Here I will give somewhat of the steps and pictures of what I am doing and what you would need to do, but for this to work you would have had to already have created a PoA Ethereum network on your computer.***

1. First add one of the ETH addresses to the pre-allocated accounts in your networkname.json. (This would allow the address to do things on the network.)


2.Delete the geth folder in each node, then re-initialize using geth --datadir nodeX init networkname.json.
This will create a new chain, and will pre-fund the new account. 
(You would have previously created nodes with your network and they would be setup with the network and the data would be in the geth folder. Deleting this folder will allow the node to acknowledge the change done in step 1.)


3. Add the following middleware to web3.py to support the PoA algorithm:

```
from web3.middleware import geth_poa_middleware

w3.middleware_onion.inject(geth_poa_middleware, layer=0)
```

4.Due to a bug in web3.py, you will need to send a transaction or two with MyCrypto first, since the
w3.eth.generateGasPrice() function does not work with an empty chain. You can use one of the ETH address privkey,
or one of the node keystore files.


5. You then send a transaction from the pre-funded address within the wallet to another, then copy the txid into
MyCrypto's TX Status, and screenshot the successful transaction like so:


![alt text](https://github.com/QuantessentiallyMe/wallets_on_wallets_on_wallets/blob/main/screenshots/ETH%20TRANSACTION.png)



