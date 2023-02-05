1. Describe the access(contract) pattern in words.
- access(control), when set on variables or functions, allows all the contracts to call them. They can't be called from transactions or other contracts.

2. Come up with your own unique example of when the access(contract) pattern could be used.
* You could use it when updating a variable such as totals in the gaming point system. In this example, you only want to allow the sum to be updated when a points resource calls it from within the contract.

3. Based on the following diagram, do you think this pattern could also be used with access(account)?
* I assume you can. I don't know why. It works the same accept only the calling contracts can assess the variables or functions in another contract. All contracts must be in the same account, so other accounts would not have access.

4. Using the FLOAT Contract, find at least one example of the access(contract) pattern being used (hint: if your answer to quest #3 is correct, you should be able to find one by searching all the public interfaces for a certain function that has a specific access modifier).
* Did you mean access(account)? I'm not seeing a reference to access(contracts). But the pattern you are looking for is in the code below, I think. This code borrows the `GrantedAccountAccess` capability. This checks to see you are the list to be allowed to run other functions in the contract. 
```        pub fun borrowSharedRef(fromHost: Address): &FLOATEvents {
            let sharedInfo = getAccount(fromHost).getCapability(GrantedAccountAccess.InfoPublicPath)
                                .borrow<&GrantedAccountAccess.Info{GrantedAccountAccess.InfoPublic}>() 
                                ?? panic("Cannot borrow the InfoPublic from the host")
            assert(
                sharedInfo.isAllowed(account: self.owner!.address),
                message: "This account owner does not share their account with you."
            )
            let otherFLOATEvents = getAccount(fromHost).getCapability(FLOAT.FLOATEventsPublicPath)
                                    .borrow<&FLOATEvents{FLOATEventsPublic}>()
                                    ?? panic("Could not borrow the public FLOATEvents.")
            return otherFLOATEvents.borrowEventsRef()
        }```
