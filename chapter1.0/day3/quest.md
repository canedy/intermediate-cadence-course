1. Explain the two most common mistakes Cadence developers make regarding poor capability links.

- They don't add the complete set of interfaces for NonFungibleToken
- They use reference &AnyResource

2. Rewrite this script from the NBATopShot official repo to link users' collections properly.

- acct.link<***&TopShot.Collection***{***TopShot.MomentCollectionPublic, NonFungibleToken.CollectionPublic, NonFungibleToken.Receiver, MetadataViews.ResolverCollection***}>(/public/MomentCollection, target: /storage/MomentCollection)

3. What do you think the transaction looked like to set up this user's collection?

- acct.link<&ExampleNFT.Collection{ExampleNFT.CollectionPublic}>() ?? panic("Your collection is not set up correctly.")

4. What do you think the transaction looked like to set up this user's collection?

- acct.link<&{NonFungibleToken.CollectionPublic}>() ?? panic("Your collection is not set up correctly.")
