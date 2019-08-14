<template>
  <div class="message"># {{ count }}<br>
    <!-- <p># {{ count }}</p> -->
    <button v-on:click.prevent="increment">+</button>
    <button v-on:click.prevent="decrement">-</button>
  </div>
</template>

<script>

export default {
  name: 'buttonclicker',
  data() {
    return {
      count: 0
    }
  },
  props: {
    someNumber: Number,
    default: 0
  },
  methods: {
    increment () {
      this.count++;
    },
    decrement () {
      this.count--;
    }
  },
}
</script>

<style>
  .message{
    padding-bottom: 20px;
    font-weight: bold;
  }
</style>