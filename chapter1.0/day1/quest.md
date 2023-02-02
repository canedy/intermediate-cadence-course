1. Write a transaction to save a @Record.Collection to the signer's account, making sure to link the appropriate interfaces to the public path.

```rust
import Record from 0xf8d6e0586b0a20c7
import NonFungibleToken from 0xf8d6e0586b0a20c7

transaction() {
  
  prepare(signer: AuthAccount) {
    // Create a `@Record Collection` type
    let recordCollection: @NonFungibleToken.Collection <- Record.createEmptyCollection()

    // Store `@Profile` type
    signer.save(<- recordCollection, to: Record.CollectionStoragePath)
    
    // Link `@Profile` type to the public
    signer.link<&Record.Collection>(Record.CollectionPublicPath, target: Record.CollectionStoragePath)
  }

  execute {

  }
}
```

2. Write a transaction to mint some @Record.NFTs to the user's @Record.Collection

```rust
import Record from 0xf8d6e0586b0a20c7
import NonFungibleToken from 0xf8d6e0586b0a20c7

transaction(songName: String) {

  prepare(signer: AuthAccount) {
    // 1. Borrow reference to the Records Collection
    let collection = signer.borrow<&Record.Collection{Record.CollectionPublic, NonFungibleToken.CollectionPublic}>(from: Record.CollectionStoragePath)
                      ?? panic("The signer does not have a Collection.")

    // 2. Call createRecord() to create a record
    let record <- Record.createRecord(songName: songName)

    // 3. Deposit the created reocrd into the collection of the signer
    collection.deposit(token: <- record)

  }
}
```

3. Write a script to return an array of all the user's &Record.NFT? in their @Record.Collection

```rust
import Record from 0xf8d6e0586b0a20c7
import NonFungibleToken from 0xf8d6e0586b0a20c7

pub fun main(address: Address): [UInt64] {
  let publicCollection = getAccount(address).getCapability(Record.CollectionPublicPath)
                          .borrow<&Record.Collection{Record.CollectionPublic, NonFungibleToken.CollectionPublic}>()
                          ?? panic("The address does not have a Collection")

  return publicCollection.getIDs()
}
```

4. Write a transaction to save a @Artist.Profile to the signer's account, making sure to link it to the public so we can read it

```rust
import Artist from 0xf8d6e0586b0a20c7
import Record from 0xf8d6e0586b0a20c7

transaction(name: String) {

  prepare(signer: AuthAccount) {

    // 1. Create a capability
    let recordCapability: Capability<&Record.Collection{Record.CollectionPublic}> = signer.getCapability<&Record.Collection{Record.CollectionPublic}>(Record.CollectionPublicPath)

    // 2. Create a resource artist @profile
    let profile: @Artist.Profile <- Artist.createProfile(name: name, recordCollection: recordCapability)

    // 3. Save resource @profile type
    signer.save(<- profile, to: /storage/Profile)

    // 4. Link resource artist @profile to the public
    signer.link<&Artist.Profile>(/public/Profile, target: /storage/Profile)

  }

  execute {

  }
}
```

5. Write a script to fetch a user's &Artist.Profile, borrow their recordCollection, and return an array of all the user's &Record.NFT? in their @Record.Collection from the recordCollection

```rust
import Artist from 0xf8d6e0586b0a20c7
import Record from 0xf8d6e0586b0a20c7
import NonFungibleToken from 0xf8d6e0586b0a20c7

pub fun main(address: Address): [UInt64] {

  let artist = getAccount(address).getCapability(/public/Profile)
                .borrow<&Artist.Profile>()
                ?? panic("the address does not have a profile")


  let recordCapability: Capability<&Record.Collection{Record.CollectionPublic}> = artist.recordCollection

  let collection: &Record.Collection{Record.CollectionPublic} = recordCapability.borrow()!

  // let collection = getAccount(address).getCapability(Record.CollectionPublicPath)
  //                         .borrow<&Record.Collection{Record.CollectionPublic, NonFungibleToken.CollectionPublic}>()
  //                         ?? panic("The address does not have a Collection")

  return collection.getIDs()
}
```

6. Write a transaction to unlink a user's @Record.Collection from the public path

```rust
transaction() {
  
  prepare(signer: AuthAccount) {
    // Unlink your public profile
    signer.unlink(/public/Profile)
  }

  execute {

  }
}
```

7. Explain why the recordCollection inside the user's @Artist.Profile is now invalid

- Well, simple. We broke the link to the public storage. As we know, this is like an API where we created a link to allow public access to authorized information on the contract

8. Write a script that proves why your answer to #7 is true by trying to borrow a user's recordCollection from their &Artist.Profile

```rust
import Artist from 0xf8d6e0586b0a20c7

pub fun main(signer: Address): String {
  let account: PublicAccount = getAccount(signer)

  let capability: Capability<&Artist.Profile> = account.getCapability<&Artist.Profile>(/public/Profile1)

  // PANIC HAPPENS HERE
  let profile: &Artist.Profile = capability.borrow()!
  
  return profile.name
}
```
