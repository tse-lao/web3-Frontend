<template>
  <div class="nav">
    <button @click="admin = !admin">Admin Section</button>
    <Wallet v-model:account="account"/>
  </div>
    
  <div v-if="admin">
    <AdminSection :account="account"/>
  </div>
  <div v-else>
    <ListNFTs v-model:picture="picture" v-model:showCropping="showCropping" :account="account" v-if="!showCropping" />
  <CreateProfilePicture :backgroundImage="picture" v-model:showCropping="showCropping" v-else/>
  </div>
 
</template>

<script>

import AdminSection from './components/Admin/AdminSection.vue';
import CreateProfilePicture from './components/CreateProfilePicture.vue';
import ListNFTs from './components/ListNFTs.vue';
import Wallet from './components/wallet/Wallet.vue';

export default {
  name: 'App',
  data(){
    return {
      picture: "https://res.cloudinary.com/alchemyapi/image/upload/mainnet/32e76bd2af54947b98f96987261b912d.png",
      showCropping: false, 
      account: null,
      admin: false,
    }
  },
  components: {
    Wallet,
    CreateProfilePicture,
    ListNFTs,
    AdminSection
}
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.nav{
  display: flex;
  justify-content: space-between;
}

button{
  border: none;
  border-radius: 8px;
  padding: 1rem;
  background: rbga(0,0,0,0.1);
  text-transform: uppercase;
}

button:hover{
  background: rgba(0,0,0,0.4);
  cursor: pointer;
}

</style>
