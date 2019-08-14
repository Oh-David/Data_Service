<template>
  <div>
    
  </div>
</template>

<script>
import { eventBus } from '~/plugins/eventBus.js'

export default {
  name: "status-panel",
  components: {
  },
  methods: {
    emitData() {
      this.$emit("combinedData", this.combinedData);
    }
  },
  mounted() {
    const localurl = 'http://localhost:3000/';
    var releaseJSON_URL = 'release.json';
    var responsetimeJSON_URL = 'responsetime.json';
    var statusJSON_URL = 'status.json';
    var transactionJSON_URL = 'transaction.json';
    var resourceJSON_URL = 'resource.json';
    var fetchDate = new Date();

    var releaseJSON = 
    fetch(localurl+releaseJSON_URL)
    .then(function(response) {
      return response.json()
    });

    var responseTimeJSON = 
    fetch(localurl+responsetimeJSON_URL)
      .then(function(response) {
        return response.json()
      });

    var statusJSON = 
    fetch(localurl+statusJSON_URL)
      .then(function(response) {
        return response.json()
      });

    var transactionJSON = 
    fetch(localurl+transactionJSON_URL)
      .then(function(response) {
        return response.json()
      });

    var resourceJSON = 
    fetch(localurl+resourceJSON_URL)
      .then(function(response) {
        return response.json()
      });
    
    var date = fetchDate;

    var combinedData = {
      releaseJSON: {},
      responseTimeJSON: {},
      statusJSON: {},
      transactionJSON: {},
      resourceJSON: {},
      date: {}
      };
    Promise.all([
      releaseJSON,
      responseTimeJSON,
      statusJSON,
      transactionJSON,
      resourceJSON,
      date
      ])
    .then(function(data) {
      combinedData["releaseJSON"] = data[0]["release"];
      combinedData["responseTimeJSON"] = data[1]["metrics"];
      combinedData["statusJSON"] = data[2];
      combinedData["statusJSON"] = data[2]["environments"];
      combinedData["statusJSON"] = data[2]["incidents"]
      combinedData["transactionJSON"] = data[3]["metrics"];
      combinedData["resourceJSON"] = data[4];
      combinedData["date"] = data[5];
      eventBus.$emit('jsonData', (combinedData));
      // return JSON.stringify(combinedData);
    });
  }
}
</script>

<style>

</style>