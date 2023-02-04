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

4. Architect and implement your idea in #3. Show:

Note: One thing I would do in the future is refactor this code so that minting the event and ticket resource happen in separate transactions. 

Event.cdc
```
import NonFungibleToken from 0xf8d6e0586b0a20c7

pub contract Event: NonFungibleToken {

  pub var totalSupply: UInt64

  pub event ContractInitialized()
  pub event Withdraw(id: UInt64, from: Address?)
  pub event Deposit(id: UInt64, to: Address?)

  pub let CollectionStoragePath: StoragePath
  pub let CollectionPublicPath: PublicPath
  pub let MinterStoragePath: StoragePath

  pub resource Ticket {
    pub let id: UInt64
    pub let name: String

    init(name: String) {
        self.id = self.uuid
        self.name = name
    }
  }

  pub resource NFT: NonFungibleToken.INFT {
    pub let id: UInt64
    pub let eventTitle: String

    pub var tickets: @{String: Ticket}

    init(eventTitle: String) {
      self.id = self.uuid
      self.eventTitle = eventTitle
      self.tickets <- {}
      Event.totalSupply = Event.totalSupply + 1
    }

    pub fun getTickets(): @{String: Ticket} {
        var other: @{String:Ticket} <- {}
        self.tickets <-> other
        return <- other
    }

    pub fun setTickets(tickets: @{String: Ticket}) {
        var other <- tickets
        self.tickets <-> other
        destroy other
    }

    destroy() {
        destroy self.tickets
    }
  }

  pub resource interface CollectionPublic {
      pub fun deposit(token: @NonFungibleToken.NFT)
      pub fun getIDs(): [UInt64]
      pub fun borrowNFT(id: UInt64): &NonFungibleToken.NFT
      pub fun borrowEventNFT(id: UInt64): &Event.NFT? {
          post {
              (result == nil) || (result?.id == id):
                  "Cannot borrow the reference: the ID of the returned reference is incorrect"
          }
      }
  }

  pub resource Collection: CollectionPublic, NonFungibleToken.Provider, NonFungibleToken.Receiver, NonFungibleToken.CollectionPublic {
      pub var ownedNFTs: @{UInt64: NonFungibleToken.NFT}

      pub fun withdraw(withdrawID: UInt64): @NonFungibleToken.NFT {
          let token <- self.ownedNFTs.remove(key: withdrawID) ?? panic("missing NFT")
          emit Withdraw(id: token.id, from: self.owner?.address)
          return <- token
      }

      pub fun deposit(token: @NonFungibleToken.NFT) {
          let token <- token as! @Event.NFT
          emit Deposit(id: token.id, to: self.owner?.address)
          self.ownedNFTs[token.id] <-! token
      }

      pub fun getIDs(): [UInt64] {
          return self.ownedNFTs.keys
      }

      pub fun borrowNFT(id: UInt64): &NonFungibleToken.NFT {
          return (&self.ownedNFTs[id] as &NonFungibleToken.NFT?)!
      }

      pub fun borrowEventNFT(id: UInt64): &Event.NFT? {
          if self.ownedNFTs[id] != nil {
              let ref = (&self.ownedNFTs[id] as auth &NonFungibleToken.NFT?)!
              return ref as! &Event.NFT
          }

          return nil
      }

      init () {
          self.ownedNFTs <- {}
      }

      destroy() {
          destroy self.ownedNFTs
      }
  }



  pub fun createEmptyCollection(): @NonFungibleToken.Collection {
      return <- create Collection()
  }

		pub resource interface AdminProxyPublic {
		pub fun depositAdmin(admin: Capability<&Admin>) {

			pre {
				admin.check(): "This capability is invalid!"
			}
		}
	}	

	pub resource AdminProxy: AdminProxyPublic {
		pub var admin: Capability<&Admin>?

		pub fun depositAdmin(admin: Capability<&Admin>) {
			self.admin = admin
		}

		init() {
			self.admin = nil
		}
	}

	pub fun createProxyAdmin(): @AdminProxy {
		return <- create AdminProxy()
	}

  pub resource Admin {
    
    pub fun createTicket(name: String): @Ticket {
        return <- create Ticket(name: name)
    }

  }

	pub resource interface MinterProxyPublic {
		pub fun depositMinter(minter: Capability<&NFTMinter>) {

			pre {
				minter.check(): "This capability is invalid!"
			}
		}
	}	

	pub resource MinterProxy: MinterProxyPublic {
		pub var minter: Capability<&NFTMinter>?

		pub fun depositMinter(minter: Capability<&NFTMinter>) {
			self.minter = minter
		}

		init() {
			self.minter = nil
		}
	}

	pub fun createProxy(): @MinterProxy {
		return <- create MinterProxy()
	}

  pub resource NFTMinter {

    pub fun mintNFT(eventTitle: String): @Event.NFT {
      return <- create NFT(eventTitle: eventTitle)
    }
    
  }

  init() {
      self.totalSupply = 0

      self.CollectionStoragePath = /storage/EventCollection
      self.CollectionPublicPath = /public/EventCollection
      self.MinterStoragePath = /storage/Minter

      self.account.save(<- create NFTMinter(), to: /storage/NFTMinter)
      self.account.save(<- create Admin(), to: /storage/Admin)

      emit ContractInitialized()
  }

}
```

