# Code Review: Aavegotchi GHST Staking Diamond Contract 

## Table of Contents
<ol>
    <li><a href="#introduction">Introduction</a>
        <ol>
            <li><a href="#quick-intro-to-aavegotchi---gamifidefi-staked-nfts-on-ethereum">Quick Intro To Aavegotchi</a></li>
            <li><a href="#Overview">Overview</a></li>
        </ol>
    </li>
    <li><a href="#code-structure-and-functionalities">Code Structure and Functionalities</a>
        <ol>
            <li><a href="#EIP-2535-diamonds">EIP-2535 Diamonds Implementation</a></li>
            <li><a href="#solidity-version-and-pragma">Solidity Version and Pragma</a></li>
            <li><a href="#liberaries-and-interfaces-mports">Liberaries and Interfaces Imports</a></li>
            <li><a href="#AppStorage">Diamond AppStorage</a></li>
            <li><a href="#constructor()">Constructor</a></li>
            <li><a href="#fallback()">Fallback Function</a></li>
            <li><a href="#fallback()">DiamondStorage</a></li>
        </ol>
    </li>
    <li><a href="#facets---functionality-and-logic">Facets-Functionality and Logic</a>
        <ol>
            <li><a href="#ticketsfacet.sol">TicketsFacet.sol</a></li>
            <li><a href="#stakingFacet.sol">StakingFacet.sol</a></li>
        </ol>
    </li>
    <li><a href="#conclusion">Conclusion</a></li>
    <li><a href="#contract's-author">Contract's Author</a></li>
    <li><a href="#review-by">Review By</a></li>

</ol>

