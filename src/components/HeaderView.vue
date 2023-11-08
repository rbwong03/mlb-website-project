<template>
  <header>
    <nav>
      <router-link to="/">Rankings</router-link> |
      <router-link to="/schedule">Schedule</router-link>
    </nav>
  </header>
</template>

<script lang="ts">
import { defineComponent } from "vue";

export default defineComponent({
  name: "HeaderView",
});
</script>

<style scoped>
header {
  padding: 16px;
  background-color: #081d4d;
  color: white;
}

nav a {
  color: white;
  text-decoration: none;
  margin: 0 10px;
  font-size: 1.5em;
}

nav a.router-link-exact-active {
  color: #d50032;
}
</style>
