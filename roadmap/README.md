# Proposed Workplan: the Astral Protocols

2021

It's quite a time to be alive. Despite the challenges, we're excited for what's to come, in Web3 and in the world. 

At Astral we are creating the tools developers will need to build an ecosystem of location-based dApps and composable spatial contracts. We're quite early in our journey - we've only been prototyping, researching and writing for the past few years. We have an orienting question prompting these investigations - what if we could use an entity's physical location as a condition in a smart contract? And relatedly, how can we connect verifiable insights about physical reality with smart contract logic?

Our investigations into this question has hardened our understanding of the tools and components needed. Our goal is to share these ideas openly - gather some feedback, and see if you or anyone you know would be interested in supporting one or more of these initiatives with expertise and / or capital. Our strategy is to design a modular architecture of Web3 spatial primitives - each useful in their own right, but designed to function best together.

We have built a few Astral dApp prototypes, and architectected a number more. Through this we are conceptualizing the design space as a three layer stack. 
- Data - capture, storage and perhaps processing 
- Oracles
- Spatial Contracts

What is unique about this compared to other oracle systems is that our focus is narrowly on spatial data, that is, information that contains some spatial, or location, dimension. We could argue that _all_ data is spatial data, but here specifically we are looking at data representing physical space - geospatial data, and data adhering to other spatial reference systems.


## Data

### Origin

All data has some origin, a point of capture or creation. Very often spatial data originates at a device that detects some variation in the physical environment contacting its empirical sensor, converts this analogue signal into some digital representation, and stores, processes or transmits that binary information. 

We see this as the ultimate oracle problem - how can we trust that information is true? We have done some amateur investigation into these topics, using secure enclaves (IoTeX and Intel SGX, in Hyperaware), and thinking about zero knowledge location proofs, but we lack the knowledge to meaningfully pursue this. We understand that while advancements in positioning, navigation and timing, (along with other remote sensing technologies) are developing apace, it is still not possible to complete trust an edge reading, especially a position. This represents a significant attack vector for the systems we are building, and we would like to conduct and support further research into trusted measurements from the edge of the network. 

### GeoDIDs: Web3-native spatial data

Once these measurements are captured digitally, the information is formatted to be processed by some software. Spatial data comes in many different formats - GeoJSON, KML, shapefiles, GeoTIFFs, cryptospatial coordinates etc etc. 

Astral endeavors to design an open and agnostic protocol that will gracefully develop and evolve with the state of computing. We also demand full trustlessness, self sovereignty and independent verifiability of all protocols we design. This requires a versatile, Web3-native spatial data identification, encoding and storage scheme. 

To achieve this, with support from the Filecoin Foundation and London Blockchain Labs, we have drafted a GeoDID Method Specification and are developing prototype modules to create, read, update and deactivate these geographic decentralized identifiers and the data assets they reference. 

GeoDIDs will identify these files by wrapping the spatial data assets in DID documents, thereby providing a standard schema to grant the user control and enable trustless interoperability. Different spatial data formats will be supported through GeoDID Extensions - meaning GeoDIDs can support any current or future digital spatial data format. Required member variables will include relevant metadata to enable comprehensive indexing and use of the spatial data a GeoDID identifies.

