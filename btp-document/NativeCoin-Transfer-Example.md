# Transfer Native Coins betwen Moonriver and ICON Networks

In this section, we would like to instruct you how to transfer:

- A native coin of Moonriver network, e.g. `DEV`, to ICON network
- A native coin of ICON network, `ICX`, to Moonriver network

Let's light up a torch and get ready to transfer

## Transfer 'ICX' to Moonriver Network

____

- Generate Alice's keystore and address on ICON network

```bash
CONFIG_DIR=/path/to/INode

cd $CONFIG_DIR

goloop ks gen --out $CONFIG_DIR/alice.keystore.json --password YOUR_PASSWORD

# Create a secret file 'icon-bmr.secret' and save $YOUR_PASSWORD into that file
echo -n "YOUR_PASSWORD" > $CONFIG_DIR/alice.secret

# Save icon-bmr address to a file
echo -n "Alice Addresss" > $CONFIG_DIR/alice.addr
```

- Add "fuels" to Alice's account

```bash
AMOUNT=1000000000000000000000000

# transfer 'amount' of ICX coins to Alice's account
goloop rpc --uri http://127.0.0.1:9080/api/v3/icon sendtx transfer \
--to $(cat $CONFIG_DIR/alice.addr) --value $AMOUNT \
--key_store $CONFIG_DIR/godWallet.json --key_password gochain \
--step_limit 10000000000 --nid 3

# Check the balance of Alice
goloop rpc --uri http://127.0.0.1:9080/api/v3/icon balance $(cat $CONFIG_DIR/alice.addr)
```

- In this example, we would like to use pre-funds accounts, which was provided in the `truffle.config.js`, as receiver on Moonriver network. You can specify your account for an experiment

```bash
BTP_PROJ_DIR=/path/to/PRA-Contracts/btp

# Change your current directory to '.../solidity/bsh'
cd $BTP_PROJ_DIR/solidity/bsh

# Start truffle console
truffle console --network moonbeamlocal

# Get pre-funds accounts
truffle(moonbeamlocal)> let accounts = await web3.eth.getAccounts()

# Specify one account as destination to receive 'ICX'
truffle(moonbeamlocal)> accounts[1]
'0x3Cd0A705a2DC65e5b1E1205896BaA2be8A07c6e0'

# Get BSHCore contract instance
truffle(moonbeamlocal)> let bshCore = await BSHCore.deployed()

# Query balance of accounts[1] before receiving 'ICX' from Alice
truffle(moonbeamlocal)> await bshCore.getBalanceOf(accounts[1], 'ICX')

# Convert BigNumber to easy-reading number
truffle(moonbeamlocal)> web3.utils.BN(balance._usableBalance).toNumber()

# Exit truffle console (".exit") and save a responded address as destination
BOB_ADDR=returned/account/above
echo "btp:://$(cat $CONFIG_DIR/net.btp.dst)/$BOB_ADDR" > $CONFIG_DIR/bob.btp.addr
```

- Transfer 'ICX' from Alice to Bob

```bash
cd $CONFIG_DIR

# Change your amount if needed
AMOUNT=1000000
goloop rpc --uri http://127.0.0.1:9080/api/v3/icon sendtx call \
--to $(cat $CONFIG_DIR/nativeCoinBsh.icon) --method transferNativeCoin \
--param _to=$(cat $CONFIG_DIR/bob.btp.addr) --value $AMOUNT \
--key_store $CONFIG_DIR/alice.keystore.json --key_password $(cat $CONFIG_DIR/alice.secret) \
--step_limit 10000000000 --nid 3 | jq -r . > $CONFIG_DIR/tx.AliceToBob.transfer

# Check whether a transfer is successful
goloop rpc --uri http://127.0.0.1:9080/api/v3/icon txresult $(cat $CONFIG_DIR/tx.AliceToBob.transfer)
```

- Check balance of Bob's account after receiving 'ICX' from Alice

```bash
cd $BTP_PROJ_DIR/solidity/bsh

truffle console --network moonbeamlocal

truffle(moonbeamlocal)> let bshCore = await BSHCore.deployed()

# Query balance of accounts[1] after receiving 'ICX' from Alice
truffle(moonbeamlocal)> let balance = await bshCore.getBalanceOf(accounts[1], 'ICX')

# Convert BigNumber to easy-reading number
truffle(moonbeamlocal)> web3.utils.BN(balance._usableBalance).toNumber()
```

## Transfer 'DEV' to ICON Network

____

- Query balance of a receiving account before receiving 'DEV' from an account on Moonriver network

```bash
# In this example, we are going to use an abitrary account to demonstrate this transfer
# Please specify your own account if needed
echo -n "hx8062076aa5e68f021121d1c3b4b3979d21a6dcae" > $CONFIG_DIR/carol.addr
echo "btp:://$(cat $CONFIG_DIR/net.btp.icon)/$(cat $CONFIG_DIR/carol.addr)" > $CONFIG_DIR/carol.btp.addr

# Check the balance of Carol
goloop rpc --uri http://127.0.0.1:9080/api/v3/icon balance $(cat $CONFIG_DIR/carol.addr)
```

```bash
# Change your current directory to '.../solidity/bsh'
cd $BTP_PROJ_DIR/solidity/bsh

CAROL_ADDR=$(cat $CONFIG_DIR/carol.addr)
CAROL_BTP_ADDR=$(cat $CONFIG_DIR/carol.btp.addr)
AMOUNT=74706176

# Start truffle console
truffle console --network moonbeamlocal

# Get BSHCore contract instance
truffle(moonbeamlocal)> let bshCore = await BSHCore.deployed()

# Get pre-funds accounts
truffle(moonbeamlocal)> let accounts = await web3.eth.getAccounts()

# Check current balance of accounts[2] and make sure it has sufficient amount of coins
truffle(moonbeamlocal)> web3.eth.getBalance(accounts[2])

# Then, make a transfer request
# You can replace 'hx8062076aa5e68f021121d1c3b4b3979d21a6dcae' by your address on ICON network
# You can specify your own value to transfer
truffle(moonbeamlocal)> await bshCore.transferNativeCoin(process.env.CAROL_BTP_ADDR, {from: accounts[2], value: process.env.AMOUNT})

# After a few seconds, please check the account's balance again
# If success, the balance will be deducted. Otherwise, an account will be refunded
truffle(moonbeamlocal)> web3.eth.getBalance(accounts[2])
```

- Query balance of a receiving account after receiving 'DEV' from an account on Moonriver network

```bash
goloop rpc --uri http://127.0.0.1:9080/api/v3/icon balance $(cat $CONFIG_DIR/carol.addr)
```

&nbsp;

<p align="center">
<====================== END OF DOCUMENT ======================> 
</p>