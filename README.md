# web3-Frontend
Web3  wallet connect application. 

1. run npm install to isntall all the dependencies. 
2. Create an account on Alchemy. Enter the following secret codes.. 
      VUE_APP_SECRET_KEY=XX
      VUE_APP_INFURA=XXX
      VUE_APP_INFURA_SECRET=XXX
      
3. Now the app should work as the demo. 


#FUNCTIONALITIES. 
**1. ListNFTs.vue **
List all the NFT based on the registered user profile. Before we list the users we provide a check if the smart contract is allowed by the owners. 
These threats and smart contracts are now manually placed in the file whitelisting.json
You can display the list of all the allowed NFTS, but there is also an option to display all NFTs that are not allowed. So in another functionality a users can for instance requests for whitelisting. 
Your action: integrate it to the your own database server.

**2. The CreateProfilePicture.vue.** 
It loads the picture of the selected NFT in the cardholder. where the users can perform dragging dropping and zooming on the card to position according to their needs.
It can also change the background color to any color they want. The default color of the background is set to the first pixel of the image. 
The user is able to manual change the color by typing it in. Or to use the picker and change it to any color on the screen. 
 When the user creates the pictures it is ready to be uploaded to whenever server you provide. 


**3. Wallet functionality**
Wallet is a simple connect to get the users contract address. It needs to be integrate with your own functionalities of the address, but the only necessary thing to retrieve is the users address. 