createProxyAdmin.cdc
```
import Event from 0xf8d6e0586b0a20c7

transaction() {
  prepare(signer: AuthAccount) {
    signer.save(<- Event.createProxyAdmin(), to: /storage/AdminProxy)
    signer.link<&Event.AdminProxy{Event.AdminProxyPublic}>(/public/AdminProxy, target: /storage/AdminProxy)    
  }
}
```

giveAdminRights.cdc
```
import Event from 0xf8d6e0586b0a20c7

transaction(adminProxyAddress: Address) {
  // signer - the account with the NFTAdmin resource
  prepare(signer: AuthAccount) {
    // Link the NFTAdmin to a private path
    signer.link<&Event.Admin>(/private/Admin, target: /storage/Admin)

    // Get the private capability (remember: can only be accessed on AuthAccount)
    let admin: Capability<&Event.Admin> = signer.getCapability<&Event.Admin>(/private/Admin)

    // Get the AdminProxy from the recipient
    let adminProxy = getAccount(adminProxyAddress).getCapability(/public/AdminProxy)
              .borrow<&Event.AdminProxy{Event.AdminProxyPublic}>()
              ?? panic("This account does not have a public AdminProxy.")

    // Fulfill the AdminProxy's `admin` capability
    adminProxy.depositAdmin(admin: admin)
  }
}
```

createEventTicketsAsAdmin.cdc
```
import Event from 0xf8d6e0586b0a20c7
import NonFungibleToken from 0xf8d6e0586b0a20c7

transaction(eventTitle: String) {

  prepare(signer: AuthAccount) {

    // Borrow reference &Event to collection
    let collection = signer.getCapability(Event.CollectionPublicPath)
                      .borrow<&Event.Collection{Event.CollectionPublic, NonFungibleToken.CollectionPublic}>()
                      ?? panic("This account does not have a Collection.")


    // Borrow the &MinterProxy reference
    let minterProxy: &Event.MinterProxy = signer.borrow<&Event.MinterProxy>(from: /storage/MinterProxy)!
    // Get the capability
    let minterCapability: Capability<&Event.NFTMinter> = minterProxy.minter ?? panic("The capability has not been fulfilled.")
    // Borrow the capability
    let minter: &Event.NFTMinter = minterCapability.borrow() ?? panic("The capability is no longer valid.")
    // let minter = signer.borrow<&Event.NFTMinter>(from: /storage/NFTMinter)
    //               ?? panic("Signer did not deploy the contract")


    let adminProxy: &Event.AdminProxy = signer.borrow<&Event.AdminProxy>(from: /storage/AdminProxy)!
    // Get the capability
    let adminCapability: Capability<&Event.Admin> = adminProxy.admin ?? panic("The capability has not been fulfilled.")
    // Borrow the capability
    let admin: &Event.Admin = adminCapability.borrow() ?? panic("The capability is no longer valid.")
    // let admin = signer.borrow<&Event.Admin>(from: /storage/Admin)
    //               ?? panic("Signer does not Admin privileges")

    // Call createEvent() to crate an event
    let experience <- minter.mintNFT(eventTitle: eventTitle)

    let ticket1 <- admin.createTicket(name: "Seat Near Court");
    let ticket2 <- admin.createTicket(name: "Seat Near Upper Deck");

    let tickets <- experience.getTickets()

    let oldTicket <- tickets["t1"] <- ticket1
    destroy oldTicket

    let oldTicketx <- tickets["t2"] <- ticket2
    destroy oldTicketx

    experience.setTickets(tickets: <- tickets)
    
    // Deposit the create record into the colleciton of the signer
    collection.deposit(token: <- experience)

  }

  execute {
    
  }
}
```

