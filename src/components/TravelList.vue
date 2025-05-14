<script setup lang="ts">
import { ref } from 'vue'

const props = defineProps({
    id:Number,
    name:String,
    type:String,
    location:String,
})

</script>

<template>
  <div class="info-box" :data-i-id="props.id">
    <p>Name: {{ props.name }}</p>
    <p>Type: {{ props.type }}</p>
    <p>Location: {{ props.location }}</p>
    <div style="height: 10px;"></div> <!-- Small empty space -->
  </div>
</template>

<style scoped>
.info-box {
  border: 1px solid #ccc; /* Adds a light gray border */
  padding: 5px; /* Adds some padding inside the box */
  border-radius: 5px; /* Optional: Rounds the corners */
}
</style>