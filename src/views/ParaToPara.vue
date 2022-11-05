<template>
    <div id="app">
      <p  v-if="testnetSwitch == 'Astar'" class = "intro">Transfer Astar's ASTR to any Parachain on Polkadot</p>
      <p  v-if="testnetSwitch == 'Shiden'" class = "intro">Transfer Shiden's SDN to any Parachain on Kusama</p>

      <div>
        <p style = "display: inline-block; margin-right: 15px;" class="texttt">Select network you wish to use for transfering</p>
        <b-switch style = "margin-top: 6px" v-model="testnetSwitch"
                true-value="Astar"
                false-value="Shiden">
                {{testnetSwitch}}
        </b-switch>
      </div>

      <div class="box" style="margin-top: 5%;  font-family: 'Anybody', cursive;">
        You are logged in as {{$store.state.account}}.
      </div>

      <b-field v-if="testnetSwitch=='Astar'" class="textt" label-position="inside" label="Select destination parachain on Polkadot">
        <b-select v-model="keyy" expanded placeholder="Select parachain 2" required>
          <option v-for="(item) in items" :key="item">{{item}}</option>
        </b-select>
      </b-field>

      <b-field v-if="testnetSwitch=='Shiden'" class="textt" label-position="inside" label="Select destination parachain on Kusama">
        <b-select v-model="keyy" expanded placeholder="Select parachain 2" required>
          <option v-for="(item) in itemsK" :key="item">{{item}}</option>
        </b-select>
      </b-field>
  
      <b-field class="textt" label-position="inside" label="Input recipient address">
        <b-input expanded @input.native="addrs($event)" v-model="addr"></b-input>
      </b-field>

      <b-field class="textt" label-position="inside" label="Select destination account type">
        <b-select v-model="type" expanded required>
          <option v-for="(type) in accTypes" :key="type">{{type}}</option>
        </b-select>
      </b-field>

      <b-field class="textt" label-position="inside" label="Input asset amount">
        <b-input expanded @input.native="unit($event)" v-model="amount"></b-input>
      </b-field>
      
      <b-button class="buttonn" expanded  type="is-primary" label="Send transaction" pack="fas" icon-right="file-import" @click="sendXCM($store.state.account)"/>
  
    </div>
  </template>
  
  <script lang="ts">
    import { ApiPromise, WsProvider } from '@polkadot/api'
    import { defineComponent } from '@vue/composition-api'
    import { decodeAddress } from '@polkadot/util-crypto'
    import { web3FromAddress } from "@polkadot/extension-dapp"
    import '@polkadot/api-augment';
    export default defineComponent({
    
      data() {
        return {
          items: [] as Array<number>,   //Stores Parachains connected to Relay chain Polkadot
          itemsK: [] as Array<number>,   //Stores Parachains connected to Relay chain Kusama
          keyy: "" as string,   //Selected destination parachain
          addr: "" as string,   //Recipient address is stored here
          amount: 0 as number,   //Required amount to be transfered is stored here
          type: "" as string,
          testnetSwitch: "Shiden",
          accTypes: [] as Array<string>,
        };
      },
      mounted: async function () {
        this.queryParas()
        
        this.accTypes.push("AccountKey20")
        this.accTypes.push("AccountId32")
      },
      methods: {
        //Used to store recipient address
        // eslint-disable-next-line 
        async addrs(value: any){
          this.addr=value.target.value
        },

        async queryParas(){
            this.$notify({ text: `Fetching Parachains connected to Relay chain. Please wait for success notification.`, duration: 6000,speed: 100})
            const wsProvider = new WsProvider('wss://polkadot.api.onfinality.io/public-ws');
            const api = await ApiPromise.create({ provider: wsProvider });
            //API call to query Parachains connected to Relay chain
            const parachain = await api.query.paras.parachains()
            const queryPara = JSON.stringify(parachain)
            const newParas = queryPara.split('[').join(',').split(']').join(',').split(',')
            const results = newParas.filter(element => {return element !== "";});
            const extractedParas = results.map((i) => Number(i));
            this.items = extractedParas


            const wsProvider2 = new WsProvider('wss://public-rpc.pinknode.io/kusama');
            const api2 = await ApiPromise.create({ provider: wsProvider2 });
            //API call to query Parachains connected to Relay chain
            const parachain2 = await api2.query.paras.parachains()
            const queryPara2 = JSON.stringify(parachain2)
            const newParas2 = queryPara2.split('[').join(',').split(']').join(',').split(',')
            const results2 = newParas2.filter(element => {return element !== "";});
            const extractedParas2 = results2.map((i) => Number(i));
            this.itemsK = extractedParas2
            this.$notify({ text: 'Parachains connected to Relay chain fetched successfuly!', type:"success", duration: 4000,speed: 100})
        },
        //Used to store user required transfer amount
        // eslint-disable-next-line 
        async unit(value: any){
          this.amount=value.target.value
        },
        //Used to create XCM transfer
        async sendXCM(address: string) {
          if(this.keyy == "") 
          {
            this.$notify({ title: 'Error', text: 'You did not select parachain correctly.', type: 'error', duration: 3000,speed: 100})
          }
          else 
          {
            if(this.addr=="")
            {
              this.$notify({ title: 'Error', text: 'You need to input recipient first.', type: 'error', duration: 3000,speed: 100})
            }
            else{
              if(this.amount<1000000000000000000)
              {
                this.$notify({ title: 'Error', text: 'Specified amount is less than required {1000000000000000000} - 1 Astar.', type: 'error', duration: 3000,speed: 100})
              }
              else{
                if(address == "none")
                {
                  this.$notify({ text: 'You need to login first.',type: 'error', duration: 5000,speed: 100})
                }
                else{
                var counter = 0
                const injector = await web3FromAddress(address); 
                let wsProvider
                if(this.testnetSwitch == "Astar"){
                  wsProvider = new WsProvider('wss://public-rpc.pinknode.io/astar');
                }
                else if (this.testnetSwitch == "Shiden"){
                  wsProvider = new WsProvider('wss://rpc.shiden.astar.network');
                }
                const api = await ApiPromise.create({ provider: wsProvider });
                if(this.type == "AccountId32"){
                  //Transfer scenario Parachain to Parachain
                    api.tx.polkadotXcm.reserveTransferAssets(
                        {
                            V1: {
                            parents: 1,
                            interior: {
                                X1: {
                                    Parachain: this.keyy
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
                  .signAndSend(address, { signer: injector.signer },  ({ status, txHash }) => {
                    if(counter == 0){    
                        this.$notify({ text: `Transaction hash is ${txHash.toHex()}`, duration: 10000,speed: 100})
                        counter++
                      }
                      if (status.isFinalized) {
                        this.$notify({ text: `Transaction finalized at blockHash ${status.asFinalized}`,type: 'success',duration: 10000,speed: 100})
                      }
                  });
                }
                else if (this.type == "AccountKey20"){
                  //Transfer scenario Parachain to Parachain
                    api.tx.polkadotXcm.reserveTransferAssets(
                        {
                            V1: {
                            parents: 1,
                            interior: {
                                X1: {
                                    Parachain: this.keyy
                                }
                            }
                            }
                        },
                        {
                            V1: {
                            parents: 0,
                            interior: {
                                X1: {
                                AccountKey20: {
                                network: "any",
                                id: this.addr
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
                  .signAndSend(address, { signer: injector.signer },  ({ status, txHash }) => {
                    if(counter == 0){    
                        this.$notify({ text: `Transaction hash is ${txHash.toHex()}`, duration: 10000,speed: 100})
                        counter++
                      }
                      if (status.isFinalized) {
                        this.$notify({ text: `Transaction finalized at blockHash ${status.asFinalized}`,type: 'success',duration: 10000,speed: 100})
                      }
                  });
                }
              }
            }
            }
          }
        }
      }
    })
  </script>
  
  <style scoped>
    @import url("https://fonts.googleapis.com/css2?family=Anybody:wght@300&family=BIZ+UDGothic&family=Pacifico&display=swap");
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
    .texttt{
      color: black;
      font-family: "Anybody", cursive;
      font-size: 15px;
      margin-bottom: 20px;
    }
    .intro{
      color: #2a49f5;
      font-family: "Orbitron", cursive;
      font-size: 30px;
      margin-bottom: 20px;
    }
  </style>