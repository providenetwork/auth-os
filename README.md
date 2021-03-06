![auth_os](https://uploads-ssl.webflow.com/59fdc220cd54e70001a36846/5ac504590012b0bc9d789ab8_auth_os%20Logo%20Blue.png)

### Description:

**auth_os** is an on-chain framework for developing, managing, and interacting with applications on the EVM securely. Applications are modular, upgradable, and extensible by design: using abstract storage of application data, application logic libraries define standard interfaces through which storage is accessed. The entire system is designed around a philosophy of building the 'most general, most abstract' approach to application development and use - allowing for unparalleled flexibility and interoperability of live applications.

This repository contains the auth_os kernel: a script execution proxy, an abstract storage contract, and a script registry application. These three components make up the backbone of the entire auth_os system.

### Explanation - Contracts and Functions:

##### General Structure:

Applications consist of 3 parts - a script execution proxy, an abstract storage contract, and a set of logic libraries. Logic libraries are accessed through abstract storage by the script execution proxy, and contain all of the logic necesary for an application to function. The abstract storage contract serves as a hub for these logic libraries, routing execution requests from an application instance's script execution contract, in the context of a unique execution id. Whereas most smart contract applications use addresses to distinguish one application instance from another, auth_os applications each have a per-storage-contract unique execution id, which serves as its unique identifer. The reason for this is simple - because of unique capabilities of abstract storage, *multiple application instances may share a single storage contract.* In fact, theoretically every application running on auth_os could share the same storage contract. This is made possible through the execution ids; when an application stores information, it is stored to a location (provided by the application), hashed with the instance's execution id (safely enforced by the storage contract). Because the liklihood of a hash collision is so low, it is possible for every application on the network to share a single storage contract.

This unique structure also means that, often, deploying an instance of an application does not require the deployment of a contract (i.e. use of the `create` opcode). Applications are essentially just a set of logic libraries - so once they are deployed (and built to specification), anyone can create an instance of an existing application by simply generating a new execution id for any given set of existing libraries. For example - if one user creates a token application with logic libraries at addresses A, B, and C, every user can take addresses A, B, and C, and create a new application instance in the same storage contract - generating a new, unique execution id in the process. In place of a constructor, applications have unique, single-use `init()` functions, which allow the caller to set up initial variables just as they would in a constructor. These instances will not conflict in any way - meaning that instead of deploying the same, tired ERC20 implementation to several addresses, a single ERC20 implementation can be deployed once and re-used indefinitely without contributing to blockchain bloat.

The paradigm this creates is one where applications are used and re-used by many people, ensuring that the most prevalent applications are looked at by several parties, as opposed to a single body. In time, the hope is that a system like this will create an environment where vulnerabilities and bugs are caught quickly - and that in the event of a live bug, a large, surrounding community exists, capable of coming to a consensus about the best possible solution.

##### Script Registry:

Application definitions (implementing functions, addresses, descriptions, and other metadata) are typically stored in the script registry, which is itself an application on auth_os. The script registry implements the functions required for a developer to register applications and versions, add implementing details, and finalize releases. Upgrades are meant to be pulled, not pushed - each application is initialized with an updater address, to which various functionality may be assigned. Whether applications are upgraqded in an entirely centralized manner, with one party in control, or upgraded as a part of a DAO-type organization is entirely up to the person who initializes the instance. Over time, there will likely be several options for upgradability and extensibility available at deploy-time.

##### Application Lifecycle - Registration, Implementation, and Release:

(Reference: /contracts/registry/functions/)

When using a script registry contract to store or read information on already-deployed applications, there are a few steps to take from registration to release. An application is initially registered by a 'provider.' The provider is simply an id generated from the address of the person who registered the application. Applications are registered under the provider, and only the provider may add versions, implementations, etc. The InitRegistry contract contains several useful getters for reading application and version information.

An application is first registered in `AppConsole.sol`, using `registerApp`. The provider simply defines an application name, storage address, and description, which is placed in registry storage, or, the calling contract - these functions are executed through an abstract storage contract, or (more often) by proxy through `ScriptExec`.

Following application registration, a provider may register a named version under that application, using `VersionConsole.sol`. Versions are placed in a list within application storage. Registering a version simply sets the version's name, version storage address (optional - leave empty to use default application storage address), and a version's description. Versions define implementations - using `ImplementationConsole.sol`, a provider may add implemented functions, addresses where those functions are implemented, and descriptions of functions. A provider may add as much or as little implementation detail as they like, until the version is 'finalized.' Finalizing a version is done through `VersionConsole.finalizeVersion`, and locks version information storage. This version is now considered 'stable,' and cannot be altered further.

##### Application Lifecycle - Initialization and Usage:

Applications are initialized through the `ScriptExec` contract - by specifying the registry storage address to pull app implementation and initialization details from, anyone can deploy an application by simply specifying the registry execution id under which applications are registered. `ScriptExec.sol` currently handles many aspects of initalization, use, upgrading, and migration - but will be opened to more general-use functionality in the future.

Upon initializing an application, the specified storage address returns the app's unique execution id, through which application storage requests are made. The `ScriptExec` contract, application storage contract, and execution id form a key together: ensuring applications are only input data which is permitted by the `ScriptExec` contract, and that applications are only governed by logic addresses which are permitted by the storage contract.

Upon initialization of the application, the application can be used by calling the `ScriptExec.exec` function and targeting the desired address to forward the request to.

##### Testing:

Several of these contracts have been deployed to Ropsten, to allow for open testing. Below is a list of the verified contracts being used:

Core:
1. RegistryStorage: https://ropsten.etherscan.io/address/0xb1d914e7c55f16c2bacac11af6b3e011aee38caf#code
2. ScriptExec: https://ropsten.etherscan.io/address/0x41e37c2d17cf8aa00ae22827ecd10dd017ae52b4#code
3. InitRegistry: https://ropsten.etherscan.io/address/0xbc25d8c026ef18ca00bfa328a2c03c54af3c3e95#code
4. AppConsole: https://ropsten.etherscan.io/address/0x9ca096ea086e6c27d3b69d1f8ba4278502815fa1#code
5. VersionConsole: https://ropsten.etherscan.io/address/0xa609f05557d458727c90603adad436041915a0ca#code
6. ImplementationConsole: https://ropsten.etherscan.io/address/0x80a136184e1d65b0d975cc874a48e746eae041b7#code

MintedCappedCrowdsale: (Referencing auth-os/applications/TokenWizard/MintedCappedCrowdsale)
1. InitCrowdsale: https://ropsten.etherscan.io/address/0x4feec2b6944e510eef031d96330cb70f9051c440#code
2. CrowdsaleConsole: https://ropsten.etherscan.io/address/0xf8d0a48ce98bbee75e7568b3ada3efa1b6324c9c#code
3. TokenConsole: https://ropsten.etherscan.io/address/0x1271b0587e1216579f4fd0fc088ff5cdb4f904ef#code
4. CrowdsaleBuyTokens: https://ropsten.etherscan.io/address/0xec424a2a841a6c77d270fd91bd885ccb06392be5#code
5. TokenTransfer: https://ropsten.etherscan.io/address/0x415f6e76616a4d7c24d59f1c70687a381d48d022#code
6. TokenTransferFrom: https://ropsten.etherscan.io/address/0xc96f0b7508a35bea00a9edfc6ecf11ee31693a67#code
7. TokenApprove: https://ropsten.etherscan.io/address/0xbb3d0d9c0630e7d7b81d30758b2d940ede81ab93#code

DutchCrowdsale: (Referencing auth-os/applications/TokenWizard/DutchCrowdsale)
1. InitCrowdsale: https://ropsten.etherscan.io/address/0xbe9c4888a51761f6c5a7d3803106edab7c96196e#code
2. CrowdsaleConsole: https://ropsten.etherscan.io/address/0xb469710858cf3330d8d4ce8886bda2db24c46e20#code
3. TokenConsole: https://ropsten.etherscan.io/address/0xdd434cc4ae76beedc71c232fb399cacf262eed63#code
4. CrowdsaleBuyTokens: https://ropsten.etherscan.io/address/0x8519215210f8ac2c034eb6afc6a6766db4f97a6a#code
5. TokenTransfer: https://ropsten.etherscan.io/address/0x2cb3897e1b7eff0dec33365d0f5c864004bec326#code
6. TokenTransferFrom: https://ropsten.etherscan.io/address/0xefe7acdd5e28a9fc35854bdc7c91a2d2c419f5b0#code
7. TokenApprove: https://ropsten.etherscan.io/address/0x95971b6ff4a5a076c6b74a2b53ec8a3db96f91d5#code

The 'default' variables in ScriptExec.sol can be examined for information on the registry's execution id, as well as the storage address of the registry. 

###### Deployment -> Use of Script Registry and Crowdsale contracts:

A. Initial Deployment (storage contract, registry contracts, application contracts):
  1. We first deploy all contracts, except the Script Exec contract. The Script Exec contract is built in such a way that it depends on already-deployed contracts, as well as the Script Registry contracts being initialized and in use.
  2. First round of deployment - `RegistryStorage`, `InitRegistry`, `AppConsole`, `VersionConsole`, `ImplementationConsole`, `InitCrowdsale`, `CrowdsaleConsole`, `TokenConsole`, `CrowdsaleBuyTokens`, `TokenTransfer`, `TokenTransferFrom`, `TokenApprove`
  
B. Initialization of Script Registry and registration/implementation of crowdsale contracts:
  1. We now need to initialize the script registry contracts, so that the `ScriptExec` contract is able to read information about the crowdsale we want to deploy. This is called from your personal address, as ScriptExec currently does not support initializing the registry contracts (it tries to look up initialization information on the registry contracts, but the registry contracts are not yet initialized themselves!)
  2. Call: `RegistryStorage.initAndFinalize`
    - I set myself as the updater, so that I'm able to swap out faulty contracts if need be
    - is_payable should be false for the registry contracts
    - init is the address for `InitRegistry`, and `init_calldata` should be the calldata for `InitRegistry.init` (only 4 bytes - no params)
    - allowed addresses - `AppConsole`, `VersionConsole`, `ImplementationConsole`
  3. RegistryStorage should return an exec id - this will be the `default_registry_exec_id` used with the `ScriptExec` contract. It is also the exec id used by you to register application information in the script registry app. You will directly be using `RegistryStorage.exec` for this.
  4. Time to register the crowdsale application - get the appropriate calldata for `AppConsole.registerApp`, and pass that through `RegistryStorage.exec` (of course, using the `AppConsole` address as the target). This sets up an unimplemented (but named) application in the script registry app. You can view information on it through these `InitRegsitry` functions:
    - `getProviderInfo`, `getProviderInfoFromAddress`, `getAppInfo`, `getAppVersions` (should be 0)
    - In the live contracts, the crowdsale app is called "MintedCappedCrowdsale"
  5. Next, we want to create the first version of our application - get the calldata for `VersionConsole.registerVersion`, and pass that through `RegistryStorage.exec` (using `VersionConsole` as the target). This sets up the first version of our app, which I called 'v1.0'. Information on this version can be viewed through these InitRegistry functions:
    - `getVersionInfo`, `getAppVersions`
  6. Now we need to specify where and which logic contracts belong to the app. Get calldata for `ImplementationConsole.addFunctions`, and pass it through the `ScriptExec` contract. For each function, provide the address where it can be called. It is also possible to add descriptions for each function, through `ImplementationConsole.describeFunction`. Use these functions for information:
    - `getVersionImplementation`, `getImplementationInfo`
    - The live crowdsale application was implemented in 3 'addFunction' batches - first, the token and purchase contracts, then the `TokenConsole` functions and address, and finally the `CrowdsaleConsole` functions and address.
  7. Finally - we want to finalize our version, marking it as the latest, 'stable' version. This finalization allows the `ScriptExec` contract to view the app we just registered. Call `VersionConsole.finalizeVersion` through the storage exec function.
    - View init information: `getVersionInitInfo`, `getAppLatestInfo`
    
C. Deployment of `ScriptExec`, and initialization of crowdsale app:
  1. We now have the registry storage address, an exec id we've been using with the registry, and the id of the the provider that registered the apps we want to initialize (the hash of the address that called all the `RegistryStorage.exec` functions). Put the appropriate informatino in the constructor, and deploy
  2. To initialize our application - get the calldata you want for the crowdsale, from `InitCrowdsale.init`. Pass that, and the app's name (MintedCappedCrowdsale), as well as 'true' for is_payable, into `ScriptExec.initAppInstance`. You should get back an exec id - this it the crowdsale's exec id, to be used only through the `ScriptExec` contract
  
D. Initialization of MintedCappedCrowdsale:
  1. MintedCappedCrowdsale's `InitCrowdsale.init` function was already called during the previous step. However, we can now do a few things to finish the job and get an up-and-running crowdsale app. First, we need to initialize the crowdsale token. Get the calldata for `CrowdsaleConsole.initCrowdsaleToken`, and pass that through `ScriptExec.exec`, with `CrowdsaleConsole` as the target address.
    - You can view info on the created token in `InitCrowdsale`: `getCrowdsaleInfo`, `getCrowdsaleStartTime`, `getCurrentTierInfo`, `getCrowdsaleTier`, `getTokenInfo`
  2. We can now call `CrowdsaleConsole.initializeCrowdsale`, or we can do any of the following:
    - Add whitelisted tiers and users (`CrowdsaleConsole.createCrowdsaleTiers` and `CrowdsaleConsole.whitelistMulti`)
    - Update tier duration (must be before tier begins) (`CrowdsaleConsole.updateTierDuration`)
    - Set, update, or delete reserved tokens: (`TokenConsole.updateMultipleReservedTokens`, `removeReservedTokens`)
  3. Finally - call `CrowdsaleConsole.initializeCrowdsale`. This will open the app for purchasing (once the start time is reached)
  
E. Buying tokens:
  1. Buying tokens is done through the `CrowdsaleBuyTokens.buy` function. It takes simply the `context` array, which should be the crowdsale app's exec id, the sender's address, and the amount of wei sent. Passing this into `ScriptExec.exec` and sending the correct amount of wei should net the sender tokens (if there are some to be sold)
  
F. Finalization of crowdsale and ditribution of reserved tokens:
  1. The owner can finalize the crowdsale at any point - by calling `CrowdsaleConsole.finalizeCrowdsale` (through `ScriptExec.exec`). Finalization unlocks tokens for transfer, and allows reserved tokens to be distributed.
  2. Reserved tokens are distributed through `TokenConsole.distributeReservedTokens`. The function takes an 'amount' of destinations you want to cycle through and distribute to - so that batching is possible in the event of several destinations.

That's it! Typically, of course, only C.2. and onwards will need to be done - everything else is already there.
