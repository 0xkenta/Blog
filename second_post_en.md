Introduction

There were so many changes in web3 area 2021. NFT was one of their main topics. We heard much news of NFT. For example, purchases by celebrities, big companies’ collections, selling by auction houses, and so on. You might have some NFTs already. Now people take an interest in NFT. However, you probably don’t know what metadata of NFT is. In this post, I want to talk about metadata and the NFT of Dev protocol. 

Metadata of NFT?

There are so many explanations of what NFT is. So we dive directly into the metadata of NFT. 

What is metadata?

Metadata is defined as data that provides information about other data.

In the case of NFT, EIP-721 explains that 

The metadata extension is OPTIONAL for ERC-721 smart contracts. This allows your smart contract to be interrogated for its name and for details about the assets which your NFTs represent.

In another word, metadata is the properties of each NFT.

How to get metadata?

First, you need to get HTTP or IPFS URL to retrieve metadata. The function tokenURL returns HTTP or IPFS URL as value. If you access this HTTP or IPFS URL, you can see metadata with JSON Schema like the below example.

The OpenSea page gives the following example of metadata.

```json
{
  "description": "Friendly OpenSea Creature that enjoys long swims in the ocean.", 
  "external_url": "https://openseacreatures.io/3", 
  "image": "https://storage.googleapis.com/opensea-prod.appspot.com/puffs/3.png", 
  "name": "Dave Starbelly",
  "attributes": [ ... ], 
}
```json

Usage of metadata?

Metadata contains the properties of each NFT. For instance, the property “image” gives us HTTP or IPFS URL and we can access images, videos, and music with these URLs. Marketplaces like OpenSea show us the properties of NFT based on this metadata. Now you have understood what metadata of NFT is.

sToken?

You may imagine that you purchase Art, image, and music on the marketplace like OpenSea if you think of NFT. But you can mint NFT on Dev protocol like some of you know. So let’s dive into metadata of Dev Protocol NFT.

How to get NFT on Dev Protocol?

NFT of Dev Protocol is called sToken. So what exactly is sToken? The next quote tells us what sToken is.

sToken is an NFT given as a certificate of staking to patrons when they support (stake) creators.

sToken is a token that is provided by staking. Please keep in mind that.

Feature of sToken?

Now creators can add illustrations or images to their minted NFT. So you can potentially get a unique asset with staking.

Here I summarizethe NFT of Dev Protocol.

①A Token that proves the staking
②Creators can put additional a unipue asset into NFT

Please check this post if you want to know the detail of sToken.

sToken of metadata?

Maybe you know already what metadata of Dev Protocol NFT contains. There must be above 2 pieces of information in the metadata. 

In STokensManager.sol of Dev Protocol, you can find the tokenURL function that we talked about. This function returns Base64. If you try to decode this Base64 you can see metadata that is written with JSON schema. This metadata is composed of 3 properties “name”, “description”, and “image”.

①“name” describes pieces of information about the Property Address that you staked and the amount of staking.
②In “image” image information that is encoded with Base64 is stored.

As you see, the metadata of sToken contains some properties like staking information and unique asset by creators that exactly represent Dev protocol.

Conclusion

We went through the metadata of NFT. Please try to find metadata of your sToken if you have already staked in Dev Protocol. If you are considering to stake, you can try to find your favorite art and stake your DEV on the creators. Thank you for reading.

References

https://eips.ethereum.org/EIPS/eip-721
 
Merriam Webster,
https://www.merriam-webster.com/dictionary/metadata
 
IPFS Blog, How to Store and Maintain NFT MetaData,
https://blog.ipfs.io/how-to-store-and-maintain-nft-metadata/
 
OpenSea, Metadata Standardds,
https://docs.opensea.io/docs/metadata-standards
 
Openzeppelin forum, How to provide metadata for ERC721,
https://forum.openzeppelin.com/t/how-to-provide-metadata-for-erc721/4057/3
 
Yusuke Ikeda, Launching NFT art function of sToken,
https://initto.devprotocol.xyz/ja/s-token-update/
