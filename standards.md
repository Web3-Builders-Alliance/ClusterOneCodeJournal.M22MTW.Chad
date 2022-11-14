# Wait, CosmWasm has its own standards?

Looking through [cw-plus contracts](https://github.com/Web3-Builders-Alliance/cw-plus/tree/main/contracts), I see a lot of stuff like `cw20`, which is the Fungible Token equivalent.

(Aside: it's handy for Ethereum-developer onboarding that the standard numbers correspond to Ethereum's. [ERC-20](https://ethereum.org/en/developers/docs/standards/tokens/erc-20/) is Ethereum's original Fungible Token; `cw20` is the CosmWasm equivalent. I also see a `cw1155`, which I assume corresponds to Ethereum's [multi-token standard](https://ethereum.org/en/developers/docs/standards/tokens/erc-1155/).)

Anyhow, the question this raises for me is: Where are standards defined?

The other blockchains I'm familiar with have standards committees that define standards for the whole protocol/blockchain/ecosystem.

But CosmWasm is a single library for authoring smart contracts. Why are standards prefixed with `cw`?

This seems like if the React team were authoring ECMAScript 2023 and branding it "ReactScript."

Let's figure out what's going on.

Some useful reading my searches turned up:

- [CW20 Spec in CosmWasm docs](https://docs.cosmwasm.com/cw-plus/0.9.0/cw20/spec/)
- [CosmWasm v1.0 announcement](https://medium.com/cosmwasm/cosmwasm-1-0-0-finalized-fadc148f9e18)
- [cw-plus README](https://github.com/CosmWasm/cw-plus)
- [Comparison to Solidity contracts](https://docs.cosmwasm.com/docs/1.0/architecture/smart-contracts)
- [What is Cosmwasm, How it fits into Cosmos & How it's Seeing Large Scale Adoption w/ Ethan Frey!](https://www.youtube.com/watch?v=RzZpZ0QNS9U)

Ok that was a lot of reading/watching but my perhaps-somewhat-incorrect takeaway is

# Cosmos barely had smart contracts prior to CosmWasm

Like, you could make your own smart contracts, but you also had to launch your own _blockchain_ with your own _validators_. aka _App-Specific Chains._ Each app was expected to have its whole own blockchain to go with it.

(This has always been a hangup for me when thinking about Cosmos! I think the NEAR founders were right that most projects don't need a whole chain of their own.)

And in the early days (aka pre-CosmWasm), you wrote contracts directly with the CosmosSDK, which is also I guess the thing you use to configure your whole blockchain. It's like coding smart contract logic right into the EVM runtime on Ethereum, or skipping the Wasm on NEAR and just tweaking the Rust code that the validators run. No encapsulation; no security model. Because the Cosmos assumption is that you trust all the code running on your chain, since each app has its own.

So CosmWasm isn't _just_ a library for writing smart contracts! I thought of it as something much thinner than it is.

Instead, CosmWasm also contains the glue to attach itself to CosmosSDK. That is, to make it easy for people launching their own chains to add CosmWasm to their chain. 

It's for smart contract developers, but it's also for people building whole new blockchains within Cosmos.

Which seems like it must make it possible to launch a chain within Cosmos that is somewhat more like a NEAR-equivalent—a chain that makes it easy for lots of developers to just write smart contracts and upload them, rather than needing to run their own chain.

## Aside: more gripes about the Cosmos model

But what gets me about this is that _it's really nice to be on the popular chain._ You can interoperate with all the other smart contracts already deployed there, without needing to go back through some CosmosHub and then out to another chain and then wait for the two IBC relays to come the whole way back to you. Bridges are cool, but dang doesn't that sound slow? Wouldn't you rather have fast same-chain interop?

So one CosmWasm chain gets popular for simple-use-case, just-smart-contract apps, and then everyone starts deploying to it, and aw dang, victim of its own success. CryptoKittes brings down the whole network. Because probably that NEAR-like Cosmos chain isn't actually NEAR, and doesn't have the dynamic resharding that NEAR is [promising to deliver](https://near.org/blog/near-launches-nightshade-sharding-paving-the-way-for-mass-adoption/). Because Cosmos didn't expect one specific chain to get so popular; it's scaling model was this one-chain-per-app idea.

So I guess if some CosmWasm chain gets too popular, that would actually be bad!

## So back to those standards

At this point it makes sense to me that if the mission of CosmWasm isn't just "make it easier to write smart contracts on Cosmos" but is instead the much-grander "make it possible to write smart contracts on any Cosmos chain," and if CosmWasm essentially _created the missing pieces to enable untrusted smart contract creation on Cosmos_, then it also needed to create some standards for those new smart contracts.
