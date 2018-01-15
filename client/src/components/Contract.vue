<template>
<v-container grid-list-md text-xs-center>
  <v-layout row wrap>
    <v-flex xs12>
      <h1>{{ contract.name }}</h1>
    </v-flex>
  </v-layout>
  <v-list>
    <v-card>
      <v-card-title>
        <div>Event</div>
        <div>Parameters</div>
      </v-card-title>
    </v-card>
  </v-list>
</v-container>
</template>

<script>
export default {
  props: ['contract'],
  data () {
    return {

    }
  }
}
</script>