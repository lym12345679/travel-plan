<script setup lang="ts">
import { ref } from 'vue'
import IconHotel from '@/components/icons/IconHotel.vue'
import IconSpot from '@/components/icons/IconSpot.vue'
import IconTransportation from '@/components/icons/IconTransportation.vue'

const props = defineProps({
  id:Number,
  name:String,
  type:String,
  location:String,
  imageUrl: String, // 新增图片链接属性
})

</script>

<template>
  <div class="info-box" :data-i-id="props.id">
    <span v-if="props.type === 'hotel'" class="icon">
      <IconHotel />
    </span>
    <span v-else-if="props.type === 'spot'" class="icon">
      <IconSpot />
    </span>
    <span v-else-if="props.type === 'transportation'" class="icon">
      <IconTransportation />
    </span>
    <img v-if="props.imageUrl" :src="props.imageUrl" alt="图片" class="info-image" /> <!-- 添加图片展示 -->
    <img v-else src="../assets/nopicture.jpg" alt="Placeholder" class="info-image" /> <!-- 默认图片 -->
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

.icon{
  width: 50px;
  height: 50px;
}

.info-image {
  width: 100px;
  height: 100px;
  object-fit: cover;
  margin: 10px 0;
}
</style>