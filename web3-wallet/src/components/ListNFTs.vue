<script lang="js">
import { Alchemy, Network } from "alchemy-sdk";
import whitelisting from "./whitelisting.json";

const settings = {
  apiKey: process.env.VUE_APP_SECRET_KEY, // Replace with your Alchemy API Key.
  network: Network.ETH_MAINNET, // Replace with your network.
};

const alchemy = new Alchemy(settings);

export default {
  name: "ListNFTs",
  data() {
    return {
      nfts: [],
      invalidNft: [],
      showInvalid: false, 
      totalNfts: 0,
    };
  },
  setup() {},
  methods: {
    async getAllNfts() {
        ///reset it all
        this.nfts = []
        this.invalidNft = []

      // Print owner's wallet address:
      const ownerAddr = "0x0dbfb8e5d9e25cb9fda369c3662a0b94919b172e";
    //const ownerAddr = this.account;

      console.log("fetching NFTs for address:", ownerAddr);
      
      const ownersNFT = await alchemy.nft.getNftsForOwner(ownerAddr);
        this.totalNfts = ownersNFT;
        
        for (const nft of ownersNFT.ownedNfts) {
            this.getNFTDetails(nft.contract.address, nft.tokenId);
        }
    },

    async getNFTDetails(contractAddress, tokenID) {
      const response = await alchemy.nft.getNftMetadata(
        contractAddress,
        tokenID
      );

      //now we put them in a list.

      let { status, result } = this.whiteListedContract(
        response.contract.address, 
        response.rawMetadata.attributes
      );
      
      console.log(result)

      if (status) {
        this.nfts.push({
          contract: response.contract,
          metaData: response.rawMetadata,
          media: response.media[0],
        });
      } else {
        this.nfts.push({
          contract: response.contract,
          metaData: response.rawMetadata,
          media: response.media[0],
        });
      }
    },

    whiteListedContract(contractAddress, attributes ) {
      let findItem = whitelisting.find((el) => el.contract === contractAddress);
      if (findItem != undefined) {
        if(findItem.filter_traits != undefined){
            let {status, result} = this.checkBannedTraits(findItem.filter_traits, attributes);
            
            return {status: status, result: result}
        }   return { status: true, result: "perfectly done"}
      } return {status: false,result: "Contract: " + contractAddress + "is not on the whitelist."};
    },

    checkBannedTraits(bannedTraits, traits) {
      for (var i = 0; i < bannedTraits.length; i++) {
        for (var y = 0; y < traits.length; y++) {
          if (bannedTraits[i].value === traits[y].value && bannedTraits[i].trait_type == traits[y].trait_type) {
              return {status: false, result: "trait type matched with banne trait_type: " + traits[y].trait_type + "and value: "+ traits[y].value}
          }
        }
      }  return {status: true, result: "everything is good!"}
    },

    useImage(image) {
      console.log(image);
      this.emit("update:background", image);
      this.$emit("update:showCropping", true);
    },
  },
  props:{
    account:String
  },
  watch: {
    account(newValue){
        this.getAllNfts();
        console.log("we switched to new account: " + newValue)
    }
  }
  
};
</script>
<template>
  <main v-if="account!=null">
    
      <div class="options">
          <button v-if="invalidNft.length > 0 && !showInvalid" @click="showInvalid = !showInvalid">View Blocked NFT</button>
          <button v-if="nfts.length > 0 && showInvalid" @click="showInvalid = !showInvalid">Show whitelisted NFT</button>
          
          <img @click="getAllNfts" src="../assets/refresh.png" alt='refresh.png'/>
      </div>
      <div v-if="showInvalid">
            <div v-if="invalidNft.length > 0">
                <div class="banned-list" v-for="(item, key) in invalidNft" :key="key">
                    <span>
                        {{item.contract.name}}
                    </span>
                        
                    <span class="error-text">
                        {{item.result}}
                    </span>
                </div>
            </div>
        </div>
    <div v-else>
            <div class="items" v-if="nfts.length > 0">
                <div class="list-item" v-for="(item, key) in nfts" :key="key">
                <img
                    :src="item.media.gateway"
                    alt="could not find thumbnail"
                    @click="
                    (e) => {
                        $emit('update:picture', item.media.gateway);
                        $emit('update:showCropping', true);
                    }
                    "
                />
                <div class="nft-contract-address">
                    <span>{{ item.contract.name }} </span>
                    <span
                    >{{ item.metaData.edition }} /
                    {{ item.contract.totalSupply }}</span
                    >
                </div>
                </div>
            </div>
            <div v-else>
                Unfortunately we have not found valid NFT
            </div>
      </div>
   
  </main>
</template>
<style scoped>
main{
    margin: 2rem;
}
.items {
  display: flex;
  flex-direction: row;
  gap: 2rem;
  flex-wrap: wrap;
  margin: 2rem;
}
.options{
    display: flex;
    align-content: center;
    align-items: center;
    gap: 2rem;
}

.options img:hover{
    transform: rotate(90deg);
    cursor: pointer;
}
.error-text{
    color: #C0533F;
    font-weight: 600;
}
span {
  text-align: left;
}
.nft-contract-address {
  background: #bfe4d7;
  width: auto;
  padding: 1rem 0rem;
}

.banned-list{
display: flex;
flex-direction: row;
gap: 2rem;
font-size: 1.2rem;
margin: 2rem;
}

.list-item {
  border: 1px solid black;
  flex: 1;
  display: flex;
  flex-direction: column;
  border-radius: 8px;
}

.list-item img {
  width: 100%;
}

.tabs {
  margin-top: 2rem;
}

button {
  background: rgba(0,0,0,0.2);
  border: none;
  border-radius: 6px;
  padding: 1rem;
}
span::after {
  content: "\A";
  white-space: pre;
}

@media (min-width: 600px) {
    .list-item {
       max-width: calc(100vw / 4);
    }
}
</style>