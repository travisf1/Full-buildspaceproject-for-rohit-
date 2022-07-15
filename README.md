# Full-buildspaceproject-for-rohit-
Full buildspace project including contract, scripts, and transactions.

## Contract


```cadence
import NonFungibleToken from 0x631e88ae7f1d7c20;
import MetadataViews from 0x631e88ae7f1d7c20;


pub contract buildSpacemyPicsfinalproj: NonFungibleToken {

  pub var totalSupply: UInt64

  pub event ContractInitialized()
  pub event Withdraw(id: UInt64, from: Address?)
  pub event Deposit(id: UInt64, to: Address?)

  pub let CollectionStoragePath: StoragePath
  pub let CollectionPublicPath: PublicPath
  pub let MinterStoragePath: StoragePath //added this

  pub resource NFT: NonFungibleToken.INFT, MetadataViews.Resolver { 
    pub let id: UInt64

    pub let name: String
    pub let description: String
    pub let thumbnail: String

    init(
      id: UInt64,
      name: String,
      description: String,
      thumbnail: String,
    ) {
      self.id = id
      self.name = name
      self.description = description
      self.thumbnail = thumbnail
    }
                
    pub fun getViews(): [Type] {
      return [
        Type<MetadataViews.Display>()
      ]
    }

      

    pub fun resolveView(_ view: Type): AnyStruct? {
      switch view {
        case Type<MetadataViews.Display>():
          return MetadataViews.Display(
            name: self.name,
            description: self.description,
            thumbnail: MetadataViews.HTTPFile(
              url: self.thumbnail
            )
          )
      }
      return nil
    }
  }

        
  pub resource interface buildSpacemyPicsfinalprojCollectionPublic {
    pub fun deposit(token: @NonFungibleToken.NFT)
    pub fun getIDs(): [UInt64]
    pub fun borrowNFT(id: UInt64): &NonFungibleToken.NFT
    /*add function below to match docs contract?, unsure if needed
    pub fun borrowbuildSpacemyPicsfinalproj(id: UInt64): &buildSpacemyPicsfinalproj.NFT? {
            post {
                (result == nil) || (result?.id == id):
                "Cannot borrow buildSpacemyPicsfinalproj reference: the ID of returned reference is incorrect"
            }
    }
            */

  }

  pub resource Collection: buildSpacemyPicsfinalprojCollectionPublic, NonFungibleToken.Provider, NonFungibleToken.Receiver, NonFungibleToken.CollectionPublic, MetadataViews.ResolverCollection {
    pub var ownedNFTs: @{UInt64: NonFungibleToken.NFT}

    init () {
      self.ownedNFTs <- {}
    }

    pub fun getIDs(): [UInt64] {
      return self.ownedNFTs.keys
    }
    
    pub fun withdraw(withdrawID: UInt64): @NonFungibleToken.NFT {
      let token <- self.ownedNFTs.remove(key: withdrawID) ?? panic("missing NFT")

      emit Withdraw(id: token.id, from: self.owner?.address)

      return <-token
    }

    pub fun deposit(token: @NonFungibleToken.NFT) {
      let token <- token as! @buildSpacemyPicsfinalproj.NFT

      let id: UInt64 = token.id

      let oldToken <- self.ownedNFTs[id] <- token

      emit Deposit(id: id, to: self.owner?.address)

      destroy oldToken
    }

    pub fun borrowNFT(id: UInt64): &NonFungibleToken.NFT {
      return (&self.ownedNFTs[id] as &NonFungibleToken.NFT?)!
    }

    
    pub fun borrowViewResolver(id: UInt64): &AnyResource{MetadataViews.Resolver} {
      let nft = (&self.ownedNFTs[id] as auth &NonFungibleToken.NFT?)!
      let buildSpacemyPicsfinalproj = nft as! &buildSpacemyPicsfinalproj.NFT
      return buildSpacemyPicsfinalproj as &AnyResource{MetadataViews.Resolver}
    }
          

    destroy() {
      destroy self.ownedNFTs
    }
  }

  pub fun createEmptyCollection(): @NonFungibleToken.Collection {
    return <- create Collection()
  }


  pub resource NFTMinter {

//changed from mintNFT
  pub fun myMintNFT( 
    recipient: &{NonFungibleToken.CollectionPublic},
    name: String,
    description: String,
    thumbnail: String,
  ) {

    let metadata: {String: AnyStruct} = {} // added this
    
     


    var newNFT <- create NFT(
      id: buildSpacemyPicsfinalproj.totalSupply,
      name: name,
      description: description,
      thumbnail: thumbnail
    )

    recipient.deposit(token: <-newNFT)

    buildSpacemyPicsfinalproj.totalSupply = buildSpacemyPicsfinalproj.totalSupply + UInt64(1)
  }
  }
  init() {
    self.totalSupply = 0

    self.CollectionStoragePath = /storage/buildSpacemyPicsfinalprojCollection
    self.CollectionPublicPath = /public/buildSpacemyPicsfinalprojCollection
    self.MinterStoragePath = /storage/buildSpacemyPicsfinalprojMinter //added this

    let collection <- create Collection()
    self.account.save(<-collection, to: self.CollectionStoragePath)

    self.account.link<&buildSpacemyPicsfinalproj.Collection{NonFungibleToken.CollectionPublic, buildSpacemyPicsfinalproj.buildSpacemyPicsfinalprojCollectionPublic, MetadataViews.ResolverCollection}>(
      self.CollectionPublicPath,
      target: self.CollectionStoragePath
    )

    let minter <- create NFTMinter() // added this
    self.account.save(<-minter, to: self.MinterStoragePath) //added this 

    emit ContractInitialized()
  }
}

```


