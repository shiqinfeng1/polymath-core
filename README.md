[![Build Status](https://travis-ci.org/PolymathNetwork/polymath-core.svg?branch=master)](https://travis-ci.org/PolymathNetwork/polymath-core)
[![Coverage Status](https://coveralls.io/repos/github/PolymathNetwork/polymath-core/badge.svg?branch=master)](https://coveralls.io/github/PolymathNetwork/polymath-core?branch=master)
[![Gitter](https://img.shields.io/badge/chat-gitter-green.svg)](https://gitter.im/PolymathNetwork/Lobby)
[![Telegram](https://img.shields.io/badge/50k+-telegram-blue.svg)](https://gitter.im/PolymathNetwork/Lobby) [![Greenkeeper badge](https://badges.greenkeeper.io/PolymathNetwork/polymath-core.svg)](https://greenkeeper.io/)

![Polymath logo](Polymath.png)

# Polymath Core

The Polymath Core smart contracts provide a system for launching regulatory-compliant securities tokens on a decentralized blockchain. This particular repository is the implementation of a system that allows for the creation of ST-20-compatible tokens. This system has a modular design that promotes a variety of pluggable components for various types of issuances, legal requirements, and offering processes.


# ST-20 Interface Overview
## Description
An ST-20 token is an Ethereum-based token implemented on top of the ERC-20 protocol that adds the ability for tokens to control transfers based on specific rules. ST-20 tokens rely on Transfer Managers to determine the ruleset the token should apply in order to allow or deny a transfer, be it between the issuer and investors, in a peer to peer exchange, or a transaction with an exchange.

## How it works
ST-20 tokens must implement a `verifyTransfer` method which will be called when attempting to execute a `transfer` or `transferFrom` method. The `verifyTransfer` method will determine whether that transaction can be completed or not. The implementation of `verifyTransfer` can take many forms, but the default approach is a whitelist controlled by the `GeneralTransferManager`.

### The ST-20 Interface

```
contract IST20 {

    // off-chain hash
    bytes32 public tokenDetails;

    //transfer, transferFrom must respect the result of verifyTransfer
    function verifyTransfer(address _from, address _to, uint256 _amount) view public returns (bool success);

    //used to create tokens
    function mint(address _investor, uint256 _amount) public returns (bool success);
}
```


# The Polymath Core Architecture
The diagram below depicts a high-level view of the various modules, registries, and contracts implemented in Polymath Core:

![Polymath Core architecture](https://github.com/PolymathNetwork/polymath-core/blob/master/docs/images/PolymathCore.png)

## Components
### SecurityToken
`SecurityToken` is an implementation of the ST-20 protocol that allows the addition of different modules to control its behavior. Different modules can be attached to `SecurityToken`:
- [TransferManager modules](contracts/modules/TransferManager): These control the logic behind transfers and how they are allowed or disallowed.
By default, the ST (Security Token) gets a `GeneralTransferManager` module attached in order to determine if transfers should be allowed based on a whitelist approach. The `GeneralTransferManager` behaves differently depending who is trying to transfer the tokens.
a) In an offering setting (investors buying tokens from the issuer) the investor's address should be present on an internal whitelist managed by the issuer within the `GeneralTransferManager`.
b) In a peer to peer transfer, restrictions apply based on real-life lockups that are enforced on-chain. For example, if a particular holder has a 1-year sale restriction for the token, the transaction will fail until that year passes.
- [Security Token Offering (STO) modules](contracts/modules/STO): A `SecurityToken` can be attached to one (and only one) STO module that will dictate the logic of how those tokens will be sold/distributed. An STO is the equivalent to the Crowdsale contracts often found present in traditional ICOs.
- [Permission Manager modules](contracts/modules/PermissionManager): These modules manage permissions on different aspects of the issuance process. The issuer can use this module to manage permissions and designate administrators on his token. For example, the issuer might give a KYC firm permissions to add investors to the whitelist.   
- [Checkpoint Modules](contracts/modules/Checkpoint): These modules allow the issuer to define checkpoints at which token balances and the total supply of a token can be consistently queried. This functionality is useful for dividend payment mechanisms and on-chain governance, both of which need to be able to determine token balances consistently as of a specified point in time.

### TickerRegistry
The ticker registry manages the sign up process to the Polymath platform. Issuers can use this contract to register a token symbol (which are unique within the Polymath network). Token Symbol registrations have an expiration period (7 days by default) in which the issuer has to complete the process of deploying their SecurityToken. If they do not complete the process in time, their ticker symbol will be made available for someone else to register.

### SecurityTokenRegistry
The security token registry keeps track of deployed STs on the Polymath Platform and uses the TickerRegistry to allow only registered symbols to be deployed.

### ModuleRegistry
Modules allow custom add-in functionality in the issuance process and beyond. The module registry keeps track of modules added by Polymath or any other users. Modules can only be attached to STs if Polymath has previously verified them. If not, the only user able to utilize a module is its owner, and they should be using it "at their own risk".

# Setting up Polymath Core

### v2.0.0 MAINNET

    ----------------------- Polymath Network Smart Contracts: -----------------------
    PolymathRegistry:                     0xdfabf3e4793cd30affb47ab6fa4cf4eef26bbc27
    SecurityTokenRegistry (Proxy):        0x240f9f86b1465bf1b8eb29bc88cbf65573dfdd97
    ModuleRegistry (Proxy):               0x4566d68ea96fc2213f2446f0dd0f482146cee96d
    FeatureRegistry:                      0xa3eacb03622bf1513880892b7270d965f693ffb5

    ETHOracle:                            0x60055e9a93aae267da5a052e95846fa9469c0e7a
    POLYOracle:                           0x52cb4616E191Ff664B0bff247469ce7b74579D1B

    STFactory:                            0x47da34f192d3fd946fd6ce7494e9eedf171a1208
    GeneralTransferManagerFactory:        0xdc95598ef2bbfdb66d02d5f3eea98ea39fbc8b26
    GeneralPermissionManagerFactory:      0xf0aa1856360277c60052d6095c5b787b01388cdd

    CappedSTOFactory:                     0x77d89663e8819023a87bfe2bc9baaa6922c0e57c
    USDTieredSTOFactory:                  0x5a3a30bddae1f857a19b1aed93b5cdb3c3da809a
    USDTieredSTOProxyFactory:             0x4965930872da851dc81275b142920de3c976de74

    CountTransferManagerFactory:          0xd9fd7e34d6e2c47a69e02131cf8554d52c3445d5
    PercentageTransferManagerFactory:     0xe6267a9c0a227d21c95b782b1bd32bb41fc3b43b
    ManualApprovalTransferManagerFactory: 0xda89fe5b254c04e2ee10d5acb24ed72f1d60ceed
    EtherDividendCheckpointFactory:       0x968c74c52f15b2de323eca8c677f6c9266bfefd6
    ERC20DividendCheckpointFactory:       0x82f9f1ab41bacb1433c79492e54bf13bccd7f9ae
    ---------------------------------------------------------------------------------
    
New SecurityTokenRegistry 0x538136ed73011a766bf0a126a27300c3a7a2e6a6
(fixed bug with getTickersByOwner())


### v2.0.0 KOVAN

New Kovan PolyTokenFaucet: 0xb347b9f5b56b431b2cf4e1d90a5995f7519ca792

    ----------------------- Polymath Network Smart Contracts: -----------------------
    PolymathRegistry:                     0x5b215a7d39ee305ad28da29bf2f0425c6c2a00b3
    SecurityTokenRegistry (Proxy):        0x91110c2f67e2881a8540417be9eadf5bc9f2f248
    ModuleRegistry (Proxy):               0xde6d19d7a68d453244227b6ccc5d8e6c2314627a
    FeatureRegistry:                      0x8967a7cfc4b455398be2356cd05cd43b7a39697e

    ETHOracle:                            0xCE5551FC9d43E9D2CC255139169FC889352405C8
    POLYOracle:                           0x461d98EF2A0c7Ac1416EF065840fF5d4C946206C

    STFactory:                            0x22f56100c6f18b656dbf1b156334206326fc672a
    GeneralTransferManagerFactory:        0x650e9507e983077d6f822472a7dcc37626d55c7f
    GeneralPermissionManagerFactory:      0xbf0bd6305b523ce055baa6dfaa9676d6b9e6090b

    CappedSTOFactory:                     0xa4a24780b93a378eb25ec4bfbf93bc8e79d7eeeb
    USDTieredSTOFactory:                  0x9106d7fbbd2996ef787913876341d0070cbdfc95
    USDTieredSTOProxyFactory:             0xb004ff6893b95dc8a19b9e09b2920a44a609bae3

    CountTransferManagerFactory:          0xc7cf0c1ddc85c18672951f9bfeb7163ecc8f0e2f
    PercentageTransferManagerFactory:     0xfea5fcb254bcb4ada0f86903ff822d6372325cb1
    ManualApprovalTransferManagerFactory: 0x8e96e7199b9ba096d666033f058ebb0050786baf
    EtherDividendCheckpointFactory:       0x18ae137fc6581e121f3d37ed85c423dbc3c9b964
    ERC20DividendCheckpointFactory:       0x8c724a1504643e02bb02b23cdd414da637872c80
    ---------------------------------------------------------------------------------
    



## Mainnet

### v1.3.0 (TORO Release)

| Contract                                                         | Address                                                                                                                       |
| ---------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| TickerRegistry:                                               | [0xc31714e6759a1ee26db1d06af1ed276340cd4233](https://etherscan.io/address/0xc31714e6759a1ee26db1d06af1ed276340cd4233)                                              |
| SecurityTokenRegistry:                                        | [0xef58491224958d978facf55d2120c55a24516b98](https://etherscan.io/address/0xef58491224958d978facf55d2120c55a24516b98)                                              |
| ModuleRegistry:                                               | [0x31d85fffd7e38bd42d2ae0409ac149e3ef0fd92c](https://etherscan.io/address/0x31d85fffd7e38bd42d2ae0409ac149e3ef0fd92c)                                              |
| Polymath Registry:                                            | [0x06595656b93ce14834f0d22b7bbda4382d5ab510](https://etherscan.io/address/0x06595656b93ce14834f0d22b7bbda4382d5ab510)                                              |
| CappedSTOFactory:                                               | [0x2aa1b133f464ac08f66c2f702581d014e4603d31](https://etherscan.io/address/0x2aa1b133f464ac08f66c2f702581d014e4603d31)                                              |
| EthDividendsCheckpointFactory:                                  | [0x0da7ed8789348ac40937cf6ae8ff521eee43816c](https://etherscan.io/address/0x0da7ed8789348ac40937cf6ae8ff521eee43816c)                                              |
| ERC20 Dividends Checkpoint Factory:                             | [0x6950096964b7adae34d5a3d1792fe73afbe9ddbc](https://etherscan.io/address/0x6950096964b7adae34d5a3d1792fe73afbe9ddbc)                                              |
| General Permission Manager Factory:                             | [0xeba0348e243f2de2f1687060f9c795ac279c66af](https://etherscan.io/address/0xeba0348e243f2de2f1687060f9c795ac279c66af)                                              |
| Count Transfer Manager Factory:                             | [0xa662a05647a8e713be1bed193c094805d20471ff](https://etherscan.io/address/0xa662a05647a8e713be1bed193c094805d20471ff)                                              |
| Percentage Transfer Manager Factory:                             | [0x3870ee581a0528d24a6216311fcfa78f95a00593](https://etherscan.io/address/0x3870ee581a0528d24a6216311fcfa78f95a00593)                                              |


## KOVAN

### v1.3.0 (TORO Release)

| Contract                                                         | Address                                                                                                                       |
| ---------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| TickerRegistry:                                               | [0xc9af1d88fe48c8a6aa8677a29a89b0a6ae78f5a8](https://kovan.etherscan.io/address/0xc9af1d88fe48c8a6aa8677a29a89b0a6ae78f5a8)                                              |
| SecurityTokenRegistry:                                        | [0xced6e4ec2ac5425743bf4edf4d4e476120b8fc72](https://kovan.etherscan.io/address/0xced6e4ec2ac5425743bf4edf4d4e476120b8fc72)                                              |
| ModuleRegistry:                                               | [0x961913dcbe2f36176bf25774337f3277796820eb](https://kovan.etherscan.io/address/0x961913dcbe2f36176bf25774337f3277796820eb)                                              |
| Polymath Registry:                                            | [0x05a6519e49e34239f78167abf293d94dae61b299](https://kovan.etherscan.io/address/0x05a6519e49e34239f78167abf293d94dae61b299)                                              |
| CappedSTOFactory:                                               | [0xde4f3cfb6b214e60c4e69e6dfc95ede3c4e3d709](https://kovan.etherscan.io/address/0xde4f3cfb6b214e60c4e69e6dfc95ede3c4e3d709)                                              |
| EthDividendsCheckpointFactory:                                  | [0x870a07d45b0f4c5653fc29a4cb0697a01e0224b1](https://kovan.etherscan.io/address/0x870a07d45b0f4c5653fc29a4cb0697a01e0224b1)                                              |
| ERC20 Dividends Checkpoint Factory:                             | [0x7e823f5df6ed1bb6cc005c692febc6aedf3b8889](https://kovan.etherscan.io/address/0x7e823f5df6ed1bb6cc005c692febc6aedf3b8889)                                              |
| General Permission Manager Factory:                             | [0x6f5fec2934a34d2e2374042cca6505f1c87ef79b](https://kovan.etherscan.io/address/0x6f5fec2934a34d2e2374042cca6505f1c87ef79b)                                              |
| Count Transfer Manager Factory:                             | [0xb540b6fa752a91c7e7834523172309e543a99a06](https://kovan.etherscan.io/address/0xb540b6fa752a91c7e7834523172309e543a99a06)                                              |
| Percentage Transfer Manager Factory:                             | [0xfe908f07e6db57aa6bbd8374e59aac86b60374b0](https://kovan.etherscan.io/address/0xfe908f07e6db57aa6bbd8374e59aac86b60374b0)                                              |



## Package version requirements for your machine:

- node v8.x.x or v9.x.x
- npm v6.x.x or newer
- Yarn v1.3 or newer
- Homebrew v1.6.7 (for macOS)
- Truffle v4.1.11 (core: 4.1.11)
- Solidity v0.4.24 (solc-js)
- Ganache CLI v6.1.3 (ganache-core: 2.1.2) or newer

## Setup

The smart contracts are written in [Solidity](https://github.com/ethereum/solidity) and tested/deployed using [Truffle](https://github.com/trufflesuite/truffle) version 4.1.0. The new version of Truffle doesn't require testrpc to be installed separately so you can just run the following:

```bash
# Install Truffle package globally:
$ npm install --global truffle

# (Only for windows) set up build tools for node-gyp by running below command in powershell:
$ npm install --global --production windows-build-tools

# Install local node dependencies:
$ yarn
```

## Testing

To test the code simply run:

```bash
# on *nix systems
$ npm run test

# on windows systems
$ npm run wintest
```


# Extending Polymath Core

1. Deploy `ModuleRegistry`. `ModuleRegistry` keeps track of all available modules that add new functionalities to
Polymath-based security tokens.

2. Deploy `GeneralTransferManagerFactory`. This module allows the use of a general `TransferManager` for newly issued security tokens. The General Transfer Manager gives STs the ability to have their transfers restricted by using an on-chain whitelist.

3. Add the `GeneralTransferManagerFactory` module to `ModuleRegistry` by calling `ModuleRegistry.registerModule()`.

4. Deploy `TickerRegistry`. This contract handles the registration of unique token symbols. Issuers first have to claim their token symbol through the `TickerRegistry`. If it's available they will be able to deploy a ST with the same symbol for a set number of days before the registration expires.

5. Deploy SecurityTokenRegistry. This contract is responsible for deploying new Security Tokens. STs should always be deployed by using the SecurityTokenRegistry.

## Deploying Security Token Offerings (Network Admin Only)

Security Token Offerings (STOs) grant STs the ability to be distributed in an initial offering. Polymath offers a few out-of-the-box STO models for issuers to select from and, as the platform evolves, 3rd party developers will be able to create their own offerings and make them available to the network.

As an example, we've included a `CappedSTO` and `CappedSTOFactory` contracts.

In order to create a new STO, developers first have to create an STO Factory contract which will be responsible for instantiating STOs as Issuers select them. Each STO Factory has an STO contract attached to it, which will be instantiated for each Security Token that wants to use that particular STO.

To make an STO available for Issuers, first, deploy the STO Factory and take note of its address. Then, call `moduleRegistry.registerModule(STO Factory address);`

Once the STO Factory has been registered to the Module Registry, issuers will be able to see it on the Polymath dApp and they will be able to add it as a module of the ST.

Note that while anyone can register an STO Factory, only those "approved" by Polymath will be enabled to be attached by the general community. An STO Factory not yet approved by Polymath may only be used by it's author.


# Code Styleguide

The polymath-core repo follows the [Solidity style guide](http://solidity.readthedocs.io/en/develop/style-guide.html).

# Links    

- [Polymath Website](https://polymath.network)
- [Ethereum Project](https://www.ethereum.org/)
- [Solidity Docs](https://solidity.readthedocs.io/en/develop/)
- [Truffle Framework](http://truffleframework.com/)
- [Ganache CLI / TestRPC](https://github.com/trufflesuite/ganache-cli)
