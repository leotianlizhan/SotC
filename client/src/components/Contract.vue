<template>
<v-container grid-list-md text-xs-center class="grey lighten-4">
  <v-layout v-if="contract.address.length!=42" row wrap justify-space-around align-center>
    <v-alert outline class="mt-4" color="info" icon="info" :value="true">
      <div class="headline">No contract selected</div>
    </v-alert>
  </v-layout>
  <div v-else>
  <v-layout row wrap justify-space-around align-center>
    <v-flex xs12>
      <h1>{{ contract.name }}</h1>
    </v-flex>
  </v-layout>
  <v-layout row wrap justify-space-around align-center>
    <v-progress-circular v-show="showLoading && hasEvents" indeterminate color="primary"></v-progress-circular>
    <div v-show="!showLoading" class="title mb-1">Found ya!</div>
  </v-layout>
  <v-layout row wrap justify-space-around v-if="events.length===0 && hasEvents">
    <div class="title mt-4">No event yet. Listening...</div>
  </v-layout>
  <!-- No Events Error -->
  <v-layout row wrap text-xs-left v-if="!hasEvents">
    <v-alert color="error" icon="warning" value="true">
      <div class="title">This contract has no events! There can be 3 reasons:</div>
      <ol class="ml-5 mt-2 body-1">
        <li>Your given ABI is incorrect.</li>
        <li>Your given ABI is incomplete (missing the events information).</li>
        <li>Assuming the ABI is correct and complete, this contract does not fire any event in the first place.</li>
      </ol>
    </v-alert>
  </v-layout>
  <v-layout row wrap justify-space-around>
    <v-flex xs12 md10>
      <v-expansion-panel focusable is="transition-group" name="list">
        <v-expansion-panel-content v-for="(item,i) in events" v-bind:key="item.txHash">
          <v-container slot="header" grid-list-md>
            <v-layout row wrap class="caption text-xs-left">{{item.blockNumber}}</v-layout>
            <v-layout row wrap class="title">{{item.name}}</v-layout>
          </v-container>
          <v-container slot="header" grid-list-md v-for="(param,j) in item.params" :key="j" v-if="j<maxParamsInHeader">
            <v-layout row class="caption text-xs-left">{{slice12(param.name)}}:</v-layout>
            <v-layout row class="body-1 text-xs-left">{{param.display}}</v-layout>
          </v-container>
          <div slot="header" v-if="item.params.length>maxParamsInHeader">...</div>
          <!-- Dropdown -->
          <v-card class="grey lighten-3">
            <v-card-text>
              <table>
                <tr>
                  <th align="right">Block #</th>
                  <td>{{item.blockNumber}}</td>
                </tr>
                <tr>
                  <th align="right">Transaction</th>
                  <td><a :href="etherscan + item.txHash" target="_blank">{{item.txHash}}</a></td>
                </tr>
                <tr>
                  <th><v-divider></v-divider></th>
                  <td><v-divider></v-divider></td>
                </tr>
                <tr v-for="(param,k) in item.params" :key="k">
                  <th align="right">{{param.name}}</th>
                  <td v-if="param.type==='address'"><a :href="etherscan + param.data" target="_blank">{{param.fullDisplay}}</a></td>
                  <td v-else>{{param.fullDisplay}}</td>
                </tr> 
              </table>
            </v-card-text>
          </v-card>
        </v-expansion-panel-content>
      </v-expansion-panel>
    </v-flex>
  </v-layout>
  </div>
</v-container>
</template>

<script>
import ethers from 'ethers'
var Contract = ethers.Contract
var provider = ethers.providers.getDefaultProvider();
var contract
var events = {}
export default {
  props: ['contract'],
  data () {
    return {
      etherscan: 'https://etherscan.io/search?q=',
      showLoading: true,
      hasEvents: true,
      events: [
      ]
    }
  },
  watch: {
    contract: function (newVal, oldVal) {
      var self = this
      console.log('Contract changed: ', newVal, ' | was: ', oldVal)
      self.events = [];
      if(newVal.address=='' || newVal.abi==null) return;
      // unregister previous listeners
      if(contract!=undefined){
        for (var Evt in contract.interface.events) {
          var evt = contract.interface.events[Evt]();
          provider.removeAllListeners(evt.topics);
        }
      }
      contract = new Contract(newVal.address, newVal.abi, provider)
      if(Object.keys(contract.interface.events).length===0){
        self.hasEvents = false;
      }else{
        self.hasEvents = true;
      }
      for (var Evt in contract.interface.events) {
        var evt = contract.interface.events[Evt]();
        console.log(evt);
        (function(curEvt){
          provider.on(curEvt.topics, log=>{
            if(log.address!=contract.address) return;
            // get parameters stored as arrayish
            var arrayish = curEvt.parse(log.topics, log.data)
            var parameters = []
            console.log(log);
            for(var i=0; i<arrayish.length; i++){
              if(curEvt.inputs[i].type==='address'){ 
                var fullDisplay = arrayish[i];
                var display = arrayish[i].slice(0,7) + '...';
              } else if(curEvt.inputs[i].type==='uint256' || curEvt.inputs[i].type==='bytes32') {
                var fullDisplay = arrayish[i].toString();
                var display = arrayish[i].toString();
                if(display.length>8) display = display.slice(0,8) + '...';
              } else {
                var fullDisplay = arrayish[i];
                var display = arrayish[i];
              }
              parameters.push({
                name: curEvt.inputs[i].name,
                type: curEvt.inputs[i].type,
                display: display,
                fullDisplay: fullDisplay,
                data: arrayish[i]
              })
            }
            self.events.unshift({
              name: curEvt.name,
              blockNumber: log.blockNumber,
              blockHash: log.blockHash,
              txHash: log.transactionHash,
              params: parameters
            })
          })
        })(evt);
      }
    },
    events: function (newVal, oldVal) {
      var self = this;
      self.showLoading = false;
      setTimeout(function(){
        self.showLoading = true;
      }, 1200);
    }
  },
  methods: {
    slice12: function(val) {
      if (val.length>12) return val.slice(0,12) + '..';
      else return val;
    }
  },
  computed: {
    maxParamsInHeader: function() {
      if(this.$vuetify.breakpoint.width<400) return 3;
      else return 7;
    }
  }
}
</script>

<style lang="css" scoped>
.list-item {
  display: inline-block;
  margin-right: 10px;
}
.list-enter-active {
  transition: all 1.5s;
}
.list-enter /* .list-leave-active below version 2.1.8 */ {
  opacity: 0;
  transform: translateY(-30px);
}
td{
  padding-left: 15px;
  word-break:break-all;
}
table{
  word-wrap:break-word;
  table-layout:fixed;
}
</style>