# Polkadot Token Standard

- **OSWG Number:** 1
- **Champion:** [@pserg1, @Veniamin]
- **Status:** Draft
- **Created:** 2020-12-21
- **Contributors**
    - Bryan, bryan@acala.network, Acala
    - Jakub, (N/A), HydraDX  
    - Nate, (N/A), Moonbeam    

## Summary

To falicitate creation of ecosystem-wide standard for asset/token storage, generation and management. 

## Motivation

Currently there are many polkadot-based projects using either standard balances module or modifications thereof intended to support variations in business logic. We want to unify the way projects handle assets/tokens within the ecosystem and address following issues 

* Integration: Provide unified interfaces for wallets/exchanges to integrate different polka-based projects.
* Unified Assets: Allow projects / substrates to create tokens/assets in a unified way across projects without runtime updates.
* User-generated Assets: Allow users to freely create assets/tokens (ERC-20 style) on different substrates in a unified way.

## Setup

The definition how the working group collectivly works together in the sense of sessions, meetings, related documents or anything that is relevant to the working group.
* [OSWG Channel on Element](https://app.element.io/app)

## Findings

The table below shows the information on projects who shared their specifics/requirements in relation to balances/transfers and general asset handling.

|Project|Standard Balances?|Requirements|Balance Type|Get Balance|Make Transfer|
|---|---|---|---|---|---|
|Acala|No|standard balances for native assets, [orml tokens](https://github.com/open-web3-stack/open-runtime-module-library/blob/master/tokens/src/lib.rs) for other assets, orml-currencies provides a unified API to access orml-tokens & pallet-balances|u128|tokens.accounts(AccountId, CurrencyId): AccountData, system.account(AccountId): accountInfo - for ACA|currencies.transfer(dest, currency_id, amount), balances.transfer(dest.value) - for ACA|
|Centrifuge|Yes|Support multiple assets/tokens; support issue / burn|u128|system.account(AccountId): AccountInfo|balances.transfer(dest.value)|
|Equilibrium|No|support multiple assets/tokens; support negative balances; mint/burn stablecoins; "free" balance based on LTV|u64|balances.account(AccountId, Currency): SignedBalance|balances.transfer(currency, to, value)|
|HydraDX|No|standardized token balance / type (i.e. u128); create new tokens; standardized price type; ability to stake with non-native tokens (freeze, unfreeze)|u128|N/A|N/A|
|Moonbeam|Yes|accounts are created in ethereum keypair format (H160 instead of H256) directly in polkadot.js there is separate transact extrinsic inside the ethereum pallet|u128|system.account(AccountId): AccountInfo|balances.transfer(dest.value)|
|Plasm|Yes|no specific requirements, looking for XCM|u128|balances.account(AccountId): AccountData|balances.transfer(dest.value)|
|Snowfork|No|[assets](https://polkaeth-rustdocs.netlify.app/artemis_asset/index.html); Support for multiple fungible currencies/assets; the ability to identify a currency by a 20-byte identifier (for supporting bridged ERC20 tokens); The ability to store account balances using 256-bit precision|u256|N/A|N/A|

## Proposition

To tackle the Integration, Unified assets and User-generated assets issues outlined in the previous section, we propose the following:


1. Each project’s substrate storage should have its own asset registry. We propose to have it as a separate module so the proxy and rpc may be added easily there.
2. Assets/tokens should have a unique identifier which will be an asset symbol initially. With the introduction of XCMP the identifier will change to symbol + multilocation or a combination of multiasset/multilocation.
3. RPC methods:

* getAsset(symbol) - returns asset/token information (symbol, name, decimals, token supply, multilocation). 
* getAssets - return the list of all assets from the registry. 
* getBalance(accountId) - returns array of balances (symbol, balance) for given accountID

4. Extrinsics:
 
- registerAsset (symbol, name, decimals, token supply, multilocation) - registers user-specified asset/token on a substrate of choice. There may be projects with a static asset list, where the addition of a new asset requires a runtime update, and there may be projects with a dynamic asset list where introduction of the registerAsset method looks feasible. 

- transfer (symbol, to, quantity) - transfer an asset. For backward compatibility with existing polkadot-based projects who use standard balances pallet the symbol parameter should be omitted in the transfer method. This way polkadot will avoid updating the runtime while still supporting the standard by adding corresponding RPC. 


5. Separate Proxy pallet for those projects who have more than 1 assets module (e.g. Acala, Moonbeam). Which will handle the transition logic between “generic” assets and, for example, ERC-20 tokens within the ethereum pallet. Or add a transfer method  to existing balances pallet with such logic/API.

