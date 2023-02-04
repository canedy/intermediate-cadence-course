1. In the very last transaction of the Using Private Capabilities section in today's lesson, there was this line:
```rust #RRGGBB
// Borrow the capability
let minter = &ExampleNFT.NFTMinter = minterCapability.borrow() ?? panic("The capability is no longer valid.")
```
Explain in what scenario this would panic.

- Well you are borrowing the capability you created. At this point in the code it is saying that we thought a private cablity existed in this case at `/private/NFTMinter` but it never got created or given to the account or it was unlinked. 

2. Explain two reasons why passing around a private capability to a resource you own is different from simply giving users a resource to store in their account.

- Private storage is owned by the account and people or things that are given special access.

3. Write (in words) a scenario where you would use private capabilities. It cannot be the same NFT example we saw today

- Given that a you are creating an Event and Ticketing system. You could set up contract that only the event admin can mint a new event and assign tickets to that event. But once the event starts the Admin might want to give tempoary staff the ability to mint tickets and then revoke after the event.

![image](https://user-images.githubusercontent.com/2507134/216756116-c6b26934-5361-450b-b5c3-974585ccec4a.png)



