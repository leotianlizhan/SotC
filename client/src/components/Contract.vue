<template>
<v-container grid-list-md text-xs-center>
  <v-layout row wrap>
    <v-flex xs12>
      <h1>{{ contract.name }}</h1>
    </v-flex>
  </v-layout>
  <v-data-table
      v-bind:headers="headers"
      :items="events"
      hide-actions
      class="elevation-1"
    >
    <template slot="items" slot-scope="props">
      <td>{{ props.item.name }}</td>
      <td class="text-xs-right">{{ props.item.calories }}</td>
      <td class="text-xs-right">{{ props.item.fat }}</td>
      <td class="text-xs-right">{{ props.item.carbs }}</td>
      <td class="text-xs-right">{{ props.item.protein }}</td>
      <td class="text-xs-right">{{ props.item.sodium }}</td>
      <td class="text-xs-right">{{ props.item.calcium }}</td>
      <td class="text-xs-right">{{ props.item.iron }}</td>
    </template>
  </v-data-table>
</v-container>
</template>

<script>
import ethers from 'ethers'
var Contract = ethers.Contract
var contract
var events = {}
export default {
  props: ['contract'],
  data () {
    return {
      headers: [
        {
          text: 'Event Name',
          align: 'left',
          sortable: false,
          value: 'name'
        },
        { text: 'Calories', value: 'calories' },
        { text: 'Fat (g)', value: 'fat' },
        { text: 'Carbs (g)', value: 'carbs' },
        { text: 'Protein (g)', value: 'protein' },
        { text: 'Sodium (mg)', value: 'sodium' },
        { text: 'Calcium (%)', value: 'calcium' },
        { text: 'Iron (%)', value: 'iron' }
      ],
      events: [
        {
          value: false,
          name: 'Frozen Yogurt',
          calories: 159,
          fat: 6.0,
          carbs: 24,
          protein: 4.0,
          sodium: 87,
          calcium: '14%',
          iron: '1%'
        },
        {
          value: false,
          name: 'Ice cream sandwich',
          calories: 237,
          fat: 9.0,
          carbs: 37,
          protein: 4.3,
          sodium: 129,
          calcium: '8%',
          iron: '1%'
        }
      ]
    }
  },
  watch: {
    contract: function (newVal, oldVal) {
      var self = this
      console.log('Prop changed: ', newVal, ' | was: ', oldVal)
      if(newVal.address=='' || newVal.abi==null) return;
      var provider = ethers.providers.getDefaultProvider();
      contract = new Contract(newVal.address, newVal.abi, provider)
      for (var Evt in contract.interface.events) {
        var evt = contract.interface.events[Evt]();
        console.log(evt.inputs);
        (function(curEvt){
          provider.on(curEvt.topics, log=>{
            var arrayish = curEvt.parse(log.topics, log.data)
            for(var i=0; i<arrayish.length; i++) {
              self.events.push({
                name: curEvt.name
              })
            }
          })
        })(evt);
      }
    }
  }
}
</script>