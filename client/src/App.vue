<template>
  <v-app>
    <v-navigation-drawer
      fixed
      :clipped="$vuetify.breakpoint.width > 1264"
      v-model="drawer"
      class="grey lighten-4"
      app
    >
      <v-list
        dense
        class="grey lighten-4"
      >
        <v-layout row align-center>
          <v-flex xs6>
            <v-subheader>
              Contracts
            </v-subheader>
          </v-flex>
          <v-flex xs6 class="text-xs-right">
            <v-btn small flat @click.stop="dialog.show = !dialog.show"><v-icon>add</v-icon>add</v-btn>
          </v-flex>
        </v-layout>
        <template v-for="(item, i) in contracts">
          <v-list-tile
            :key="i"
            @click="viewContract(item)"
          >
            <v-list-tile-action>
              <v-icon>assignment</v-icon>
            </v-list-tile-action>
            <v-list-tile-content>
              <v-list-tile-title class="grey--text">
                {{ item.name }}
              </v-list-tile-title>
            </v-list-tile-content>
          </v-list-tile>
        </template>
        <v-list-tile v-if="contracts.length <=0">
          <v-list-tile-content>
            <v-list-tile-title class="grey--text text-xs-center">
              Click "add" to add a new contract
            </v-list-tile-title>
          </v-list-tile-content>
        </v-list-tile>
      </v-list>
    </v-navigation-drawer>
    <v-toolbar color="teal lighten-1" app absolute clipped-left>
      <v-toolbar-side-icon v-if="$vuetify.breakpoint.width <= 1264" @click="drawer = !drawer"></v-toolbar-side-icon>
      <span class="title">State of the Contracts</span>
    </v-toolbar>
    <v-content id="main">
      <v-container fluid class="grey lighten-4">
        <v-layout justify-center align-center>
          <contract :contract="contract"></contract>
        </v-layout>
      </v-container>
      
    </v-content>
    <v-dialog v-model="dialog.show" width="800px">
      <v-card>
        <v-card-title
          class="grey lighten-4 py-4 title"
        >
          Add a contract
        </v-card-title>
        <v-container grid-list-sm class="pa-4">
          <v-layout row wrap>
            <v-flex xs12 align-center justify-space-between>
              <v-layout align-center>
                <v-avatar size="40px" class="mr-3">
                  <img
                    src="//ssl.gstatic.com/s2/oz/images/sge/grey_silhouette.png"
                    alt=""
                  >
                </v-avatar>
                <v-text-field
                  placeholder="Name"
                  v-model="dialog.name"
                ></v-text-field>
              </v-layout>
            </v-flex>
            <v-flex xs12>
              <v-text-field
                prepend-icon="place"
                placeholder="Address"
                v-model="dialog.address"
              ></v-text-field>
            </v-flex>
            <v-flex xs12>
              <v-text-field
                prepend-icon="code"
                placeholder="ABI"
                multi-line
                v-model="dialog.abi"
              ></v-text-field>
            </v-flex>
          </v-layout>
        </v-container>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn flat color="primary" @click="dialog.show = false">Cancel</v-btn>
          <v-btn flat @click="newContract">Save</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-app>
</template>

<script>
  import Contract from '@/components/Contract'
  export default {
    data () {
      return {
        drawer: null,
        dialog: {
          show: false,
          name: '',
          address: '',
          abi: ''
        },
        contracts: [
          {name: 'CryptoKitties', address: '', abi: ''}
        ],
        contract: {
          name: '',
          address: '',
          abi: ''
        }
      }
    },
    methods: {
      newContract: function (event) {
        var self = this
        var nContract = {name: self.contract.name, address: self.contract.address, abi: self.contract.abi}
        self.contracts.push(nContract)
        self.contract = nContract
        self.dialog.show = false
      },
      viewContract: function (contract) {
        var self = this
        self.contract = contract
        self.dialog.show = false
        if (self.$vuetify.breakpoint.width <= 1264) self.drawer = false
      },
      components: {
        'contract': Contract
      }
    },
    mounted () {
      if (this.contracts.length > 0) this.contract = this.contracts[0]
    }
  }
</script>
