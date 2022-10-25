# NFT-Smart-Contracts-Library

**Library of smart contracts implementing various use-cases of Non-Fungible Tokens (NFT).

Non-Tungible Tokens smart contracts are simply decentralized registries that records ownership of non-fungible assets with specific business rules.

Basically, these registries record token references (named `tokenId`) that are assigned to owners' accounts addresses. Owners could either be an `Externally Owned Account` (EOA) or a `Contract Account` (CA).

This enable the implementation of various use-cases.

This library highlight NFT smart contracts implementation through the following use-cases:

Smart Contract | Description | Use-Case | Building blocks | status
------ | ------ | ------ | ------ | ------
Extensible Base NFTs | NFTs mintable with new metadata through lazy minting | Certificate of Authenticity, Intellectual properties | Token URI, Lazy minting | ongoing |
Limited-edition Base NFTs | NFTs with pre-defined metadata and mintable through lazy minting | Collectibes | Base URI, Lazy minting | ongoing |
Delayed reveal NFTs | NFTs revealed at some point in time | Collectibes | Base URI, Lazy minting | ongoing |
Whitelisted NFTs | NFTs mintable only by the whitelisted address | Collectibes | TBC | ongoing |
Payable NFTs | NFTs mintable only against payment | Collectibes | TBC | ongoing |
... | ... | ... | ... | ... |

## NFT use-cases

Any non-fungible assets requiring keeping track of ownership can leverage decentralized ledger through Non-Fungible Tokens smart contracts in order to benefit from trustless, autonomous, secure, transparent and interoperable systems.

These assets can either be tradable or non-transferable (i.e. `soulbonded`).

**Tradable use-cases kind:

Use-Case | Description
------ | ------
Collectibles | Collection of collectibles
Virtual items | Items existing in a virtual world
Certificates of Authenticity | Certificate for valuable goods
Intellectual properties | Rights ownership
Tickets | TBC
Memberships | TBC
Domain names | TBC
Real estates | TBC
... | ...


**Soulbond use-cases kind:

Use-Case | Description
------ | ------
Digital identity | -
Proof of Attendance | -
Achievements | -
Reputation | -
Copyrights | -
... | -

Use-cases are realized through NFTs smart contracts that have a base structure, implementing standard interfaces (e.g. EIP-721, EIP-1155), with on top of it additional feature such as whitelisting, royalty management, delayed reveal or any other that may be needed.

## NFT implementations

**The base structure of a smart contract is made of:

Name | Description
------ | ------
[Data structure] | Structure for unique tokens (EIP-721) vs structure for tokens with several copies (EIP-1155)
[Base Functions] | NFT functions vs Multi-NFT functions (_mint, _burn, transfer, approve...)
[Metadata storage] | Global URI (baseURI) vs Individual URI (tokenURI)

**The additional features should be developped as reusable building blocks that we can plug and play according to the use-case.

Below are interesting features that could be leverage:

[Minting] | Pre-mint vs Lazy-mint
[Delayed random reveal] | Provenance hash and fair random starting index
[Whitelisting] | Merkle tree & Merkle proof
[Royalty management] | Secondary sales royalty distribution (EIP-2981)  
... | ...

### Data structure

**The data structure of the registry depends of the edition of the non-fungible asset, he can either be unique or have several editions, i.e copies.

1) If each token is unique, the data structure could be a mapping from `tokenId` to the account
This would correspond to the EIP-721
```solidity
mapping(uint256 => address) private _owners;
```

2) If each token can have several copies, the data structure could be a mapping from `tokenId` to the account balance. The owner could have 0 to many editions of a non-fungible asset.
This would correspond to the EIP-1155
```solidity
mapping(uint256 => mapping(address => uint256)) private _balances;
```
### Base Functions

Base functions enable to read and write in the registry. These are defined by standards such as EIP-721 or EIP-1155.

**ERC721 functions:
https://eips.ethereum.org/EIPS/eip-721
```solidity
function balanceOf(address owner) external view returns (uint256 balance);
function ownerOf(uint256 tokenId) external view returns (address owner);
function safeTransferFrom(address from, address to, uint256 tokenId, bytes calldata data) external; // Guarantee safe transfers with Contract Account
function transferFrom(address from, address to, uint256 tokenId) external;
function approve(address to, uint256 tokenId) external; // Gives or removes permission to an external party to transfer a specific token
function setApprovalForAll(address operator, bool _approved) external; // Gives or removes permission to an external party to transfer any token owned
function getApproved(uint256 tokenId) external view returns (address operator);
function isApprovedForAll(address owner, address operator) external view returns (bool);
```
**ERC1155 functions:
https://eips.ethereum.org/EIPS/eip-1155
```solidity
function balanceOf(address account, uint256 id) external view returns (uint256);
function balanceOfBatch(address[] calldata accounts, uint256[] calldata ids) external view returns (uint256[] memory);
function setApprovalForAll(address operator, bool approved) external;
function isApprovedForAll(address account, address operator) external view returns (bool);
function safeTransferFrom(address from, address to, uint256 id, uint256 amount, bytes calldata data) external;
```

### Metadata storage

Metadata is generally stored in external web resources accessible via URIs.

These URIs can be stored at different level in the smart contract depending whether the entire tokens of the collection are known upfront, or if 2) the tokens are known progressively.

1) If the entire tokens collection is known beforehand, metadata could be stored globally in the `baseURI`
This store an URI pointing to a collection of metadata:
```solidity
string public baseURI;
```

2) If the tokens are known afterhand, metadata can be stored individually for each token in a `tokenURI`
This store an URI pointing directly to metadata:
```solidity
mapping(uint256 => string) private _tokenURIs;
```

Reading a token metadata is done through `tokenURI(uint256 tokenId)` function. This active either the first or second use-case according if the `baseURI` state is set or not.
```solidity
bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";
```

### Minting

Minting is the process of token creation.

This process can be launch by the smart contract owner or by any user according to certain conditions.

1) Pre-mint
All NFT are minted by the contract owner or the contract itself.

```solidity
function mint(address to_, uint256 tokenId_) external onlyOwner { _mint(to_, tokenId_); }
```

2) Lay-mint
NFT are minted over time by users.
Users could mint the NFT if they meet some criteria like:
- An amount paid (e.g. 1 ETH)
- Having a voucher (e.g. Signed message with tokenId, tokenURI)
- Being Whitelisted
- ...

```solidity
function mint(address to_, uint256 tokenId_) external payable {...}
function mint(address to_, uint256 tokenId_, string calldata voucher) external {...}
function mint(address to_, uint256 tokenId_, string calldata merkleproof) external {...}
```
 
### Delayed random reveal

Some use-cases show up metadata after the minting process (e.g. Booster, Tombolas...)

The challenge for this use-case is to implement a fair randomness that would prevent all users from sniping rare tokens, whether it is external user or  issuer team.

To do so one solution is to leverage a `provenanceHash` and generate an on-chain `startingIndex`.

### Whitelisting

Some use-cases might want to restrict the mint to a list of authorized address.

To do so one solution is to leverage a `merkletree` and proove to be whitelisted with `merkleproof`.


### Royalty management

Some use-case might want to manage royalty for each secondary sales.

To do so a standard has been defined to provide the royalty information to market places (EIP-2981).