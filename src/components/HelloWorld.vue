<template>
  <div>
    <h1>{{ msg }}</h1>
    <h1 @click="$store.commit('add')">{{$store.state.counter}}</h1>
    <p>
      <a href="https://vitejs.dev/guide/features.html" target="_blank">
        Vite Documentation
      </a>
      |
      <a href="https://v3.vuejs.org/" target="_blank">Vue 3 Documentation</a>
    </p>

    <button @click="state.count++">count is: {{ state.count }}</button>
    <p>
      Edit to test hot module replacement.
    </p>
    <button @click="getApi">click</button>
  </div>
</template>

<script setup>
import { defineProps, reactive } from 'vue'

defineProps({
  msg: String
})
const getApi = () => {
  fetch("/api/getUsers")
      .then((res) => res.json())
      .then((data) => console.log(data));
};

const state = reactive({ count: 0 })
</script>

<style scoped>
a {
  color: #42b983;
}
</style>
