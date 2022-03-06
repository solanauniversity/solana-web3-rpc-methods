# Solana Web3 important RPC methods

In this quest we will be working on different web3 methods which are important to build Solana Decentralized Application. We will learn different ways to fetch the data from the Solana blockchain using Web3 methods.

## Subquest: What methods will be covered in this quest ?

1. How to fetch account information based on the public key ?
2. How to fetch the main account balance ?
3. How to fetch the SPL token balance?
4. How to create the associate account against a mint key?
5. How to identify the Custom Token account associated with the mint key ?

## Subquest: What will you get after going through this quest ?

1. You will know how to fetch account information based on the public key.
2. You will know how to fetch the main account balance.
3. You will know how to fetch the SPL token balance.
4. You will know to identify if a Custom Token account exists or not in the blockchain.
5. You will know to identify the Custom Token account associated with the mint key.


### Prerequisites to work on this quest

1. Knowledge of crypto wallet (Phantom wallet specifically).
2. A basic react app with wallet connectivity feature.
3. Solana cluster can be : local, devnet or testnet.
4. For better understanding, please go through our first quest on **creating a wallet connection** with react app.

## Subquest: Useful RPC methods via Web3

### How to fetch account information based on the public key ?
**getAccountInfo** : This method helps to find out the account information for the public key passed to it.
```javascript
const accountDetails = await connection.getAccountInfo(publicKey)
```
In the above code, publicKey will be base-58 encoded string.
getAccountInfo is being called on the connection instance from the Solana cluster.
The above function can return 
1. null : If the account do not exist yet
2. {lamports, owner, data, executable, rentEpoch} : 
> lamports: number of lamports assigned to this account, as a u64 <br>
> owner: base-58 encoded Pubkey of the program this account has been assigned to <br>
> data: data associated with the account, either as encoded binary data or JSON format, depending on encoding parameter<br>
> executable: boolean indicating if the account contains a program (and is strictly read-only) <br>
> rentEpoch: the epoch at which this account will next owe rent, as u64

### How to fetch the main account balance ?
**getBalance** : This method helps to find out the account balance in lamports for the public key passed to it.
```javascript
import { Connection, PublicKey } from "@solana/web3.js";
const NETWORK = web3.clusterApiUrl("devnet");
const connection = new Connection(NETWORK);

let lamports = await connection.getBalance(payer.publicKey);
```
In the above code, publicKey will be base-58 encoded string.
getBalance is being called on the connection instance from the Solana cluster.
> lamports: The returned value will be in lamports

### How to fetch the SPL token balance ?
To fetch the balance of custom SPL token is a 2 step process.
In first step, the correct token public key of the associated account mapped with mint key gets fetched.
In the second step, the balance of the token public key gets fetched.

1. Fetch the correct associated account 
```javascript
import { Connection, PublicKey } from "@solana/web3.js";
const NETWORK = web3.clusterApiUrl("devnet");
const connection = new Connection(NETWORK);

const associatedAccounts = await connection.getTokenAccountsByOwner(
      providerPublicKey,
      { mint: new PublicKey(ANY_MINT_KEY) }
    );
```
In the above code, 
**getTokenAccountsByOwner** 
* accepts two parameters
>providerPublicKey: The public key of the connected wallet or it can be any wallet key against which the associated account needs to be fetched.
>ANY_MINT_KEY: It can be any mint key against which the custom token was created.

* returns
It returns the array of associated accounts mapped to a mint key, because an owner can hold multiple associated accounts to the same mint key.
Hence while fetching the total balance of the associated account, you always need to loop through the result

2. Fetch the balance of associated account 
```javascript
import { Connection, PublicKey } from "@solana/web3.js";
const NETWORK = web3.clusterApiUrl("devnet");
const connection = new Connection(NETWORK);

    let associatedAccountBalanceInLamports = 0;

    for (let i = 0; i < associatedAccounts.value.length; i++) {
        const associatedAccountPubKey = associatedAccounts.value[i].pubkey;
        const associatedAccountBalanceResult = await connection.getTokenAccountBalance(
          associatedAccountPubKey
        );
        associatedAccountBalanceInLamports += associatedAccountBalanceResult.value.amount?  parseInt(associatedAccountBalanceResult.value.amount): 0;
    }
```
In the above code, 
**getTokenAccountBalance** 
* accepts one parameter
It will accept the associated account's public key which we fetched from the first step.

* returns
It will return the balance in lamports in the form of string. Hence we need to parse it before calculating the total balance against the associated account.

### How to identify the Custom Token account associated with the mint key ?
**findAssociatedTokenAddress**: This method helps to find out the token address associated with a mint key and wallet key or not.<br>
If no associate is found then it creates a public key which will be associated with the mint public key and wallet public key.<br>
The code is added with the below topic for better understanding.

### How to create the associate account against a mint key?

**findAssociatedTokenAddress** : 
This is one of the most useful methods in the Solana ecosystem as to transfer any custom token or NFT, the user needs to have the associate account of the same mint key for which the transfer has to be made.
```javascript
import { Connection, PublicKey } from "@solana/web3.js";
const NETWORK = web3.clusterApiUrl("devnet");
const connection = new Connection(NETWORK);

const TOKEN_PROGRAM_ID = new PublicKey("TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA")
const SPL_ASSOCIATED_TOKEN_ACCOUNT_PROGRAM_ID = new PublicKey("ATokenGPvbdGVxr1b2hvZbsiqW5xWH25efTNsLJA8knL")
const associatedTokenPubkey = await findAssociatedTokenAddress(ownerPubkey, mintPubkey)

async function findAssociatedTokenAddress(
  walletAddress,
  tokenMintAddress
) {
    return (await PublicKey.findProgramAddress(
        [
            walletAddress.toBuffer(),
            TOKEN_PROGRAM_ID.toBuffer(),
            tokenMintAddress.toBuffer(),
        ],
        SPL_ASSOCIATED_TOKEN_ACCOUNT_PROGRAM_ID
    ))[0];
}

```
In the above code we have created a wrapper around findProgramAddress which will return either the existing associated account or it will create a account associated with the mint key and the wallet key passed.<br>
**toBuffer()** : This is a utility function which convert the string into stream of bytes which can be transferred over the solana network.
**TOKEN_PROGRAM_ID** : It is the program id of the SPL program (smart contract) deployed on the solana blockchain, which handles all the token creation logic.<br>
**SPL_ASSOCIATED_TOKEN_ACCOUNT_PROGRAM_ID** : It is the program id of the associated SPL program (smart contract) deployed on the solana blockchain, which handles all the associated public key creation logic.

## Subquest : What's next?
### What can you build taking this quest as a base ?
- Distribute your own token to the user’s solana account via Airdrop.
- You can create the associate account for the user.
- You can check the balance of the user's account and take the relevant action.
- You can check for the specific NFT presents in the user's wallet based on the mint key.