We have designed the initial draft of the spec and would be curious for feedback from you or anyone interested when we release the spec to the community for review. Additionally, we invite anyone interested in using or contributing to the GeoDID project to reach out to us via Twitter @AstralProtocols or on our Discord [here](https://discord.gg/kBu3zSva).

GeoDIDs are agnostic to the spatial data assets being identified. However, we are designing the alpha implementation of the GeoDID modules to rely on IPFS by default, so we can cryptographically verify the integrity of the data assets referenced. 

## Oracles

We have done exceedingly little work on spatial oracles `link [hyperaware backend]`, `[sprout backend]`, instead focusing to date on the data storage and spatial contracts layers of the protocol stack. Our early thinking suggests that making a full suite of spatial analytics algorithms (raster and vector) available at the oracle layer would be useful for on-demand processing of geospatial data. 

For example, one concept protocol we have designed is a parametric insurance system. With this, we trustlessly insure physical assets in space - initially conceived of as static areas or volumes like land parcels or administrative jurisdictions (maritime, terrestrial, airspace etc). Upon purchasing a policy, agents would register their land parcel, represented using a GeoDID identifying a polygon or polyhedrohn. Additional information like the policy duration, indemnity process and, crucially, insured parameter and data source, would be specified upon policy creation. See this [relatively simple example](https://www.nature.org/en-us/what-we-do/our-insights/perspectives/insuring-nature-to-ensure-a-resilient-future/) deployed by traditional insurers.

Asset monitoring could be configured in a few ways. In the example above, periodic checks to the parameterized data feed could be made (would this be a point to use k3pr? or gelato network?), and a payout could be triggered automatically if the parameter threshold is exceeded. Alternatively, the insurance contract could be reactive, requiring a policy holder to submit a claim transaction. In this event, the contract would trigger the oracle to fetch both the land parcel information and the relevant parameterized external information. To enable a scalable, fully decentralized system, we suspect the most efficient architecture will require an oracle or some Layer 2 consensus network to apply a spatial analysis algorithm to these inputs to determine if the claim is valid. (This differs to many existing DeFi insurance protocols - these often rely on some entity - a trusted individual or DAO committee - to assess the evidence off chain and submit an attestation to settle a claim or trigger automatic indemnity - see [IBISA](https://ibisa.network/) and certain review strategies employed by Protekt's [Claims Manager](https://docs.protektprotocol.com/#/?id=protekt-contracts). (Obvs probably others too, not too familiar with insurance protocols - Nexus? Cover? Etc.))

This functionality was also required to detect the amount of time devices spent in policy zones in Hyperaware, and to supply NOx levels to the sustainability-linked bond dApp we prototyped during the KERNEL Genesis Block.  

Needless to say, much research into these oracle capabilities - including privacy-preserving techniques - for bringing spatial insights on chain in an efficient way is warranted, as it seems this is an unavoidably layer of the Astral stack.

| | |
| --- | --- | 
| Product researcher | 3 months | 
| Technical researcher | 3 months |
| Target deliverable | Research note on spatial data oracles | 

## Spatial.sol: a Solidity library of topological and geometric functions

Many of the applications we envision will require us to perform spatial operations in smart contracts. To this end, a few years ago (at ETHParis) we outlined and experimented with a Solidity library for performing these operations - measuring the length of a linestring, testing to see if lines intersect, checking to see if a point is inside a polygon, etc. We prototyped a location-aware wallet (early experiments that may lead to truly **local** currencies) At the time we were discouraged due to computation complexity leading to high gas costs, but Layer 2 solutions like Polygon and xDai, alternative lower cost EVM-compatible blockchains like IoTeX or Avalanche and, of course, Eth2 does mean that these kinds of operations will be increasingly viable in smart contracts.

We are developing the first version of the Spatial.sol library and plan to release it under an open source license. Developing the library entails tackling several technical challenges, and to build a production-quality library will be quite involved. 

First, Spatial.sol has a dependency: Trigonometry.sol, developed and open sourced by Sikorka ([docs](https://github.com/Sikorkaio/sikorka#trigonometry), [code](https://github.com/Sikorkaio/sikorka/blob/master/contracts/trigonometry.sol)). This library appears to have been abandoned a few years ago, and lacks a `tangent` method, which is useful in some operations like the [bearing](https://github.com/Turfjs/turf/blob/41a123a0e151be6a7e312dfecb91b69c4ff3f3f2/packages/turf-bearing/index.ts#L53) between two points. Spatial.sol will require an audited Trigonometry.sol to be secure.

We are working to design the most appropriate architecture, working out how these primitives could fit together. Spatial.sol - especially if we incorporate raster processing algorithms - will be useful in many Astral dApps. 


| | |
| --- | --- | 
| Solidity engineers | 3 months (?) | 
| Audit | ? |
| Target deliverable | `Spatial.sol v0.0`, `Trigonometry.sol v0.0` | 

## Verifiable spatial data registries

Leveraging GeoDIDs and Spatial.sol, we are early in the process of developing a standard contract for creating spatial data registries. There is a wide breadth of opportinities in this design space as well. The more interesting ones - and the ones specific to Astral - take into account spatial dimensions to the registration of records. 

In the Zora protocol, a piece of cryptomedia can be minted for every 256 bit number - media is hashed using Sha256 then minted. Once minted, this piece of cryptomedia is claimed, and another cannot be registered.

The spatial data registries standard we are designing aims to achieve the same functionality, but applied to physical territory in a spatial reference system. This most clearly applied to land parcels or administrative boundaries. These are "non-intersecting" boundaries, defined as Type 1 jurisdictions by [Hooghe and Marks (2003)](https://www.researchgate.net/publication/248233581_Unraveling_the_Central_State_but_How_Types_of_Multilevel_Governance).

We have been experimenting `[geolocker], [hyperaware jurisdiction registry]` with smart contracts that would enable a spatial data registry of polygons representing zones on the Earth's surface (or areas / volumes within any spatial reference system ...). Land registries have been touted as a great use case for blockchain, but all physical and virtual land registries we've seen either don't have spatial data on chain, represent land areas as grid cells or hexagons, or rely on a level of trust that polygons are maintained accurately (BenBen, Coadjute, IBISA Network, OVR, Coadjute and many others). This is easy to work with on the resource-constrained EVM, but fails to meet basic requirements for creating a functional system for representing and governing physical jurisdictions. 


We would like to develop a trustless spatial data registry that could detect and handle these topological intersections (along with, ). Consider the insurance protocol described above: if two entities could both register an intersecting area, it could enable insurance fraud. A smart contract that could detect these topological collisions and, perhaps, incorporating some hierarchical sovereignty system would be highly useful in a wide range of settings.

We would like to design this protocol to make polygons composable building blocks to use across Web3. Our understanding of the potential of this is quite nascent. One early project in our community that would benefit from this contract standard is Earthify, a project to incentivize the public to standardize (!) and onboard land parcel data from every county in the United States. Phase 1 of our development plan involves deploying contracts to govern and reward the acquisition of these data assets. Phase 2 will involve deploying them in a format consumable by other smart contracts. 

Another idea, being developed by community members blairvee and johnx25bd, are truly local currencies.  These are possible (if coupled with secure location proofs - a challenge described above) by creating location-bound smart contract methods. johnx25bd prototyped this concept at ETHParis 2019 with a Location Aware Wallet: a smart contract that would alter behavior based on a geographic position submitted with a transaction. In his simple prototype, an externally owned account submitted a transaction with a `recipient address` and an `amount`, as well as their `position`. Using Spatial.sol's `pointInPolygon` method, the smart contract then referenced that position against registered Jurisdiction geometries. Depending on the jurisdiction a transaction originated in, different sections of code were executed - paying a certain tax in one area, and another in a neighbor's territory. It could even be possible for currencies to have different values in different places ... 

We are curious what kind of opportunities could be created if we could detect and, eventually, arbitrate this kind of 'topological collision' - disputed borders - in a fully decentralized and trustless way. This type of contract  will be required for many different applications including land registries, insurance, spatial finance applications, location-based governance apps, local currencies, location-based NFTs etc etc. 

We're researching how to efficiently design a system that would enable this kind of a spatial data registry, along with requirements. We're trying to work out exactly what the needs might be - tough without any user engagement or feedback from the market - but we anticipate that this will emerge as a contract standard. We can imagine lots of variations of this type of contract deployed, each storing its own registry of polygons - for mobility apps, administrative jurisdictions, maritime governance zones, restricted airspace etc etc etc. The spatial reference system is configurable - so it could define zones in relation to the sun, another planet, a vehicle, and so on. Such a standard would contribute to composability of these spatial dApps.


| | |
| --- | --- | 
| Protocol engineer | 3 months |
| Solidity engineer | 3 months | 
| Typescript developer | 1 month |
| Audit | ? |
| Dependencies | | 
| Target deliverable | EIP? White paper? ERCx standard contract. ERCx.js package. | 

---

That's the state of our thinking right now - building tools to make it easy for developers to build location-based dApps. If any of this interests you please reach out. 