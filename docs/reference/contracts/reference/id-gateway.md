---
outline: [2, 3]
---

# ID Gateway

The ID Gateway registers new Farcaster IDs and adds them to the [ID Registry](/reference/contracts/reference/id-registry.md).

If you want to create a new Farcaster ID, use the ID Gateway.

## Read

### price

Get the price in wei to register an fid. This includes the price of 1 storage unit. Use the `extraStorage` parameter to include extra storage units in the total price.

| Parameter    | type                 | Description                            |
| ------------ | -------------------- | -------------------------------------- |
| extraStorage | `uint256` (optional) | The number of additional storage units |

### nonces

Get the next unused nonce for an address. Used for generating an EIP-712 [`Register`](#register-signature) signature for [registerFor](#registerfor).

| Parameter | type      | Description                  |
| --------- | --------- | ---------------------------- |
| owner     | `address` | Address to get the nonce for |

## Write

### register

Register a new fid to the caller and pay for storage. The caller must not already own an fid.

| Parameter    | type                 | Description                                |
| ------------ | -------------------- | ------------------------------------------ |
| `msg.value`  | `wei`                | Amount to pay for registration             |
| recovery     | `address`            | Recovery address for the new fid           |
| extraStorage | `uint256` (optional) | Number of additional storage units to rent |

### registerFor

Register a new fid to a specific address and pay for storage. The receiving
address must sign an EIP-712 [`Register`](#register-signature) message approving the registration. The receiver must not already own an fid.

| Parameter    | type                 | Description                                        |
| ------------ | -------------------- | -------------------------------------------------- |
| `msg.value`  | `wei`                | Amount to pay for registration                     |
| to           | `address`            | The address to register the fid to                 |
| recovery     | `address`            | Recovery address for the new fid                   |
| deadline     | `uint256`            | Signature expiration timestamp                     |
| sig          | `bytes`              | EIP-712 `Register` signature from the `to` address |
| extraStorage | `uint256` (optional) | Additional storage units                           |

#### Register signature

To register an fid on behalf of another account, you must provide an EIP-712 typed signature from the receiving address in the following format:

`Register(address to,address recovery,uint256 nonce,uint256 deadline)`

| Parameter | type      | Description                                                                       |
| --------- | --------- | --------------------------------------------------------------------------------- |
| to        | `address` | Address to register the fid to. The typed message must be signed by this address. |
| recovery  | `address` | Recovery address for the new fid                                                  |
| nonce     | `uint256` | Current nonce of the `to` address                                                 |
| deadline  | `uint256` | Signature expiration timestamp                                                    |

::: code-group

```ts [@farcaster/hub-web]
import { ViemWalletEip712Signer } from '@farcaster/hub-web';
import { walletClient, account } from './clients.ts';
import { readNonce, getDeadline } from './helpers.ts';

const nonce = await readNonce();
const deadline = getDeadline();

const eip712Signer = new ViemWalletEip712Signer(walletClient);
const signature = await eip712signer.signRegister({
  to: account,
  recovery: '0x00000000FcB080a4D6c39a9354dA9EB9bC104cd7',
  nonce,
  deadline,
});
```

```ts [Viem]
import { ID_GATEWAY_EIP_712_TYPES } from '@farcaster/hub-web';
import { walletClient, account } from './clients.ts';
import { readNonce, getDeadline } from './helpers.ts';

const nonce = await readNonce();
const deadline = getDeadline();

const signature = await walletClient.signTypedData({
  account,
  ...ID_GATEWAY_EIP_712_TYPES,
  primaryType: 'Register',
  message: {
    to: account,
    recovery: '0x00000000FcB080a4D6c39a9354dA9EB9bC104cd7',
    nonce,
    deadline,
  },
});
```

```ts [helpers.ts]
import { ID_GATEWAY_ADDRESS, idGatewayABI } from '@farcaster/hub-web';
import { publicClient, account } from './clients.ts';

export const getDeadline = () => {
  const now = Math.floor(Date.now() / 1000);
  const oneHour = 60 * 60;
  return now + oneHour;
};

export const readNonce = async () => {
  return await publicClient.readContract({
    address: ID_GATEWAY_ADDRESS,
    abi: idGatewayABI,
    functionName: 'nonces',
    args: [account],
  });
};
```

<<< @/examples/contracts/clients.ts

:::

## Errors

| Error            | Selector   | Description                                                                                                  |
| ---------------- | ---------- | ------------------------------------------------------------------------------------------------------------ |
| InvalidSignature | `8baa579f` | The provided signature is invalid. It may be incorrectly formatted, or signed by the wrong address.          |
| SignatureExpired | `0819bdcd` | The provided signature has expired. Collect a new signature from the signer with a later deadline timestamp. |

## Source

[`IdGateway.sol`](https://github.com/farcasterxyz/contracts/blob/1aceebe916de446f69b98ba1745a42f071785730/src/IdGateway.sol)
