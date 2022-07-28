<template>
  <div style="display: flex; flex-direction: column">
    <span>{{ voter }}</span>
    <span v-for="(pick, i) in picks" :key="i">{{ pick }}</span>
  </div>
</template>

<script>
export default {
  props: {
    picks: { type: Array, required: true, default: () => [] },
    voter: { type: String, required: true, default: `` },
  },
  data() {
    return {
      counter: 0,
    }
  },
  methods: {
    increment() {
      this.counter += 1
    }
  },
}
</script>

<style scoped>
  p {
    font-family: serif;
  }
</style>
