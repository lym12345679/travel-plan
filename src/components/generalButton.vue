<script setup lang="ts">
import { ref } from 'vue'

const props = defineProps({
  title:String,
});

const isClicked = ref(true);
const onClick = () => {
  isClicked.value = !isClicked.value;
  //console.log(`Button clicked: ${props.title}`);
};

</script>

<template>
  <div :class="isClicked?'gen-button':'gen-button-clicked'" @click="onClick">{{title}}</div>
</template>

<style scoped>
.gen-button {
  display: inline-block;
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
  text-align: center;
}
.gen-button:hover {
  background-color: #0056b3;
}
.gen-button:active {
  background-color: #004085;
}

.gen-button-clicked {
  display: inline-block;
  padding: 10px 20px;
  background-color: #0056b3;
  color: white;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
  text-align: center;
}
.gen-button-clicked:hover {
  background-color: #004085;
}
.gen-button-clicked:active {
  background-color: #003057;
}
</style>