# Infrastructure
This a infrasture code for the cattlechain project.

**Important: all the file are development env only (don't use for production)**

## Prerequisites

- [Docker and Docker-compose](https://docs.docker.com/compose/install/)

| ⚠️ **Note**: If on MacOS or Windows, please ensure that you allow docker to use upto 4G of memory or 6G if running Privacy examples under the _Resources_ section. The [Docker for Mac](https://docs.docker.com/docker-for-mac/) and [Docker Desktop](https://docs.docker.com/docker-for-windows/) sites have details on how to do this at the "Resources" heading       |
| ---                                                                                                                                                                                                                                                                                                                                                                                |


| ⚠️ **Note**: This has only been tested on Windows 10 Build 18362 and Docker >= 17.12.2                                                                                                                                                                                                                                                                                              |
| ---                                                                                                                                                                                                                                                                                                                                                                                |

- On Windows ensure that the drive that this repo is cloned onto is a "Shared Drive" with Docker Desktop
- On Windows we recommend running all commands from GitBash
- [Nodejs](https://nodejs.org/en/download/) or [Yarn](https://yarnpkg.com/cli/node)


## 1. Running Orion
```sh 
docker-compose -f orion up
```
this will run the following service:

1. Orion Context Broker (NGSI-LD)
2. Mongodb


## 2. Running KeyRock
```sh 
docker-compose -f keyrock up
```
this will run the following service:

1. KeyRock
2. MySQL

## 3. Running PEP Proxy
```sh 
docker-compose -f pep-proxy up
```
this will run the following service:

1. PEP Proxy


## 4. Running DRACO
```sh 
docker-compose -f draco up
```
this will run the following service:

1. draco
2. Apache NIFI


## 5. Running Canis Major
```sh 
docker-compose -f canis-major up
```
this will run the following service:

1. Canis-Major Adaptor
2. MySQL


# Running Quorum Dev Blockchain
## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Usage](#usage)
3. [Dev Network Setups](#dev-network-setups)
    1. [POA Network](#poa-network)
    2. [POA Network with Privacy](#poa-network-privacy)
    3. [Smart Contracts & DApps](#poa-network-dapps)
                  

## Usage 

Change directory to the artifacts folder: 

`cd quorum-test-network` (default folder location) 
 
**To start services and the network:**

`./run.sh` starts all the docker containers

**To stop services :**

`./stop.sh` stops the entire network, and you can resume where it left off with `./resume.sh` 

`./remove.sh ` will first stop and then remove all containers and images


## Dev Network Setups
All our documentation can be found on the [Besu documentation site](https://besu.hyperledger.org/Tutorials/Examples/Private-Network-Example/).

Each quickstart setup is comprised of 4 validators, one RPC node and some monitoring tools like:
- [Alethio Lite Explorer](https://besu.hyperledger.org/en/stable/HowTo/Deploy/Lite-Block-Explorer/) to explore blockchain data at the block, transaction, and account level
- [Metrics monitoring](https://besu.hyperledger.org/en/stable/HowTo/Monitor/Metrics/) via Prometheus and Grafana to give you insights into how the chain is progressing (only with Besu based Quorum)
- Optional [logs monitoring](https://besu.hyperledger.org/en/latest/HowTo/Monitor/Elastic-Stack/) to give you real time logs of the nodes. This feature is enabled with a `-e` flag when starting the sample network

The overall architecture diagrams to visually show components of the blockchain networks is shown below. 
**Consensus Algorithm**: The Besu based Quorum variant uses the `IBFT2` consensus mechanism.
**Private TX Manager**: The Besu based Quorum variant uses [Orion](https://github.com/PegaSysEng/orion)

![Image blockchain](./static/blockchain-network.png)
 

### i. POA Network <a name="poa-network"></a>

This is the simplest of the networks available and will spin up a blockchain network comprising 4 validators, 1 RPC 
node which has an [EthSinger](http://docs.ethsigner.consensys.net/) proxy container linked to it so you can optionally sign transactions. To view the progress 
of the network, the Alethio block explorer can be used and is available on `http://localhost:25000`. 
Hyperledger Besu based Quorum also deploys metrics monitoring via Prometheus available on `http://localhost:9090`, 
paired with Grafana with custom dashboards available on `http://localhost:3000`. 

Essentially you get everything in the architecture diagram above, bar the yellow privacy block

Use cases: 
 - you are learning about how Ethereum works 
 - you are looking to create a Mainnet or Ropsten node but want to see how it works on a smaller scale
 - you are a DApp Developer looking for a robust, simple network to use as an experimental testing ground for POCs. 
 
 
### ii. POA Network with Privacy <a name="poa-network-privacy"></a>

This network is slightly more advanced than the former and you get everything from the POA network above and a few 
Ethereum clients each paired with a Private Transaction Mananger. The Besu based Quorum variant uses [Orion](https://github.com/PegaSysEng/orion) for it's Private Transaction Mananger.

As before, to view the progress of the network, the Alethio block explorer can be used and is available on `http://localhost:25000`. 
Hyperledger Besu based Quorum also deploys metrics monitoring via Prometheus available on `http://localhost:9090`, 
paired with Grafana with custom dashboards available on `http://localhost:3000`. 

Essentially you get everything in the architecture diagram above.

Use cases:
- you are learning about how Ethereum works
- you are a user looking to execute private transactions at least one other party
- you are looking to create a private Ethereum network with private transactions between two or more parties.

Once the network is up and running you can send a private transaction between members and verify that other nodes do not see it.
Under the smart_contracts folder there is an `EventEmitter` contract which can be deployed and tested by running:
```
cd smart_contracts
npm install
node scripts/deploy.js
```
which deploys the contract and sends an arbitrary value (47) from `Member1` to `Member3`. Once done, it queries all three members (orion)
to check the value at an address, and you should observe that only `Member1` & `Member3` have this information as they were involved in the transaction 
and that `Member2` responds with a `0x` to indicate it is unaware of the transaction.

```
node scripts/deploy.js 
Creating contract...
Getting contractAddress from txHash:  0x10e8e9f46c7043f87f92224e065279638523f5b2d9139c28195e1c7e5ac02c72
Waiting for transaction to be mined ...
Contract deployed at address: 0x649f1dff9ca6dfbdd27135c94171334ea0fab5ee

Transaction Hash: 0x30b53a533afe909aee59df716e07f7003c0605075a13f97799b29cdd3c2c42a7
Waiting for transaction to be mined ...
Transaction Hash: 0x181e37e64cdfb8d3cb0f076ee63045981436f3273942bac47820c7ec1aad0c23
Transaction Hash: 0xa27db2772689fe8ca995d32d1753d2695421120c9f171d6d32eb0873f2b96466
Waiting for transaction to be mined ...
Waiting for transaction to be mined ...
Member3 value from deployed contract is: 0x000000000000000000000000000000000000000000000000000000000000002f
Member1 value from deployed contract is: 0x000000000000000000000000000000000000000000000000000000000000002f
Member2 value from deployed contract is: 0x
```

Further [documentation](https://besu.hyperledger.org/en/stable/Tutorials/Privacy/eeajs-Multinode-example/) 

There is an additional erc20 token example that you can also test with: executing `node example/erc20.js` deploys a `HumanStandardToken` contract and transfers 1 token to Node2.

This can be verified from the `data` field of the `logs` which is `1`.
