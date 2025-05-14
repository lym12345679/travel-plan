<template>
  <div class="folding-button-container">
    <!-- 圆形折叠按钮 -->
    <button class="folding-button" @click="toggleIcons">
      <span :class="isExpanded ? 'icon-close' : 'icon-expand'">×</span>
    </button>

    <!-- 图标容器 -->
    <transition name="slide">
      <div v-if="isExpanded" class="icons-container">
        <div
          class="icon-item"
          :class="{ 'icon-selected': selectedIcons.hotel }"
          @click="handleIconClick('hotel')"
        >
          <div class="icon-wrapper">
            <IconHotel />
            <div v-if="selectedIcons.hotel" class="icon-slash"></div>
          </div>
          <span>酒店</span>
        </div>
        <div
          class="icon-item"
          :class="{ 'icon-selected': selectedIcons.spot }"
          @click="handleIconClick('spot')"
        >
          <div class="icon-wrapper">
            <IconSpot />
            <div v-if="selectedIcons.spot" class="icon-slash"></div>
          </div>
          <span>景点</span>
        </div>
        <div
          class="icon-item"
          :class="{ 'icon-selected': selectedIcons.transportation }"
          @click="handleIconClick('transportation')"
        >
          <div class="icon-wrapper">
            <IconTransportation />
            <div v-if="selectedIcons.transportation" class="icon-slash"></div>
          </div>
          <span>交通</span>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import IconHotel from './icons/IconHotel.vue'
import IconSpot from './icons/IconSpot.vue'
import IconTransportation from './icons/IconTransportation.vue'

// 控制图标显示状态
const isExpanded = ref(false)

// 定义图标类型
type IconType = 'hotel' | 'spot' | 'transportation'

// 图标选择状态
const selectedIcons = ref({
  hotel: false,
  spot: false,
  transportation: false,
})

// 切换图标显示/隐藏
const toggleIcons = () => {
  isExpanded.value = !isExpanded.value
}

// 点击图标时的处理函数
const handleIconClick = (type: IconType) => {
  // 切换该图标的选中状态
  selectedIcons.value[type] = !selectedIcons.value[type]
  console.log(`选择了 ${type}，状态: ${selectedIcons.value[type] ? '禁用' : '启用'}`)
  // 发出事件通知父组件
  emit('filterChange', { type, selected: selectedIcons.value[type] })
}

// 定义要发出的事件
const emit = defineEmits<{
  (e: 'filterChange', payload: { type: string; selected: boolean }): void
}>()
</script>

<style scoped>
.folding-button-container {
  position: relative;
  display: flex;
  justify-content: start;
  align-items: start;
}

.folding-button {
  width: 40px;
  height: 40px;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 50%;
  cursor: pointer;
  font-size: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
}

.folding-button:hover {
  background-color: #45a049;
  transform: scale(1.05);
}

.icon-expand,
.icon-close {
  line-height: 1px;
  position: relative;
  top: -1px;
  transition: transform 0.3s;
}

.icon-expand {
  transform: rotate(-45deg);
}

.icons-container {
  display: flex;
  margin-left: 16px;
}

.icon-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 0 8px;
  cursor: pointer;
  transition: transform 0.2s;
}

.icon-item:hover {
  transform: scale(1.1);
}

.icon-wrapper {
  position: relative;
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.icon-item svg {
  width: 30px;
  height: 30px;
}

.icon-slash {
  position: absolute;
  width: 100%;
  height: 2px;
  background-color: #ff0000;
  transform: rotate(-45deg);
  top: 50%;
  left: 0;
}

.icon-item span {
  margin-top: 4px;
  font-size: 11px;
}

/* 过渡动画 */
.slide-enter-active,
.slide-leave-active {
  transition: all 0.3s ease;
}

.slide-enter-from,
.slide-leave-to {
  transform: translateX(-20px);
  opacity: 0;
}
</style>
