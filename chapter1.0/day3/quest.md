1. Explain the two most common mistakes Cadence developers make regarding poor capability links.

- They don't add the complete set of interfaces for NonFungibleToken
- They use reference &AnyResource

2. Rewrite this script from the NBATopShot official repo to link users' collections properly.

-  ```acct.link<**&TopShot.Collection**{NonFungibleToken.CollectionPublic, TopShot.MomentCollectionPublic, MetadataViews.ResolverCollection}>(/public/MomentCollection, target: /storage/MomentCollection)``
