<template>
    <div id="app">
      <p class = "intro">Transfer Polkadot's DOT to Astar</p>
      
      <div class="box" style="margin-top: 12%;  font-family: 'Anybody', cursive;">
        You are logged in as {{$store.state.account}}.
      </div>

      <b-field class="textt" label-position="inside" label="Input recipient address">
        <b-input expanded @input.native="addrs($event)" v-model="addr"></b-input>
      </b-field>
      
      <b-field class="textt" label-position="inside" label="Input DOT amount">
          <b-input expanded @input.native="unit($event)" v-model="amount"></b-input>
      </b-field>
      
      <b-button class="buttonn" pack="fas" icon-right="file-import" expanded  type="is-primary" @click="sendXCM($store.state.account)">Send transaction</b-button>
    
    </div>
  </template>
  
  <script lang="ts">
    import { ApiPromise, WsProvider } from '@polkadot/api'
    import { defineComponent } from '@vue/composition-api'
    import '@polkadot/api-augment';
    import { web3FromAddress } from "@polkadot/extension-dapp"
    import { decodeAddress } from '@polkadot/util-crypto'
    export default defineComponent({
      name: "RelayToPara",
      data() {
        return {
          addr: "" as string,   //Recipient address is stored here
          amount: 0 as number,   //Required amount to be transfered is stored here
        };
      },
      methods: {
        //Used to store recipient address
        // eslint-disable-next-line 
        async addrs(value: any){
          this.addr=value.target.value
        },
        //Used to store user required transfer amount
        // eslint-disable-next-line 
        async unit(value: any){
          this.amount=value.target.value
        },
        //Used to create XCM transfer
        async sendXCM(address: string) {
            if(this.addr=="" )
            {
              this.$notify({ title: 'Error', text: 'You need to input recipient first.', type: 'error', duration: 3000,speed: 100})
            }
            else{
              if(this.amount<5000000000)
              {
                  this.$notify({ title: 'Error', text: 'Specified amount is less than required {5000000000} - 0.5DOT.', type: 'error', duration: 3000,speed: 100})
              }
              else{
                if(address == "none")
                {
                  this.$notify({ text: 'You need to login first.',type: 'error', duration: 5000,speed: 100})
                }
                else{
                  const wsProvider = new WsProvider('wss://polkadot.api.onfinality.io/public-ws');
                  const api = await ApiPromise.create({ provider: wsProvider });
                  var counter = 0
                  const injector = await web3FromAddress(address); // finds an injector for an address
                  console.log(`polakdotSigner ===> injector: `,injector);
                    //API call for XCM transfer From Relay chain to Parachains /w injected wallet
                        api.tx.xcmPallet
                        .reserveTransferAssets(
                        {
                            V1: {
                            parents: 0,
                            interior: {
                                X1: {
                                    Parachain: 2006
                                }
                            }
                            }
                        },
                        {
                            V1: {
                            parents: 0,
                            interior: {
                                X1: {
                                AccountId32: {
                                network: "any",
                                id: api
                                    .createType("AccountId32", decodeAddress(this.addr))
                                    .toHex()
                                }
                                }
                            }
                            }
                        },
                        {
                            V1: [
                            {
                                id: {
                                Concrete: {
                                    parents: 0,
                                    interior: "Here"
                                }
                                },
                                fun: {
                                Fungible: this.amount
                                }
                            }
                            ]
                        },
                        0
                        )
                        .signAndSend(address, { signer: injector.signer }, ({ status, txHash }) => {
                        if(counter == 0){    
                            this.$notify({ text: `Transaction hash is ${txHash.toHex()}`, duration: 10000,speed: 100})
                            counter++
                        }
                        if (status.isFinalized) {
                            this.$notify({ text: `Transaction finalized at blockHash ${status.asFinalized}`,type: 'success', duration: 10000,speed: 100})
                        }
                        });
                    }
                    }
                }
                }
            }
    })
  </script>
  
  <style scoped>
    @import url('https://fonts.googleapis.com/css2?family=Orbitron&display=swap');
    #app {
      font-family: Avenir, Helvetica, Arial, sans-serif;
      -webkit-font-smoothing: antialiased;
      -moz-osx-font-smoothing: grayscale;
      text-align: center;
      color: #2c3e50;
      background-color: white;
      margin-top: 20px;
      margin-left: 20%;
      margin-right: 20%;
    }
    select {
      width: 150px;
      margin: 10px;
    }
    select:focus {
      min-width: 150px;
      width: auto;
    }
    .textt{
      color: black;
      font-family: "Anybody", cursive;
      font-size: 30px;
      margin-bottom: 20px;
    }
    .intro{
      color: #2a49f5;
      font-family: "Orbitron", cursive;
      font-size: 30px;
      margin-bottom: 20px;
    }
  </style>