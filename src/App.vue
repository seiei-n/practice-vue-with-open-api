<template>
  <div id="app" class="title">
    <h1>都道府県の総人口推移</h1>
  </div>
  <Graph />
</template>

<script>
import Graph from './components/Graph.vue';

export default {
  components: {
    Graph
  }
};
</script>

<style scoped>
#app {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100px;
}
</style>
