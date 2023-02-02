1. In the very last transaction of the Using Private Capabilities section in today's lesson, there was this line:
```rust #RRGGBB
// Borrow the capability
let minter = &ExampleNFT.NFTMinter = minterCapability.borrow() ?? panic("The capability is no longer valid.")
```
Explain in what scenario this would panic.

➡ Well you are borrowing the capability you created. At this point in the code it is saying that we thought a private cablity existed in this case at `/private/NFTMinter`
but it never got created or given to the account or it was unlinked. 

