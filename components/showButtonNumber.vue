<template>
  <div>
    <buttonClick />
  </div>
</template>

<script>
import buttonClick from '~/components/buttonClick.vue'

export default {
  data() {
    return {
      count: 0
    }
  },
  components: {
    buttonClick
  },
}
</script>

<style>

</style>