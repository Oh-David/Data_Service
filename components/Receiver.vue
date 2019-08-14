<template>
  <div>
    <buttonClick />
  </div>
</template>

<script>
import { eventBus } from '~/plugins/eventBus.js'

import buttonClick from '~/components/buttonClick.vue'

export default {
  name: 'receiver',
  components: {
    buttonClick,
  },
  created() {
    eventBus.$on('jsonData', (json) => {
      this.combinedData = json;
    });
  },
  mounted() {
    eventBus.$on('jsonData', (json) => {
      this.combinedData = json;
      console.log(this.combinedData)
    });
  }
}
</script>

<style>
</style>