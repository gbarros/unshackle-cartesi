# Unshackle-cartesi
Deploy the Cartesi Rollups framework on any EVM network

Cartesi is a Rollups framework than can be used with any EVM network. However, due to some development choices, it is currently only possible to deploy it on a few target networks. This project aims to unshackle Cartesi from this limitation.


Here you will be deploying the whole set of contracts, **have some gasToken with you before trying this.** 


## Docker images

I've changed the source code to of cartesi repos that develop the contracts and then build new images that accept any network configuration. If it doesn't find previous deployments, it deploys from scratch.

Use the following images:

- **Rollups-hardhat:**  gbbabarros/rollups-hardhat:1.0.1
- **Rollups-cli:** gbbabarros/rollups-cli:1.0.1


## Step-by-step

From the `rollup-examples` repository, follow the steps below:


### 1. Change `deploy-testnet.yml`

1) Change the `authority-deployer` image to `gbbabarros/rollups-hardhat:1.0.1`
2) Delete the deployer selector; Remove "--tags" and "Authority" lines.
3) Add the missing NETWORK variable to `authority-deployer`, you can copy from `dapp-deployer`.

**OR**

You can copy the file from this repo. [Find the "deploy-testnet.yml".](./deploy-testnet.yml)


### 2. Create your "env.customNet" file for deployment

Here is an example for polygon ZKEVM testnet:

```env
NETWORK=polygonZkEvmTestnet
CHAIN_ID=1442
BLOCK_CONFIRMATIONS=10
BLOCK_CONFIRMATIONS_TX=11
RPC_URL=https://rpc.public.zkevm-test.net
MNEMONIC="test test test test test test test test test test test junk"
```

Names of the networks can be found on the [WAGMI-CHAINS packages](https://wagmi.sh/core/chains). Prefer to use these names, as the framework will try to load RPC, ChainID and other information from the wagmi package. If you can't find yours be sure to fill correctly all the necessary network information, it will work just fine.


### 3. Add missing "deploymentFile" variable to `deploy.sh`

Under `/build/testnet-dapp-deployer` you will find the `deploy.sh` script. Add the following line to the `create` call:

```bash
    --deploymentFile "/deployments/$NETWORK/rollups.json" \
```

It should look like this:

```bash
cartesi-rollups create \
    --rpc "$RPC_URL" \
    --mnemonic "$MNEMONIC" \
    --deploymentFile "/deployments/$NETWORK/rollups.json" \
    --templateHashFile /var/opt/cartesi/machine-snapshots/latest/hash \
    --outputFile "/deployments/$NETWORK/$DAPP_NAME.json" \
    --consensusAddress "$CONSENSUS_ADDRESS"
```

### 4. Change the base image of `dapp-deployer`.

Also under `/build/testnet-dapp-deployer` you will find the `Dockerfile`. Change the base image to `gbbabarros/rollups-cli:1.0.1`.


## You are done! 

You can follow the normal deployment steps from the `rollup-examples` repository. Basically, run the following command, changing the dappname and the custom env file:

```bash 
DAPP_NAME=$NAME docker compose --env-file ../env.NETWORK_NAME -f  ../deploy-testnet.yml up
```


# Extra

## Reuse custom deploys

You can download the `rollups.json` file from the `deployments` folder for custom networks and use it to deploy your dapp. Look for the available deployment files in the [deployments folder](./deployments).

