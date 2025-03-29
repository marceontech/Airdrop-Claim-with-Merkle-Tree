# Airdrop-Claim-with-Merkle-Tree
Airdrop Claim smart contract using Merkle Tree. This is part of the Advanced Foundry course training by Patrick Collins at Cyfrin Updraft.

This smart contract implements a secure airdrop system using a Merkle Tree proof mechanism and EIP712 signature verification. 
It is designed to distribute ERC‑20 tokens to eligible users—only those who can prove (via a Merkle Proof) that they are part of a pre-approved list—and ensures that each address can only claim once.

# Overview
The Merkle Airdrop contract performs the following tasks:

Verifies Claims with a Merkle Proof:
Each eligible user's data (their address and the airdrop amount) is a "leaf" in a Merkle tree. The root of this tree is stored in the contract, and when a user wants to claim their tokens, they must supply a Merkle Proof that confirms their inclusion.

Uses EIP712 for Secure Signature Verification:
To add an extra layer of security, the contract requires a valid signature that confirms the claim. It uses EIP712 standards to securely hash the claim data and then verifies that the provided signature matches the claimant’s address.

Prevents Double Claims:
Once an address has claimed its tokens, it is marked to ensure that no one can claim more than once.

# Key Components

# 1. Imported Libraries
MerkleProof (OpenZeppelin):
Provides functions to verify Merkle Tree proofs.

SafeERC20 & IERC20 (OpenZeppelin):
Enable safe interactions with ERC‑20 tokens. This ensures tokens are transferred correctly and securely.

EIP712 (OpenZeppelin):
Implements the EIP712 standard for typed data hashing and signing, which helps in verifying off-chain signatures on-chain.

ECDSA (OpenZeppelin):
Contains functions for recovering the signer’s address from a given signature, which is used for signature validation.

# 2. State Variables
`i_airdropToken` :
The ERC‑20 token that will be distributed. Declared as immutable (set only once during deployment).

`i_merkleRoot `:
The root hash of the Merkle tree that represents the eligible addresses and their token amounts. Also immutable.

`s_hasClaimed `:
A mapping that tracks whether an address has already claimed its tokens. This prevents double claims.

# 3. Struct and Constants
`AirdropClaim` Struct:
Defines the structure for a claim, containing the claimant’s address and the token amount.

`MESSAGE_TYPEHASH `:
A constant used to create the typed data hash (EIP712) for a claim message. This is computed from the string `AirdropClaim(address account,uint256 amount)`.

# 4. Core Functions
Constructor:
Initializes the contract with the Merkle root and the token address. It also sets up the EIP712 domain (with name `Merkle Airdrop` and version `1.0.0`).

`claim` Function:
This is the main function users call to receive their tokens.

Step 1: Double Claim Check: It reverts if the user has already claimed.

Step 2: Signature Verification:
Uses `_isValidSignature ` to confirm that the provided signature (split into `v`, `r`, and `s`) is valid for the given claim.

Step 3: Merkle Proof Verification:
Computes the leaf by hashing the encoded `account` and `amount` and verifies it against the stored Merkle root using OpenZeppelin's `MerkleProof.verify` function.

Step 4: Token Transfer:
Marks the claim as completed, emits an event, and transfers the tokens using `safeTransfer`.

`getMessageHash` Function:
Constructs and returns the EIP712 hash of a claim. This is the message that must be signed by the claimant.

`Internal _isValidSignature` Function:
Uses ECDSA recovery to check if the signature was produced by the claimant's address.

# 5. Events

`Claimed`:
Emitted when a user successfully claims tokens. It includes the claimant's address and the amount claimed.

`MerkleRootUpdated`:
(Not used directly in this code snippet, but available for potential future functionality to update the Merkle root.)

# How to Use the Contract

# Deployment

1) Prepare the Merkle Tree:
Generate a Merkle tree off-chain with the eligible addresses and their corresponding token amounts. Compute the Merkle root from the tree.
Example: If you have an array of `{address, amount}` pairs, use a tool or script to create the Merkle tree and extract its root.

2) Deploy the Contract:
Deploy the contract by passing the computed Merkle root and the address of the ERC‑20 token you wish to airdrop.

``` new MerkleAirdrop(merkleRoot, airdropTokenAddress); ```

# Claiming Tokens

1) Prepare Your Claim:

The user (or an off-chain service) must prepare a claim by generating a signature over the message hash obtained from `getMessageHash(account, amount)`. The signature should be created by the account owner off-chain.

2) Call the `claim` Function:

Submit a transaction calling `claim` with the following parameters:

`account`: The user’s address.

`amount`: The number of tokens to claim.

`merkleProof`: An array of `bytes32` values representing the Merkle proof.

`v, r, s`: The components of the signature. This transaction will trigger the following:

It will check that the user hasn’t already claimed.

It will validate the signature against the claim data.

It will verify that the claim is valid by checking the Merkle proof.

Upon success, it will mark the claim as completed and transfer the tokens.

# Error Handling

`MerkleAirdrop__InvalidProof`:
Reverts if the supplied Merkle proof is incorrect.

`MerkleAirdrop__AlreadyClaimed`:
Reverts if the user has already claimed their tokens.

`MerkleAirdrop__InvalidSignature`:
Reverts if the provided signature is invalid.


# How to Integrate a Frontend Integration:

Build a UI where users can:

* Connect their wallet.
* Input their claim details.
* Submit the Merkle proof and signature. The UI can call the claim function via libraries like ethers.js or web3.js.

* Off-Chain Signature Generation:
Provide instructions or tools for users to sign the claim message. This can be done via wallet integrations (e.g., MetaMask) using the EIP712 standard.
