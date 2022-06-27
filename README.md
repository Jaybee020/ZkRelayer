# Surrogeth

Zk Relayer is a general transaction callData that helps to broadcast tansactions to the network
that uses frontrunners as relayers network. It's designed to a case where a dapp's users
shouldn't be paying the gas costs for their transactions in order to improve anonymity.

The target contract does not need to know about the relayer system and just needs to have a mechanism to pay msg.sender a fee specified in the function encoded calldata.End users should also be have a way of picking a reliable way to pick a broadcaster for their transaction.

## Contents

The relayer system consists of three components.

| component | description                                                                                    |
| --------- | ---------------------------------------------------------------------------------------------- |
| server    | A node server that anyone can run to contribute to the network.                                |
| contracts | A trustless mechanism to help users discover nodes and help broadcasters forward transactions. |
| client    | Client-side lib for integrating your app with the relayer network.                             |

## Contracts

There are 2 main contracts that help in the relaying system. These include
| component | description |
| --------- | ---------------------------------------------------------------------------------------------- |
| Registry | This contract helps keeps track of transaction broadcasters(along with their respective locators).It also helps users to pick a suitable transaction relayer based on total fees and no of transactions relayed |
| Forwarder | A contract that helps call other contracts when provided with the right callData and contract Address. This contract then pays the relayer a fee |

### To run contracts test

```http
npx hardhat tests
```

## Server

After cloning this repo and installing dependencies. `cd` into this directory

```http
cd server
```

### Environment Variables and Config

Set up a local .env file that contains the following configuration variables

```http
    PRIVATE_KEY ="YOUR PRIVATE KEY";
    TESTNET_RPC_URL = "https://api.s0.ps.hmny.io";
    MIN_TX_PROFIT = YOUR PROFIT IN ONE;
    FEE = YOUR BASIC FEE IN ONE;
    PORT = PORT;
```

Then proceed to edit the config.js file

```js
export const PRIVATE_KEY = `${String(process.env.PRIVATE_KEY)}`;
export const TESTNET_RPC_URL = String(process.env.TESTNET_RPC_URL);
export const REGISTRY_CONTRACT = "0x4a0a5D875322De27e170f7c6E3678d47f711A50F"; //Registry Address
export const FORWARDER_CONTRACT = "0xF2E505107bbD79D9eb0C4EF475623A71BcDF6DE1"; //Forwarder Address
export const MIN_TX_PROFIT = String(process.env.MIN_TX_PROFIT);
export const FEE = String(process.env.FEE);
export const ENDPOINT = "localhost:3000"; //Your endpoint to be published on registry contract
export const MAINNET_DISALLOWED_RECIPIENTS = [""]; //Array of contract addresses not to relay to
export const PORT = parseInt(String(process.env.PORT));
```

### Running the Server

    To run the server on your machine

```http
npx ts-node app.ts
```

Then proceed to register yourself on the registry contract

```
npx ts-node register.ts
```

## Client

This is a light weight js module to help integrate your apps with the relayer system

### Get Relayer(s) Data

```js
export async function getRelayers(n = 1) {
  const provider = await getProvider("https://api.s0.ps.hmny.io");
  const client = new RegistryClient(provider);
  const relayersAddrs = await client.getRelayers(n);
  const relayersData = Promise.all(
    relayersAddrs.map(async (relayerAddr) => {
      const { count, sum } = await client.getRelayerFee(relayerAddr);
      const locator = await client.getRelayerLocator(relayerAddr);
      return {
        addr: relayerAddr,
        count: count.toNumber(),
        sum: sum.toNumber(),
        locator: locator,
      };
    })
  );
  return relayersData;
}
```

### To submit a transaction using a relayer locator

```js
export async function submitTx(relayerLocator, txn) {
  const provider = await getProvider("https://api.s0.ps.hmny.io");
  const client = new RegistryClient(provider);
  const result = await client.submitTx(relayerLocator, txn);
  return result;
}
```
