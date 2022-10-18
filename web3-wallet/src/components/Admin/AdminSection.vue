<template>
  <main>
    <div v-if="owner">
        
        <div class="options">
            <input v-model="contract" placeholder="enter contract address"/>
            <button class="success" @click="addContract">Add Contract</button>
        </div>
        <div v-for="(item, key) in list" :key="key" class="contract">            
            {{item.contract}}
            
            <div v-for="(item, key) in item.filter_traits" class="traits" :key="key">
                {{item.value}} -  {{item.trait_type}}
            </div>
            
            <div class="traits">
                <div class="form">
                    <input v-model="value" placeholder="enter value" />
                    <input v-model="trait_type" placeholder="enter trait trype" />
                    <button @click="blockTraits(item.contract)">Block</button>    
                </div>
                
            </div>
        </div>
    </div>
    <div v-else>
        You are not the owner Unfortunately
        
    
    </div>
  </main>
</template>


<script lang="js">

import whitelisting from "../whitelisting.json";
    export default{
        name: "AdminSection", 
        data(){
          return{
            owner: false,
            list: [], 
            contract: "",
            value: "", 
            trait_type:"", 
          }  
        },
        methods: {
            isOwner(){
                console.log(this.account);
                if(this.account == '0xfFf09621F09CAa2C939386b688e62e5BE19D2D56'.toLowerCase()) {this.owner = true}
            }, 
            
            listingContracts(){
                this.list = whitelisting;            
            },
            addContract(){
                
                if(this.contract.length < 10){
                    console.log("invalid contract....")
                    return
                }
                let item = this.list.find(el => el.contract == this.contract);
                if(item != undefined){
                    console.log("We already have this contract whitelisted...");
                    return
                }
                //here the function to right it to the correct place. 
                this.list.push({contract: this.contract, filter_traits: []});
                this.contract = "";
            }, 
            blockTraits(contractAddress){
                let item = this.list.findIndex(el => el.contract == contractAddress);
                console.log(item)
                
                
                if(this.value.length > 0 && this.trait_type.length > 0){
                    this.list[item].filter_traits.push({"value": this.value, "trait_type": this.trait_type})
                }
                
            
                
            }
        
        }, 
        mounted(){
          this.isOwner()  
          
          this.listingContracts();
        },
        watch: {
            account(newValue){
                this.isOwner(newValue);
                console.log("update the value")
                
                this.listingContracts();
            }
        },
        props:{
            account: String
        }
    }
</script>
<style scoped>
.contract{
    margin: 2rem;
    background: rgba(0,0,0,0.05);
    display: flex;
    flex-direction: column;
    border-radius: 8px;
    padding: 1rem;
}
.traits{
    margin: 1rem;
    font-weight: 100;
    font-style: italic;
}
.success{
    background-color: #bfe4d7;
    padding: 0.5rem;
    border-radius: 0px 8px 8px 0px; 
}
.options input{
    padding: 0.5rem;
    border: none;
    width: 350px;
    background-color: rgba(0,0,0,0.05)
}
.form input{
    padding: 0.5rem;
    margin: 0.2rem;
    text-transform: capitalize;
}
</style>