## Introduction 
[Github Link](https://github.com/aavegotchi/ghst-staking/blob/master/contracts/GHSTStakingDiamond.sol)
The Aavegotchi GHST Staking Diamond contract demonstrates the implementation of staking functionalities using the diamond-2 pattern. But, a quick overview on Aavegotchi itself first.

### Quick Intro To Aavegotchi - Gamifi/DeFi-staked NFTs on Ethereum
[Aavegotchi Docs](https://docs.aavegotchi.com/overview/overview)
Aavegotchis are NFTs with on-chain SVG layers. Each Aavegotchi holds a contract address where it stores `aTokens`, which are interest-generating ERC20 tokens from the Aave protocol.

To acquire an Aavegotchi, players first need to obtain a Portal from the Aavegotchi Shop. They can then open the Portal and claim an Aavegotchi by staking the required amount of `aTokens`. The amount needed varies based on the Aavegotchi's rarity.

Portals are bought from the Aavegotchi Store using `GHST`. Each Portal contains 10 Aavegotchis with randomly generated traits, determined by a Chainlink VRF call during the opening process.

Haunts represent different "generations" of Aavegotchis. The initial Haunt allows for the minting of 10,000 Portals. The AavegotchiDAO can vote to create new Haunts and adjust Portal prices (in `GHST`) in each Haunt.

Collaterals are specific ERC20 tokens whitelisted by the protocol for staking with Aavegotchis. These tokens are typically interest-bearing `aTokens` from Aave protocol. The choice of Collateral for each Aavegotchi is randomly determined by Chainlink VRF, along with its traits.

Traits are numeric values assigned to each Aavegotchi by Chainlink VRF. They range from 0-99 initially but can be influenced by collateral `modifiers`, equipped Wearables, and Consumables. Traits follow a bell-curve distribution, with extreme values contributing to `rarityScore`.

The base rarity of an Aavegotchi is calculated by summing the rarity of its `numericTraits`, including bonuses from Wearables, Collateral `modifiers`, and Consumables.

Items in Aavegotchi include Wearables, Consumables, and Badges. Wearables provide trait bonuses when equipped, Consumables offer temporary or permanent stat boosts, and Badges act as a record of the Aavegotchi's activities.

Kinship is a dynamic value influenced by the owner's interaction with the Aavegotchi. It starts at 50 and increases over time through interactions and consumption of Kinship potions.

An Aavegotchi's level depends on its accumulated experience. Aavegotchis can gain experience by consuming certain experience-giving items, or by participating in certain events and getting rewarded by the AavegotchiDAO. Every three levels, an Aavegotchi gets a `skillPoint` which can be assigned towards increasing one of its `numericTraits`. 
<hr />
<hr/>
The provided code represents the Aavegotchi GHST Staking Diamond contract, designed for staking GHST tokens. The contract is based on the diamond-2 implementation and is primarily focused on the management of staking GHST tokens and various functionalities related to token management.

### Overview 
This repository implements `contracts/GHSTSTakingDiamond.sol`. This is a diamond that utilizes the facets found in `contracts/facets/`.

`TicketsFacet.sol` implements simple ERC1155 token functionality. The ERC1155 tokens are called 'tickets'. There are six different kinds of 'tickets'.

 `StakingFacet.sol` implements functions that enable people to stake their GHST ERC20 tokens, or to stake Uniswap pool tokens from the GHST/ETH pair contract. Staking these earns people frens or frens points which are a non-transferable points system. The frens points are calculated with the `frens` function. The `claimTickets` function enables people to claim or mint up to six different kinds of tokens.  Each different ticket kind has a different frens price which is specified in the `ticketCost` function.

 `GHSTSTakingDiamond` will be deployed as an immutable diamond, also known as a 'Single Cut Diamond'.  This means that all of the facets of the diamond will be added to it in the constructor function of the diamond. The `diamondCut` function will not be added to the diamond and so upgrades will not be possible.

 Diamonds are used to organize smart contract functionality in a modular and flexible way, and they are used for upgradeable systems and they overcome the max-contract size limitation. 

 The functionality for this repository is relatively simple being 6 ERC1155 token types and staking functions and the ability to claim tickets with frens. We chose to implement this functionality as a diamond to streamline our contract development and familiarize everyone involved with diamonds, which is the primary contract architecture for the core Aavegotchi contracts in development.

`GHSTSTakingDiamond` is deployed using the `scripts/deploy.js` script. The `DiamondCutFacet` contract is not added to the diamond to prevent adding the `diamondCut` external function.


## Code Structure and Functionalities

1. #### EIP-2535 Diamonds
- Uses the implementation of [EIP-2535 Diamonds](https://eips.ethereum.org/EIPS/eip-2535) Standard by [Nick Mudgen](https://twitter.com/mudgen). This diamond is using a "new" kind of contract storage pattern dubbed 'AppStorage'. A single struct state variable of type `AppStorage` is declared and used in `GHSTStakingDiamond` and in the `StakingFacet` and `TicketsFacet` facets. This struct state variable is the primary mechanism to share state variables for the main application functionality. This helps to avoid storage collision, and allows persistent storage. Diamond storage is used for diamond specific functionality and for contract ownership/admin functionality. This diamond uses a direct copy of the current diamond implementation from the [diamond-2](https://github.com/mudgen/diamond-2) repository.

2. #### Solidity Version and Pragma
- The contract uses Solidity version `0.7.6` and the experimental `ABIEncoderV2`. This version is relatively stable and widely used in the Ethereum community.

3. #### Liberaries and Interfaces Imports
- `GHSTStakingDiamond` contract imports two (2) liberaries, and five (5) interfaces which will be explain later.
```bash
import "./libraries/LibDiamond.sol";
import "./interfaces/IDiamondLoupe.sol";
import "./interfaces/IDiamondCut.sol";
import "./interfaces/IERC173.sol";
import "./interfaces/IERC165.sol";
import "./libraries/AppStorage.sol";
import "./interfaces/IERC1155Metadata_URI.sol";
```
4. #### AppStorage
```bash
AppStorage s;
```
- Since a diamond is a facade smart contract that delegatecalls into its facets to execute function calls. The `struct AppStorage s` imported from `./libraries/AppStorage.sol` is the state of the contract in this case. Data is stored in the contract storage of `GHSTStakingDiamond`.
- It is important to import it as the first variable here in `GHSTStakingDiamond` to make it persistent with its facets, so that they'll all write/read to/from thesame position in storage (from slot-0). The `struct AppStorage` (which will be reaveled soon) named `s` (probably to mean storage/slot) is imported into this diamond majorly because member of the struct is set at the `constructor` level of the `GHSTStakingDiamond` upon deployment.
```bash
struct AppStorage {
    mapping(address => Account) accounts;
    mapping(uint256 => Ticket) tickets;
    address ghstContract;
    address poolContract;
    string ticketsBaseUri;
    uint256 ghstStakingTokensTotalSupply;
    uint256 poolTokensRate;
    uint256 ghstUsdcRate;
    address ghstUsdcPoolToken;
    address stkGhstUsdcToken;
    bytes32 domainSeparator;
    mapping(address => uint256) metaNonces;
    address aavegotchiDiamond;
    mapping(address => bool) rateManagers;
    //new
    address ghstWethPoolToken; //token address of GHST-WETH LP
    address stkGhstWethToken; //token address of the stkGHST-WETH receipt token
    uint256 ghstWethRate; //the FRENS rate for GHST-WETH stakers
    //New for Epoch
    uint256 currentEpoch;
    mapping(address => Pool) pools;
    mapping(uint256 => Epoch) epochs;
    bool pauseTickets;
    uint256 sunsetTime;
}
```
5. #### constructor()
- `constructor(IDiamondCut.FacetCut[] memory _diamondCut, ConstructorArgs memory _args)`
- The constructor of the provided Solidity contract initializes the Diamond contract by setting the initial configuration and ownership. Let's break down its functionality step by step:

- Constructor Arguments: The constructor takes two parameters:
- `_diamondCut`: An array of FacetCut structs representing the initial facets and their function selectors to be added to the Diamond contract.
- `_args`: of type ConstructorArgs struct 
```bash
struct ConstructorArgs {
    address owner;
    address ghstContract;
    address uniV2PoolContract;
}
```
- containing additional configuration parameters required for initializing the contract.
- Parameter check: The constructor validates that the `_args` parameter is not `addresses(0)` and that essential `addresses` like `_args.owner`, `_args.ghstContract`, and `_args.uniV2PoolContract` are not set to the `addresses(0)`.

- DiamondCut Initialization: It invokes the diamondCut internal function from the LibDiamond library to initialize the contract with the provided _diamondCut. This function adds the specified facets and function selectors to the Diamond contract.

- Contract Ownership: It sets the contract owner using the setContractOwner internal function from the LibDiamond library.

- Supported Interfaces: It populates the supported interfaces in the `LibDiamond.DiamondStorage` imported from `./libraries/LibDiamond.sol` (storage instants `ds`) by setting various ERC interface IDs, such as `IERC165`, `IDiamondLoupe`, and `IERC173`.

- Additional Initialization: Other initialization steps include setting specific parameters in the contract's storage, such as `s.ghstContract`, `s.poolContract`, `s.ticketsBaseUri`, and `s.poolTokensRate = 14`. These parameters define the addresses of related contracts and default values all in the `AppStorage`.

- Event Emission: It emits an event (`PoolTokensRate`) to log the initial pool tokens rate.
And `emit TransferSingle` 6times, setting the `operator` as `msg.sender`, the `_from` as `address(0)`, the `_to` as `address(0)`, and `_id` ranging from `0-5`, and 0 `value`.
- Event structured as `event TransferSingle(address indexed _operator, address indexed _from, address indexed _to, uint256 _id, uint256 _value)`.

6. #### fallback()
- `fallback() external payable`
- This declares a `fallback function`, meaning it will be invoked when a contract receives a call that doesn't match any of its defined functions. The `external` keyword specifies that the function can be called externally, and `payable` should indicates that it can receive `Ether`. But another function, a `receive() external payable` fuction to avoid reciving `Ether` whose single line is `revert("GHSTStaking: Does not accept ether")` is used to ensure this.

```bash
LibDiamond.DiamondStorage storage ds;
bytes32 position = LibDiamond.DIAMOND_STORAGE_POSITION;
assembly {
    ds.slot := position
}
```
- Here, it accesses the storage of the Diamond contract using the `LibDiamond.DIAMOND_STORAGE_POSITION`. This is set in storage as `keccak256("diamond.standard.diamond.storage")` to ensure storage collision with `AppStorage` is impossible. It retrieves the `DiamondStorage` struct, which contains information about the facets (modules) of the Diamond contract.

`address facet = address(bytes20(ds.facets[msg.sig]))`
- Identifying the Facet for the Function. This retrieves the `address` of the facet associated with the function that was called by inspecting `msg.sig` or using it as key to find the `address`. Each function call in Ethereum includes a function signature (`msg.sig`), which uniquely identifies the function being called. Each function signature in the diamond is unique, and there can never be two or more of thesame function signature.

`require(facet != address(0), "GHSTSTaking: Function does not exist")`
- Revert if facet Not found associated with the function, it `reverts` the transaction with an `error` message.

Delegate Call to the Facet:
```bash
assembly {
    calldatacopy(0, 0, calldatasize())
    let result := delegatecall(gas(), facet, 0, calldatasize(), 0, 0)
    returndatacopy(0, 0, returndatasize())
    switch result
    case 0 {
        revert(0, returndatasize())
    }
    default {
        return(0, returndatasize())
    }
}
```
- This section performs a delegate call to the identified facet with the same arguments and data as the original call. It copies the calldata (input data of the transaction) and returns data from the original call to the current context. If the delegate call fails, it reverts with the returned data. Otherwise, it returns the returned data from the delegate call.

In summary, this fallback function dynamically routes function calls to the appropriate facet in the Diamond contract based on the function signature. If a matching facet is found, it delegates the call to that facet. If not, it reverts the transaction. This mechanism allows the Diamond contract to handle arbitrary function calls efficiently and dynamically
<hr>
## Facets - Functionality and Logic

1. #### `TicketsFacet.sol`
Let's break down this facet step by step:

**Pragma Directives and Imports**:
```solidity
pragma solidity 0.7.6;
pragma experimental ABIEncoderV2;

import "../interfaces/IERC1155.sol";
import "../interfaces/IERC1155TokenReceiver.sol";
import "../libraries/AppStorage.sol";
import "../libraries/LibDiamond.sol";
import "../libraries/LibStrings.sol";
import "../libraries/LibMeta.sol";
import "../libraries/LibEvents.sol";
```
- These lines specify the Solidity compiler version and experimental features to be used.
- Various interfaces and libraries are imported, including ERC1155 related interfaces, storage libraries, and utility libraries.

 **Interface Definition**:
```solidity
interface IERC1155Marketplace {
    // Interface methods...
}
```
- An interface `IERC1155Marketplace` is defined here which was later in the `safeTransferFrom` and `safeBatchTransferFrom` fuctions implemented within this contract.

 **Contract Definition**:
```solidity
contract TicketsFacet is IERC1155 {
    // Contract variables and functions...
}
```
- The `TicketsFacet` contract is declared, implementing the `IERC1155` interface.

 **Contract Variables and Constants**:
```solidity
AppStorage internal s;
bytes4 internal constant ERC1155_ACCEPTED = 0xf23a6e61;
bytes4 internal constant ERC1155_BATCH_ACCEPTED = 0xbc197c81;
```
- `AppStorage` struct is declared as `internal s`, representing the contract's state storage. Like I have stated earlier, it was important to state it first so it can start writing into the right location in storage (from slot 0).
- Two internal constants `ERC1155_ACCEPTED` and `ERC1155_BATCH_ACCEPTED` are defined to represent return values from `onERC1155Received` and `onERC1155BatchReceived` functions, respectively.

 **Setter Functions**:
```solidity
function setAavegotchiDiamond(address _aavegotchiDiamond) external {
    // Function body...
}

function setBaseURI(string memory _value) external {
    // Function body...
}
```
- These functions are used to set the Aavegotchi Diamond address and the base URI for tickets. An internal fuction `enforceIsContractOwner` is used an helper function of access contrrol.

 **Getter Functions**:
```solidity
function uri(uint256 _id) external view returns (string memory) {
    // Function body...
}

function totalSupplies() external view returns (uint256[] memory totalSupplies_) {
    // Function body...
}

// Other getter functions...
```
- `uri` function returns the URI for a given ticket ID.
- `totalSupplies` function returns an array of total supplies for all ticket types.
- There are also other getter functions like `totalSupply`, `balanceOfAll`, `balanceOf`, `balanceOfBatch`, and `isApprovedForAll`.

 **ERC1155 Transfer Functions**:
```solidity
function safeTransferFrom(
    address _from,
    address _to,
    uint256 _id,
    uint256 _value,
    bytes calldata _data
) external override {
    // Function body...
}

function safeBatchTransferFrom(
    address _from,
    address _to,
    uint256[] calldata _ids,
    uint256[] calldata _values,
    bytes calldata _data
) external override {
    // Function body...
}
```
- These functions handle the safe transfer of ERC1155 tokens from one address to another, either individually or in batches. This function MUST check if `_to` is a smart contract (e.g. code size > 0). If so, it MUST call `onERC1155Received` hook(s) on `_to` and act appropriately (see "Safe Transfer Rules" section of the standard).

 **Approval and Operator Functions**:
```solidity
function setApprovalForAll(address _operator, bool _approved) external override {
    // Function body...
}

function isApprovedForAll(address _owner, address _operator) external view override returns (bool) {
    // Function body...
}
```
- These functions manage approval or disapproval for third-party operators to manage tokens on behalf of token owners.

 **Ticket Migration Function**:
```solidity
function migrateTickets(TicketOwner[] calldata _ticketOwners) external {
    // Function body...
}
```
- This function uses array of `TicketOwner` struct to set it parameter(s) for migrating tickets of each user, updating balances and emitting transfer events.

2. #### `StakingFacet.sol`
Quite a lengthy one, let's try break down this facet step by step:

 **Pragma Directives**: 

   ```bash
   // SPDX-License-Identifier: MIT
   pragma solidity 0.7.6;
   pragma experimental ABIEncoderV2;
   ```

   - `SPDX-License-Identifier`: Specifies the license identifier for the contract (MIT license in this case).
   - `pragma solidity 0.7.6`: Specifies the Solidity compiler version.
   - `pragma experimental ABIEncoderV2`: Enables experimental functionality for encoding and decoding complex data types in function arguments and return values.

 **Imports**:

   ```bash
   import "../libraries/AppStorage.sol";
   import "../libraries/LibDiamond.sol";
   import "../libraries/LibERC20.sol";
   import "../interfaces/IERC20.sol";
   import "../interfaces/IERC1155TokenReceiver.sol";
   import "../libraries/LibMeta.sol";
   import "../libraries/LibEvents.sol";
   ```

   - Importing various libraries and interfaces required by the contract.

 **Interfaces**:

   ```bash
   interface IERC1155Marketplace {
       function updateBatchERC1155Listing(
           address _erc1155TokenAddress,
           uint256[] calldata _erc1155TypeIds,
           address _owner
       ) external;
   }

   interface IERC20Mintable {
       function mint(address _to, uint256 _amount) external;

       function burn(address _to, uint256 _amount) external;
   }
   ```

   - Declarations of two interfaces, `IERC1155Marketplace` and `IERC20Mintable`, which define external functions.

 **Contract Declaration**:

   ```bash
   contract StakingFacet {
   ```

   - Beginning of the `StakingFacet` contract.

 **State Variables**:

   ```bash
   AppStorage internal s;
   bytes4 internal constant ERC1155_BATCH_ACCEPTED = 0xbc197c81;
   ```

   - Declaration of state variable `s` of type `AppStorage` struct (check previous explanation on this), and a constant variable `ERC1155_BATCH_ACCEPTED`.

 **Events**:

   ```bash
   event PoolTokensRate(uint256 _newRate);
   event GhstUsdcRate(uint256 _newRate);
   event RateManagerAdded(address indexed rateManager_);
   event RateManagerRemoved(address indexed rateManager_);
   event GhstWethRate(uint256 _newRate);
   ```

   - Declaration of various events emitted by the contract.

 **Structs**:

   ```bash
   struct PoolInput {...}
   struct PoolStakedOutput {...}
   ```

   - Declaration of two structs, `PoolInput` and `PoolStakedOutput`, used to define structured data types.

 **External/Public View Functions**:

   - Functions that provide read-only access to certain data stored in the contract, such as user stakes, current epoch, etc.

   - `userEpoch(address _account)`
    - This function takes an `_account` address as input parameter.
    - It is marked as `external` which means it can be called from outside the contract.
    - It returns the current epoch of the provided `_account`.
    - It accesses the `userCurrentEpoch` variable of the `s.accounts[_account]` mapping to retrieve the user's current epoch.
    - Finally, it returns the current epoch of the user.

   - `currentEpoch()`
    - This function does not take any parameters.
    - It is marked as `external` which means it can be called from outside the contract.
    - It accesses the `currentEpoch` variable of the contract state and returns its value.

   - `getUserStake(address _account, address _poolAddress)`
    - This function takes two parameters: `_account` address and `_poolAddress`.
    - It is marked as `external` which means it can be called from outside the contract.
    - It returns the stake amount of the specified `_account` in the specified `_poolAddress`.
    - It accesses the `accountStakedTokens[_account][_poolAddress]` mapping to retrieve the stake amount for the given `_account` and `_poolAddress`.
    - Finally, it returns the stake amount.

   - `getPoolInfo(address _poolAddress, uint256 _epoch)`
    - This function takes two parameters: `_poolAddress` and `_epoch`.
    - It is marked as `external` which means it can be called from outside the contract.
    - It returns information about a pool for a given epoch.
    - It accesses the `pools[_poolAddress]` mapping to retrieve information about the pool.
    - It constructs and returns a `PoolInput` struct containing the pool address, receipt token address, epoch pool rate, pool name, and URL.

   - `poolRatesInEpoch(uint256 _epoch)`
    - This function takes one parameter: `_epoch`.
    - It is marked as `external` which means it can be called from outside the contract.
    - It returns an array of `PoolStakedOutput` structs containing information about the supported pools in the given epoch.
    - It accesses the `epochs[_epoch]` mapping to retrieve information about the epoch.
    - It iterates over the supported pools in the epoch and retrieves the pool rate, name, and URL.
    - It constructs a `PoolStakedOutput` struct for each pool and adds it to the `_rates` array.
    - Finally, it returns the array of `PoolStakedOutput` structs containing information about the supported pools in the given epoch.

   - `hasMigrated(address _account)`
    - This function takes an `_account` address as input parameter.
    - It is marked as `public` and `view`, meaning it's a publicly accessible view function that doesn't modify the contract state.
    - It returns a boolean indicating whether the `_account` has migrated or not.
    - It accesses the `hasMigrated` variable of the `_account` stored in the contract state.

   - `deprecatedFrens(address _account)`
    - This function takes an `_account` address as input parameter.
    - It is marked as `external` and `view`.
    - It returns the result of the `_deprecatedFrens` function, which calculates the amount of 'frens' for the given `_account`.

   - `frens(address _account)`
    - This function takes an `_account` address as input parameter.
    - It is marked as `public` and `view`.
    - It returns the amount of 'frens' for the given `_account`.
    - It checks if the `_account` has migrated by accessing the `hasMigrated` variable of the `_account`.
    - If the `_account` has migrated, it calls the `_epochFrens` function to calculate the 'frens' amount based on the current epoch.
    - If the `_account` has not migrated, it calls the `_deprecatedFrens` function to calculate the 'frens' amount based on the old method.

   - `bulkFrens(address[] calldata _accounts)`
    - This function takes an array of `_accounts` as input parameter.
    - It is marked as `public` and `view`.
    - It returns an array of 'frens' amounts for each `_account`.
    - It initializes a dynamic array `frens_` to store the 'frens' amounts.
    - It iterates over each `_account` in the input array and calculates the 'frens' amount for each `_account` using the `frens` function.
    - Finally, it returns the array `frens_` containing 'frens' amounts for each `_account`.

   - `stakedInEpoch(address _account, uint256 _epoch)`
    - This function takes an `_account` address and an `_epoch` number as input parameters.
    - It is marked as `public` and `view`.
    - It returns an array of `PoolStakedOutput` structs representing the staked amount for each pool in the given epoch for the specified `_account`.
    - It accesses the `epochs[_epoch]` mapping to retrieve information about the epoch.
    - It iterates over each supported pool in the epoch and retrieves the staked amount for the `_account` in each pool.
    - It constructs a `PoolStakedOutput` struct for each pool and adds it to the `_staked` array.
    - Finally, it returns the array `_staked` containing staked amounts for each pool in the given epoch for the specified `_account`.

 **Internal View Functions**:
   - Functions that perform internal check and return results without modifying the contract state.

   - `_stakedOutput(address _poolContractAddress, uint256 _epoch, uint256 _amount)`
    - This internal function takes three parameters: `_poolContractAddress`, `_epoch`, and `_amount`.
    - It is marked as `internal` and `view`, meaning it can only be accessed internally from within the contract and does not modify the contract state.
    - It returns a `PoolStakedOutput` struct.
    - It constructs and returns a `PoolStakedOutput` struct using the provided `_poolContractAddress`, retrieving the name, URL, epochPoolRate for the `_epoch`, and the `_amount` of tokens staked in the pool.

   - `_frensForEpoch(address _account, uint256 _epoch)`
    - This internal function calculates the amount of 'frens' accumulated for a given `_account` up to a specific `_epoch`.
    - It takes `_account` and `_epoch` as parameters.
    - It returns the accumulated 'frens' for the `_account` up to the specified `_epoch`.
    - It calculates the duration since the last update of 'frens' for the `_account`.
    - It iterates over the supported pools in the given `_epoch` and calculates the accumulated 'frens' based on the staked tokens, historic pool rates, and the duration since the last update.
    - It returns the total accumulated 'frens' for the `_account`.

   - `_epochFrens(address _account)`
    - This internal function calculates the total 'frens' for a given `_account` up to the current epoch.
    - It takes `_account` as a parameter.
    - It returns the total 'frens' for the `_account` up to the current epoch.
    - It retrieves the account information from the storage.
    - It initializes `frens_` with the initial 'frens' balance of the `_account`.
    - It calculates the 'frens' accumulated for the current epoch and adds it to `frens_`.
    - It iterates over the epochs behind the current epoch and calculates the 'frens' accumulated for each epoch, adding it to `frens_`.
    - It returns the total accumulated 'frens' for the `_account`.

   - `_deprecatedFrens(address _account)`
    - This internal function calculates the amount of 'frens' using the deprecated method for a given `_account`.
    - It takes `_account` as a parameter.
    - It returns the total 'frens' calculated using the deprecated method for the `_account`.
    - It retrieves the account information from the storage.
    - It calculates the time period since the last update of 'frens' for the `_account`.
    - It calculates the 'frens' generated based on the pool tokens, GHST, and GHST-WETH pool tokens using the deprecated method.
    - It returns the total 'frens' generated using the deprecated method for the `_account`.

   - `_validPool(address _poolContractAddress)`
    - This internal function checks if a given `_poolContractAddress` is a valid pool in the current epoch.
    - It takes `_poolContractAddress` as a parameter.
    - It returns a boolean indicating whether the `_poolContractAddress` is a valid pool in the current epoch.
    - It retrieves the supported pools for the current epoch from storage.
    - It iterates over the supported pools and checks if the given `_poolContractAddress` matches any of the supported pools.
    - It returns true if the `_poolContractAddress` is found in the supported pools, indicating it is a valid pool in the current epoch.

 **External/Public Write Functions**:

   - Functions that modify the contract state, such as stake into a pool, withdraw from a pool, adjust frens, etc.

   - `adjustFrens(address[] calldata _stakers, uint256[] calldata _frens)`
    - This function adjusts the 'frens' balance for multiple stakers.
    - It takes two arrays as input: `_stakers` array containing addresses of stakers and `_frens` array containing the corresponding amount of 'frens' to adjust for each staker.
    - It requires the caller to be the contract owner.
    - It iterates over the `_stakers` array and adjusts the 'frens' balance for each staker by adding the corresponding amount from the `_frens` array.

   - `adjustFrensDown(address[] calldata _stakers, uint256[] calldata _amounts)`
    - This function decreases the 'frens' balance for multiple stakers.
    - It takes two arrays as input: `_stakers` array containing addresses of stakers and `_amounts` array containing the corresponding amount of 'frens' to decrease for each staker.
    - It requires the caller to be the contract owner.
    - It iterates over the `_stakers` array and decreases the 'frens' balance for each staker by subtracting the corresponding amount from the `_amounts` array.

   - `migrateToV2(address[] memory _accounts)`
    - This function migrates accounts to a new version (V2) of the contract and updates their 'frens' balances.
    - It takes an array of account addresses `_accounts` as input.
    - It iterates over the `_accounts` array and calls the internal function `_migrateAndUpdateFrens` for each account to migrate and update their 'frens' balances.

   - `initiateEpoch(PoolInput[] calldata _pools)`
    - This function initiates a new epoch with specified pools.
    - It requires the caller to be the contract owner.
    - It can only be called for the first epoch (epoch 0).
    - It sets the begin time of epoch 0 to the current block timestamp.
    - It adds pools specified in the `_pools` array to epoch 0.
    - It emits an `EpochIncreased` event.

   - `updateReceiptToken(address _poolAddress, address _tokenAddress)`
    - This function updates the receipt token address for a specified pool.
    - It requires the caller to be the contract owner.
    - It updates the receipt token address for the specified `_poolAddress`.

   - `updateRates(uint256 _currentEpoch, PoolInput[] calldata _newPools)`
    - This function updates the rates for pools in the current epoch.
    - It requires the caller to have the `onlyRateManager` modifier.
    - It checks that the provided `_currentEpoch` matches the current epoch stored in the contract state.
    - It calls the internal function `_updateRates` to update the rates for pools in the current epoch.

   - `_updateRates(PoolInput[] calldata _newPools)`
    - This internal function updates the rates for pools in the current epoch.
    - It ends the current epoch by setting its end time to the current block timestamp.
    - It increments the current epoch counter.
    - It begins a new epoch by setting its begin time to the current block timestamp.
    - It adds pools specified in the `_newPools` array to the new epoch.
    - It emits an `EpochIncreased` event.

   - `sunsetFrens(PoolInput[] calldata _newPools)`
    - This function ends the current epoch and sets the sunset time.
    - It requires the caller to be the contract owner.
    - It calls the internal function `_updateRates` to end the current epoch and begin a new one.
    - It sets the sunset time to the current block timestamp.

   - `bumpEpoch(address _account, uint256 _epoch)`
    - This function allows bumping a user to a certain epoch.
    - It requires the user to have migrated and the specified epoch to be lower than the current epoch.
    - It updates the 'frens' balance for the specified `_account` up to the specified `_epoch` by calling the internal function `_updateFrens`.

   - `stakeIntoPool(address _poolContractAddress, uint256 _amount)`
    - This function allows a user to stake a specified amount of tokens into a pool.
    - It calls the `stakeIntoPoolForUser` function with the sender's address.

   - `stakeIntoPoolForUser(address _poolContractAddress, uint256 _amount, address _sender)`
    - This function allows a specified user to stake a specified amount of tokens into a pool.
    - It checks if the caller is authorized by verifying if they match the `_sender` address provided or the transaction origin matches the `_sender`.
    - It checks if the pool is valid in the current epoch by calling the internal function `_validPool`.
    - It checks if the sender has sufficient token balance to stake the specified amount.
    - It calls the internal function `_migrateOrUpdate` to migrate or update the user's data.
    - It credits the user's account with the specified amount of staked tokens for the specified pool.
    - If the pool is the GHST or the main pool contract, it updates the GHST staking tokens balance and emits a `Transfer` event.
    - If the pool is a custom pool, it mints the corresponding stkGHST token to the user.
    - It transfers the staked tokens from the sender to the contract.
    - It emits a `StakeInEpoch` event.

   - `withdrawFromPoolForUser(address _poolContractAddress, uint256 _amount, address _sender)`
    - This function allows a specified user to withdraw a specified amount of tokens from a pool.
    - It checks if the caller is authorized by verifying if they match the `_sender` address provided or the transaction origin matches the `_sender`.
    - It calls the internal function `_migrateOrUpdate` to migrate or update the user's data.
    - It retrieves the receipt token address associated with the pool.
    - It retrieves the staked balance of the user for the specified pool.
    - If the pool uses a receipt token (not GHST), it checks if the user has sufficient balance of the receipt token to withdraw the specified amount.
    - It checks if the user has enough staked tokens to withdraw the specified amount.
    - It reduces the user's balance of staked tokens for the specified pool.
    - If the pool is the GHST or the main pool contract, it updates the GHST staking tokens balance and emits a `Transfer` event.
    - If the pool is a custom pool, it burns the corresponding stkGHST token from the user.
    - It transfers the staked tokens from the contract to the sender.
    - It emits a `WithdrawInEpoch` event.

   - `withdrawFromPool(address _poolContractAddress, uint256 _amount)`
   - This function allows the sender to withdraw a specified amount of tokens from a pool.
   - It calls the `withdrawFromPoolForUser` function with the sender's address.

 **Internal Write Functions**:

    - Functions that perform internal state modifications, such as updating frens, migrating users, adding pools, etc.

   - `migrateOrUpdate(address _account) internal`
    - This internal function determines whether to update the account's data or migrate it.
    - It checks if the account has already migrated. If it has, it calls `_updateFrens` to update the frens balance for the current epoch. If not, it calls `_migrateAndUpdateFrens` to migrate the account and update the frens balance.

   - `addPools(PoolInput[] memory _pools) internal`
    - This internal function adds pools to the system for the current epoch.
    - It iterates through the array of `PoolInput` structures.
    - For each pool, it checks if the pool address is not the GHST contract and ensures that it has a receipt token (except for GHST).
    - It verifies that a new pool's receipt token matches the existing one for the pool if it already exists.
    - It updates the pool's name, receipt token address, epoch pool rate, and URL in the storage.
    - It adds the pool address to the list of supported pools for the current epoch.
    - It emits a `PoolAddedInEpoch` event.

   - `updateFrens(address _sender, uint256 _epoch) internal`
    - This internal function updates the frens balance for a given account and sets the user's current epoch.
    - It retrieves the account data storage reference.
    - It updates the frens balance by calling the `frens` function for the account.
    - It updates the last frens update timestamp to the current block timestamp.
    - It sets the user's current epoch to the specified epoch.

   - `migrateAndUpdateFrens(address _account) internal`
    - This internal function migrates the account data from the previous version to the current version.
    - It ensures that the account has not already migrated.
    - It retrieves the balances of GHST and various pool tokens from the previous version.
    - It sets the staked token balances for GHST and various pools in the current version based on the balances from the previous version.
    - It updates the frens balance for the account and marks the account as migrated.
    - It emits a `UserMigrated` event.

 **Modifiers**:

   - `onlyRateManager`: A modifier that restricts access to functions to only designated rate managers.

 **Ticket Functions**:

    - Functions related to the issuance, claiming, and conversion of tickets.

   - `togglePauseTickets() external`
    - This function is used to toggle the pause state for ticket claiming.
    - It ensures that only the contract owner can execute this function by calling `LibDiamond.enforceIsContractOwner()`.
    - It toggles the `pauseTickets` state by negating its current value.

   - `claimTickets(uint256[] calldata _ids, uint256[] calldata _values) external`
    - This function allows users to claim tickets by spending their frens points.
    - It checks if ticket buying is not paused.
    - It verifies that the length of `_ids` matches the length of `_values`.
    - It updates the frens balance for the user.
    - It iterates through each ticket ID and value in the input arrays.
    - For each ticket, it calculates the cost, verifies that the user has enough frens points, updates the user's frens balance, and updates the total supply of the ticket.
    - It emits a `TransferBatch` event to signal the transfer of tickets.
    - If the sender is a contract, it calls `onERC1155BatchReceived` to notify the contract about the ticket transfer.

   - `convertTickets(uint256[] calldata _ids, uint256[] calldata _values) external`
    - This function allows users to convert tickets into drop tickets.
    - It verifies that the length of `_ids` matches the length of `_values`.
    - It calculates the total cost of converting tickets into drop tickets.
    - It ensures that the conversion can only be made into drop tickets and not from drop tickets.
    - It iterates through each ticket ID and value in the input arrays.
    - For each ticket, it calculates the cost, verifies that the user has enough of that ticket, updates the user's ticket balance, and updates the total supply of the ticket.
    - It emits a `TransferBatch` event to signal the transfer of tickets.
    - It updates the user's balance and total supply for drop tickets.
    - If the sender is a contract, it calls `onERC1155BatchReceived` to notify the contract about the ticket transfer.
    - If the Aavegotchi Diamond contract is set, it updates the batch ERC1155 listing.

   - `ticketCost(uint256 _id) public pure returns (uint256 _frensCost)`
    - This function calculates the frens cost for a given ticket ID.
    - It returns the frens cost based on the ID.

 **Rate Manager Functions**:

   - Functions related to adding and removing rate managers.

   - `isRateManager(address account) public view returns (bool)`
    - This function is a view function that checks whether the given account address is a rate manager.
    - It retrieves the boolean value associated with the given account address from the `rateManagers` mapping in the storage `s` and returns it.

   - `addRateManagers(address[] calldata rateManagers_) external`
    - This function allows the contract owner to add one or more rate managers.
    - It ensures that only the contract owner can call this function by enforcing it through `LibDiamond.enforceIsContractOwner()`.
    - It iterates through the `rateManagers_` array and sets the boolean value to `true` for each address in the `rateManagers` mapping in the storage `s`.
    - It emits a `RateManagerAdded` event for each rate manager added.

   - `removeRateManagers(address[] calldata rateManagers_) external`
    - This function allows the contract owner to remove one or more rate managers.
    - It ensures that only the contract owner can call this function by enforcing it through `LibDiamond.enforceIsContractOwner()`.
    - It iterates through the `rateManagers_` array and sets the boolean value to `false` for each address in the `rateManagers` mapping in the storage `s`.
    - It emits a `RateManagerRemoved` event for each rate manager removed.

 **Deprecated Functions**:

   - Functions (Read and Write) that are marked as deprecated and no longer recommended for use.


## Conclusion

This repository implements `contracts/GHSTSTakingDiamond.sol`. This is a diamond that utilizes the facets found in `contracts/facets/`.

`TicketsFacet.sol` implements simple ERC1155 token functionality. The ERC1155 tokens are called 'tickets'. There are six different kinds of 'tickets'.

`StakingFacet.sol` implements functions that enable people to stake their GHST ERC20 tokens, or to stake Uniswap pool tokens from the GHST/ETH pair contract. Staking these earns people frens or frens points which are a non-transferable points system. The frens points are calculated with the `frens` function. The `claimTickets` function enables people to claim or mint up to six different kinds of tokens.  Each different ticket kind has a different frens price which is specified in the `ticketCost` function.


## Contract's Author

[Nick Mudgen](https://github.com/mudgen) <br>
[Coderdan](https://github.com/cinnabarhorse) <br>
[orionstardust](https://github.com/orionstardust) <br>

## Review By:

Adekunle Stephen Omorotimi
[@Okste1234](https://twitter.com/okste1234)
[Github](https://github.com/okste1234)

<br>
<hr>
<hr>
<hr>
<hr>

[Github Link](https://github.com/aavegotchi/ghst-staking/blob/master/contracts/GHSTStakingDiamond.sol)
