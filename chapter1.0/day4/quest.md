1. Explain what a resource identifier is.

- It represents a cadence type. This can be a resource type. You can get identifier information that would show you the defined address type, the contract the type is defined in, and the name of the type.

2. Is it possible for two different resources, that have the same type, to have the same identifier?

- No, because the address part of the identifier would be different, but the contract and type could be spoofed.

3. Is it possible for two different resources, that have a different type, to have the same identifier?

- Yes, as a developer or one of those bad people, you can create a resource with the same contract name and have the same identifier or resource name.

4. Based on the included comments:

- What is wrong with the following script?
  - The main problem is this line .borrow<&{NonFungibleToken.CollectionPublic}>(), and specifically, they are using a &AnyResource.

- Explain in detail how someone hack this script to always return true.
  - Because this is a &AnyResource, bad 🏴‍☠️'s could create a resource type that mimics the real one. Tricking the contract to think it is looking like a legit resource, but really it is the bad 🏴‍☠️'s contract. They could then mint 100 NFTs and get what they want, in this case, to the return of true.

- Then, what are two ways we could fix this script to make sure it is safe?
  - Be explicit and practice good coding to define the the full resource.
  - Are add check to validate the identifier being passed is what you expect to be passed.

5. Rewrite this script such that we verify we are reading a balance from a FlowToken Vault.

```import FungibleToken from 0xf233dcee88fe0abe

pub fun main(user: Address): UFix64 {
  let vault = getAccount(user).getCapability(/public/Vault)
              .borrow<&{FungibleToken.Receiver}>()
              ?? panic("Your Vault is not set up correctly.")

  assert(
    vault.getType().identifier == "A.f233dcee88fe0abe.FungibleToken.Vault", 
      message: "No Bad 🏴‍☠️ you can't do this to me!!!!!!"
  )
  
  return vault.balance
}```

