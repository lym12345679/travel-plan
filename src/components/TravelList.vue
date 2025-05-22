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
    <div class="info-content">
    <div class="info-header">
      <span v-if="props.type === 'hotel'" class="icon">
        <IconHotel />
      </span>
      <span v-else-if="props.type === 'spot'" class="icon">
        <IconSpot />
      </span>
      <span v-else-if="props.type === 'transportation'" class="icon">
        <IconTransportation />
      </span>
      <p class="info-name">{{ props.name }}</p>
    </div>
      <img
        v-if="props.imageUrl"
        :src="props.imageUrl"
        alt="Image"
        class="info-image"
      />
      <img
        v-else
        src="../assets/nopicture.jpg"
        alt="Placeholder"
        class="info-image"
      />
    </div>
    <div class="info-details">

      <p class="info-location">{{ props.location }}</p>
    </div>
  </div>

</template>

<style scoped>
.info-box {
  display: flex;
  flex-direction: column;
  align-items: center;
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 15px;
  margin: 10px;
  background-color: #f9f9f9;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s, box-shadow 0.2s;
}
.info-content {
  display: flex;
  flex-direction: row;
  align-items: center;
}
.info-box:hover {
  transform: scale(1.02);
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
}

.info-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 10px;
}

.icon {
  width: 50px;
  height: 50px;
  margin-bottom: 10px;
}

.info-image {
  width: 100px;
  height: 100px;
  object-fit: cover;
  border-radius: 8px;
  margin-bottom: 10px;
  margin-left: 8px;
}

.info-details {
  text-align: center;
}

.info-name,
.info-type,
.info-location {
  margin: 5px 0;
  font-size: 14px;
  color: #333;
}
</style>