## flow.json

```json
{
	"emulators": {
		"default": {
			"port": 3569,
			"serviceAccount": "emulator-account"
		}
	},
	"contracts": {
		"buildSpace": "./contracts/buildSpaceproject.cdc",
		"buildSpacemyPics": "./contracts/myPic-bS-project.cdc",
		"buildSpacemyPicsfinal": "./contracts/buildSpacefinal.cdc",
		"buildSpacemyPicsfinalproj": "./contracts/buildSpacefinalproj.cdc"
	},
	"networks": {
		"emulator": "127.0.0.1:3569",
		"mainnet": "access.mainnet.nodes.onflow.org:9000",
		"testnet": "access.devnet.nodes.onflow.org:9000"
	},
	"accounts": {
		"emulator-account": {
			"address": "f8d6e0586b0a20c7",
			"key": "1937014e69082f827ce06a1d486afcf284676dcef0430ebc9e786c23be11b6d1"
		},
		"testnet-account": {
			"address": "0x29f4ad0f552505bb",
			"key": "a7d70602ab23b3b45f62f6385e33fe94a6a1e2e451671778fcf2a4b3c301dd2b"
			
		}
	},
	"deployments": {
		"testnet": {
			"testnet-account": [
				"buildSpacemyPicsfinalproj"
			]
		}
	}
}

```

## Scripts


### getId_script

```js

export const getIDs = 
`
import MetadataViews from 0x631e88ae7f1d7c20;

pub fun main(address: Address): [UInt64] {
    
  let account = getAccount(address)

  let collection = account
    .getCapability(/public/buildSpacemyPicsfinalprojCollection)
    .borrow<&{MetadataViews.ResolverCollection}>()
    ?? panic("Could not borrow a reference to the collection")

  let IDs = collection.getIDs()
  return IDs;
}
;
`

```

### get metadata_script

```cadence

export const getMetadata = 
`
import MetadataViews from 0x631e88ae7f1d7c20;

pub fun main(address: Address, id: UInt64): NFTResult {
  
  let account = getAccount(address)

  let collection = account
      .getCapability(/public/buildSpacemyPicsfinalprojCollection) // Update the path here!
      .borrow<&{MetadataViews.ResolverCollection}>()
      ?? panic("Could not borrow a reference to the collection")

  let nft = collection.borrowViewResolver(id: id)

  var data = NFTResult()

  // Get the basic display information for this NFT
  if let view = nft.resolveView(Type<MetadataViews.Display>()) {
    let display = view as! MetadataViews.Display

    data.name = display.name
    data.description = display.description
    data.thumbnail = display.thumbnail.uri()
  }

  // The owner is stored directly on the NFT object
  let owner: Address = nft.owner!.address

  data.owner = owner

  return data
}
pub struct NFTResult {
  pub(set) var name: String
  pub(set) var description: String
  pub(set) var thumbnail: String
  pub(set) var owner: Address
  pub(set) var type: String

  init() {
    self.name = ""
    self.description = ""
    self.thumbnail = ""
    self.owner = 0x0
    self.type = ""
  }
}



`

```

### getTotalSupply_script


```js

export const getTotalSupply =
`
// REPLACE THIS WITH YOUR CONTRACT NAME + ADDRESS

import buildSpacemyPicsfinalproj from 0x29f4ad0f552505bb;

pub fun main(): UInt64 {

    return buildSpacemyPicsfinalproj.totalSupply;

}
`

```


## Transactions (myMintNFT_tx.js)

```js
export const myMintNFT = //changed to myMintNFT from mintNFT

`

import buildSpacemyPicsfinalproj from 0x29f4ad0f552505bb 
 
import NonFungibleToken from 0x631e88ae7f1d7c20
import MetadataViews from 0x631e88ae7f1d7c20


transaction{

    let minter: &buildSpacemyPicsfinalproj.NFTMinter

    prepare(signer: AuthAccount) {
        
        self.minter = signer.borrow<&buildSpacemyPicsfinalproj.NFTMinter>(from: buildSpacemyPicsfinalproj.MinterStoragePath)
        ?? panic("Could not borrow a reference to the NFT Minter")
    }

  execute {

    let receiver = getAccount(0x29f4ad0f552505bb)
        .getCapability(buildSpacemyPicsfinalproj.CollectionPublicPath)
        .borrow<&{NonFungibleToken.Collection}>()
        ?? panic("Could not get receiver reference to the NFT Collection")
        
        //changed from self.minter.mintNFT
  self.minter.myMintNFT(
    recipient: receiver,
    name: "",        //name, (took out to get rid of error)
    description: "", //description, (took this out)
    thumbnail: "",   //thumbnail, (took this out)
  )      

  log("Minted an NFT and stored it into the collection")
  }  

}


`

```
