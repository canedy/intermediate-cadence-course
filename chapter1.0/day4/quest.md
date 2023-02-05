1. Explain what a resource identifier is.

- It represents a cadence type. This can be resource type. You can get identfier information that would show you the address type was defined in, name of contract type was defined in, and the name of the type.

2. Is it possible for two different resources, that have the same type, to have the same identifier?

- No becuase the address part of the identfier would be diffrent, but the contract and type could be spoofed. 

3. Is it possible for two different resources, that have a different type, to have the same identifier?

- Yes, as a developer or one of those bad people you can create a resource with same contract name and have the same identifier or resouce name.

4. Based on the included comments:

- What is wrong with the following script?
  - The main problem is this line `.borrow<&{NonFungibleToken.CollectionPublic}>()` and specfically the fact they are using a &AnyResource. 

- Explain in detail how someone hack this script to always return true.
  - Becuase this is a &AnyResource a bad 🏴‍☠️'s could create a resouces type that memics the realone. Tricking the contract to think it is looking a legit resource, but really it is the bad 🏴‍☠️'s contract. They could then mint 100 nfts and get what they want, in this case to the return of true.

- Then, what are two ways we could fix this script to make sure it is safe?
  - Be explicit and practice good coding define the the full resource
  - Are add check to validate the identifier being passed is what you expect to be passed.

5. Rewrite this script such that we verify we are reading a balance from a FlowToken Vault.

import FungibleToken from 0xf233dcee88fe0abe

```pub fun main(user: Address): UFix64 {
  let vault = getAccount(user).getCapability(/public/Vault)
              .borrow<&{FungibleToken.Receiver}>()
              ?? panic("Your Vault is not set up correctly.")

  assert(
    vault.getType().identifier == "A.f233dcee88fe0abe.FungibleToken.Vault", 
      message: "No Bad 🏴‍☠️ you can't do this to me!!!!!!"
  )
  
  return vault.balance
}```

