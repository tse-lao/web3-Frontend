<script lang="js">
import Web3 from "web3";



export default {
  name: "WalletVue",
  data() {
    return {
      account: null,
      network: null,
    };
  },
  methods: {
    async initiate() {
      const web3 = new Web3(window.ethereum);
        console.log(web3)

        window.ethereum.on('accountsChanged', function (accounts) {
            this.account = accounts[0];
    });

     // detect Network account change
    window.ethereum.on('networkChanged', function(networkId){
      console.log('networkChanged',networkId);
    });
      try {
        // Request full provider if needed
        
        const result= await window.ethereum.request({method: 'eth_requestAccounts'})
        // Full provider exposed
        console.log(result);
        this.account = result[0];

        this.$emit('update:account', result[0]);
      } catch (error) {
        // User denied full provider access
      }
    },
  },
  watch:{
    
  }
};
</script>
<template>
  <main>
    <div class="wallet">
        <div v-if="account != null">
            <button class="truncate" @click="initiate">{{ this.account }}</button>
        </div>

    <div v-else>
      <button @click="initiate">Wallet Connect</button>
    </div>
    </div>
  
  </main>
</template>
<style scoped>
.wallet{
    display: flex;
    flex-direction: column-reverse;
    align-items: flex-end;
    margin-right: 2rem;
}
button {
  background: #bfe4d7;
  border: none;
  border-radius: 24px;
  padding: 1rem;
}

.truncate{
    width: 150px;
    white-space: nowrap;
    overflow:hidden;
    text-overflow: ellipsis;
}

</style>