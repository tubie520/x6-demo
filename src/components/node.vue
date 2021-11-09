<template>
  <div class="node" :data-id="data.id">
    <div>{{  data.id }}</div>
    <div class="node-data-list">
      <div
        v-for="item in relations"
        :key="item.id" class="node-data-item"
        :data-id="item.id"
      >
        {{item.name}}
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, inject, onMounted }  from "vue";
// let node = ref({});
let relations = ref([]);
const data = ref({});
onMounted(() => {
  // const graph = inject("getGraph")();
  const node = inject("getNode")();
  if (!node) return;
  data.value = node.getData();
  relations.value = data.value.relations;
  console.log(node)
});

</script>
<style>
.node {
  min-width: 200px;
  min-height: 200px;
  background: #fff;
  border: 1px solid red;
}
</style>