<script setup lang="ts">
import { ref, watch, nextTick, renderList } from 'vue'
import FoldingButton from '@/components/FoldingButton.vue'
import IconHotel from '@/components/icons/IconHotel.vue'
import IconSpot from '@/components/icons/IconSpot.vue'
import IconTransportation from '@/components/icons/IconTransportation.vue'
import TravelList from '@/components/TravelList.vue'
import * as events from 'node:events'

// 控制内容的显示状态
const showHotels = ref(true)
const showSpots = ref(true)
const showTransportation = ref(true)

// 控制右侧旅游清单的折叠状态
const isPlanExpanded = ref(true)

// 切换旅游清单的显示/隐藏
const togglePlan = () => {
  isPlanExpanded.value = !isPlanExpanded.value
}


// 定义类型
interface Position {
  x: number // 百分比位置
  y: number // 百分比位置
}

interface Hotel {
  id: number
  name: string
  location: string
  price: string
  position: Position
}

interface Spot {
  id: number
  name: string
  location: string
  rating: number
  position: Position
}

interface Transportation {
  id: number
  type: string
  route: string
  frequency: string
  position: Position
}


// 模拟数据
const hotels: Hotel[] = [
  { id: 10001, name: '海景酒店', location: '市中心', price: '¥680/晚', position: { x: 25, y: 35 } },
  { id: 10002, name: '山顶度假村', location: '山区', price: '¥520/晚', position: { x: 75, y: 20 } },
]

const spots: Spot[] = [
  { id: 20001, name: '历史博物馆', location: '东部', rating: 4.5, position: { x: 65, y: 40 } },
  { id: 20002, name: '海滩公园', location: '南部', rating: 4.7, position: { x: 30, y: 70 } },
  { id: 20003, name: '古城墙', location: '西部', rating: 4.3, position: { x: 15, y: 55 } },
]

const transportations: Transportation[] = [
  { id: 30001, type: '公交', route: '1路', frequency: '每15分钟一班', position: { x: 40, y: 45 } },
  { id: 30002, type: '地铁', route: 'A线', frequency: '每10分钟一班', position: { x: 55, y: 65 } },
]

// 选中的项目（旅游清单）
const selectedPlan = ref<{
  hotels: Hotel[]
  spots: Spot[]
  transportations: Transportation[]
}>({
  hotels: [],
  spots: [],
  transportations: [],
})

// 处理过滤器变化
const handleFilterChange = (payload: { type: string; selected: boolean }) => {
  // 根据类型和状态更新显示
  if (payload.type === 'hotel') {
    showHotels.value = !payload.selected
  } else if (payload.type === 'spot') {
    showSpots.value = !payload.selected
  } else if (payload.type === 'transportation') {
    showTransportation.value = !payload.selected
  }
}

// 检查项目是否已被选中
const isItemSelected = (type: string, id: number): boolean => {
  if (type === 'hotel') {
    return selectedPlan.value.hotels.some((hotel) => hotel.id === id)
  } else if (type === 'spot') {
    return selectedPlan.value.spots.some((spot) => spot.id === id)
  } else if (type === 'transportation') {
    return selectedPlan.value.transportations.some((transport) => transport.id === id)
  }
  return false
}

// 切换选择状态 - 确保所有类型总共只有一个图标被选中
const toggleSelectItem = (type: string, item: Hotel | Spot | Transportation, event?: Event) => {
  // 检查当前点击的图标是否已经选中
  let isCurrentlySelected = false
  let itemId: number = 0

  if (type === 'hotel') {
    const hotel = item as Hotel
    itemId = hotel.id
    isCurrentlySelected = selectedPlan.value.hotels.some((h) => h.id === hotel.id)
  } else if (type === 'spot') {
    const spot = item as Spot
    itemId = spot.id
    isCurrentlySelected = selectedPlan.value.spots.some((s) => s.id === spot.id)
  } else if (type === 'transportation') {
    const transport = item as Transportation
    itemId = transport.id
    isCurrentlySelected = selectedPlan.value.transportations.some((t) => t.id === transport.id)
  }

  // 如果当前图标已经选中，则取消选中
  if (isCurrentlySelected) {
    // 清空所有选中状态，并使用nextTick确保DOM更新
    selectedPlan.value.hotels = []
    selectedPlan.value.spots = []
    selectedPlan.value.transportations = []

    // 使用nextTick确保DOM已更新
    nextTick(() => {
      // 此时DOM已更新，所有marker-selected类都已被移除
      console.log('已取消选中:', type, itemId)
    })

    // 阻止事件冒泡，防止触发地图的点击事件
    event?.stopPropagation()
    return
  }

  // 先清空所有选中状态
  selectedPlan.value.hotels = []
  selectedPlan.value.spots = []
  selectedPlan.value.transportations = []

  // 选中当前点击的图标
  if (type === 'hotel') {
    selectedPlan.value.hotels = [item as Hotel]
  } else if (type === 'spot') {
    selectedPlan.value.spots = [item as Spot]
  } else if (type === 'transportation') {
    selectedPlan.value.transportations = [item as Transportation]
  }

  // 使用nextTick确保DOM已更新
  nextTick(() => {
    console.log('已选中:', type, itemId)
  })

  // 阻止事件冒泡，防止触发地图的点击事件
  event?.stopPropagation()
}

// 保存计划
const savePlan = () => {
  // 这里可以实现将计划保存到本地存储或发送到服务器
  alert('计划已保存！')
  console.log('保存的计划:', JSON.stringify(selectedPlan.value))
}

// 清空计划
const clearPlan = () => {
  selectedPlan.value.hotels = []
  selectedPlan.value.spots = []
  selectedPlan.value.transportations = []
}

// 清除地图上所有标记的选中状态
const clearSelection = () => {
  selectedPlan.value.hotels = []
  selectedPlan.value.spots = []
  selectedPlan.value.transportations = []
}
const currentUid = ref(0)
//右侧清单处理
interface listInfo {
  uid:number
  name: string
  type: string
  id: number
  location: string
}

const selectedInfo = ref({
  listInfo: {
    uid:0,
    name:'',
    type: '',
    id: 0,
    location: '',
  },
})

const travelInfoList = ref<listInfo[]>([])
const startDraggingSpotIcon = (spot:Spot,ev:events.Event) => {
  selectedInfo.value.listInfo = {
    uid:++currentUid.value,
    name: spot.name,
    type: 'spot',
    id: spot.id,
    location: spot.location,
  }
  drag(ev,selectedInfo.value.listInfo)
  //console.log('startDragging')
}

const startDraggingHotelIcon = (hotel:Hotel,ev:events.Event) => {
  selectedInfo.value.listInfo = {
    uid:++currentUid.value,
    name: hotel.name,
    type: 'hotel',
    id: hotel.id,
    location: hotel.location,
  }
  drag(ev,selectedInfo.value.listInfo)
  //console.log('startDragging')
}

const startDraggingTransportationIcon = (transportation:Transportation,ev:events.Event) => {
  selectedInfo.value.listInfo = {
    uid:++currentUid.value,
    name: transportation.type,
    type: 'transportation',
    id: transportation.id,
    location: transportation.route,
  }
  drag(ev,selectedInfo.value.listInfo)
  //console.log('startDragging')
}
const drag=(ev:events.Event,selectedInfo:listInfo)=> {
  //console.log('dragging:'+selectedInfo.name+' '+selectedInfo.type+' '+selectedInfo.id+' '+selectedInfo.location);
}
const startDraggingListInfo= (item:listInfo,ev:events.Event) => {
  selectedInfo.value.listInfo = {
    uid:item.uid,
    name: item.name,
    type: item.type,
    id: item.id,
    location: item.location,
  }
  drag(ev,selectedInfo.value.listInfo)
  //console.log('startDragging')
}

const clearInfo = () => {
  selectedInfo.value.listInfo = {
    uid:0,
    name:'',
    type: '',
    id: 0,
    location: '',
  }
}
function onDrop(ev:events.Event ) {
  ev.preventDefault();
  var data=selectedInfo.value.listInfo;
  //console.log('getData:'+data);
  travelInfoList.value.push(data);
  /*travelInfoList.value.forEach((item => {
    console.log('item:', item);
  }))*/
  selectedInfo.value.listInfo = {
    uid:0,
    name:'',
    type: '',
    id: 0,
    location: '',
  }
  //ev.target.appendChild(document.getElementById(data));
}
const onListDrop=(item:listInfo,ev:events.Event) => {
  ev.preventDefault();
  ev.stopPropagation();
  //console.log('onListDrop');
  let index=travelInfoList.value.indexOf(item)
  travelInfoList.value.splice(index, 0, selectedInfo.value.listInfo);

  //console.log('onListDrop');
}
function onDragLeave(ev:events.Event) {
  ev.preventDefault();
  //console.log(selectedInfo.value.listInfo);
  travelInfoList.value = travelInfoList.value.filter(x => x.uid !== selectedInfo.value.listInfo.uid);
  //console.log('onDragLeave');
}
function allowDrop(ev:events.Event)
{
  ev.preventDefault();
}
</script>

<template>
  <div class="home">
    <!-- 左栏：地图上显示景点、酒店和交通 -->
    <div class="left-panel">
      <!-- 左上角的折叠按钮控件 -->
      <div class="control-panel">
        <FoldingButton @filter-change="handleFilterChange" />
      </div>

      <!-- 地图容器 -->
      <div class="map-container" @click="clearSelection">
        <!-- 地图背景 -->
        <div class="map-background"></div>

        <!-- 酒店标记 -->
        <div
          v-if="showHotels"
          v-for="hotel in hotels"
          :key="hotel.id"
          class="map-marker hotel-marker"
          :style="{ left: hotel.position.x + '%', top: hotel.position.y + '%' }"
          :class="{ 'marker-selected': isItemSelected('hotel', hotel.id) }"
          @click.stop="toggleSelectItem('hotel', hotel, $event)"
          draggable="true"
          @dragstart ="startDraggingHotelIcon(hotel,$event)"
        >
          <div class="marker-icon hotel-icon">
            <IconHotel />
          </div>
          <div class="marker-tooltip" :class="{ 'tooltip-left': hotel.position.x > 70 }">
            <h3>{{ hotel.name }}</h3>
            <p>位置: {{ hotel.location }}</p>
            <p>价格: {{ hotel.price }}</p>
          </div>
        </div>

        <!-- 景点标记 -->
        <div
          v-if="showSpots"
          v-for="spot in spots"
          :key="spot.id"
          class="map-marker spot-marker"
          :style="{ left: spot.position.x + '%', top: spot.position.y + '%' }"
          :class="{ 'marker-selected': isItemSelected('spot', spot.id) }"
          draggable="true"
          @click.stop="toggleSelectItem('spot', spot, $event)"
          @dragstart ="startDraggingSpotIcon(spot,$event)"
        >
          <div class="marker-icon spot-icon">
            <IconSpot/>
          </div>
          <div
            class="marker-tooltip"
            :class="{ 'tooltip-left': spot.position.x > 70, 'tooltip-top': spot.position.y > 70 }"
          >
            <h3>{{ spot.name }}</h3>
            <p>位置: {{ spot.location }}</p>
            <p>评分: {{ spot.rating }}/5</p>
          </div>
        </div>

        <!-- 交通标记 -->
        <div
          v-if="showTransportation"
          v-for="transport in transportations"
          :key="transport.id"
          class="map-marker transportation-marker"
          :style="{ left: transport.position.x + '%', top: transport.position.y + '%' }"
          :class="{ 'marker-selected': isItemSelected('transportation', transport.id) }"
          @click.stop="toggleSelectItem('transportation', transport, $event)"
          draggable="true"
          @dragstart ="startDraggingTransportationIcon(transport,$event)"
        >
          <div class="marker-icon transportation-icon">
            <IconTransportation />
          </div>
          <div
            class="marker-tooltip"
            :class="{
              'tooltip-left': transport.position.x > 70,
              'tooltip-top': transport.position.y > 70,
            }"
          >
            <h3>{{ transport.type }} - {{ transport.route }}</h3>
            <p>频率: {{ transport.frequency }}</p>
          </div>
        </div>
      </div>
    </div>

    <!-- 右栏：旅游规划清单（暂时不实现具体功能） -->
    <div class="right-panel" @drop="onDrop" @dragover="allowDrop" @dragleave="onDragLeave">
      <div class="plan-header">
        <h2>旅游清单</h2>
      </div>
      <div class="plan-content" >
        <div v-for="item in travelInfoList">
          <TravelList
            :name="item.name" :type="item.type" :location="item.type" :id="item.id"
            draggable="true"
            @dragstart="startDraggingListInfo(item, $event)"
            @drop="onListDrop(item,$event)"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.home {
  width: 100%;
  height: 100vh;
  display: flex;
  background-color: #f5f7fa;
  color: #333;
  overflow: hidden;
}

/* 左侧地图面板样式 */
.left-panel {
  flex: 3;
  position: relative;
  overflow: hidden;
}

.control-panel {
  position: absolute;
  top: 16px;
  left: 16px;
  z-index: 100;
}

.map-container {
  position: relative;
  width: 100%;
  height: 100%;
}

.map-background {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image: url('background.jpg');
  background-size: cover;
  background-position: center;
  opacity: 0.8;
}

/* 地图标记样式 */
.map-marker {
  position: absolute;
  transform: translate(-50%, -50%);
  cursor: pointer;
  z-index: 10;
  /* 增加鼠标区域，提升用户体验 */
  padding: 5px;
}

.marker-icon {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.3);
  transition: all 0.3s ease;
  background-color: white;
  padding: 5px;
  border: 2px solid transparent;
}

.marker-icon svg {
  width: 100%;
  height: 100%;
}

.hotel-icon {
  border-color: #3399ff;
}

.spot-icon {
  border-color: #18cc73;
}

.transportation-icon {
  border-color: #000000;
}

.marker-selected .marker-icon {
  transform: scale(1.2);
  box-shadow: 0 3px 8px rgba(0, 0, 0, 0.4);
  background-color: #f0fff0;
}

.map-marker {
  transition: transform 0.3s ease;
}

.map-marker:hover {
  z-index: 20; /* 悬停时提高z-index */
}

.marker-selected {
  z-index: 25;
}

/* 标记工具提示 */
.marker-tooltip {
  position: absolute;
  top: 110%;
  left: 50%;
  transform: translateX(-50%);
  width: 150px;
  padding: 10px;
  background-color: white;
  border-radius: 6px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.3s;
  z-index: 30;
  pointer-events: none; /* 默认情况下不接受鼠标事件 */
}

/* 针对靠边缘的标记调整提示框位置 */
.tooltip-left {
  left: -50%;
  transform: translateX(0);
}

.tooltip-top {
  top: auto;
  bottom: 110%;
}

/* 只有在悬停或标记被选中时才显示tooltip */
.map-marker:hover .marker-tooltip,
.marker-selected .marker-tooltip {
  opacity: 1;
  visibility: visible;
  pointer-events: auto; /* 在显示时允许鼠标交互 */
}

/* 确保即使鼠标移动到tooltip上也能保持显示状态 */
.marker-tooltip:hover {
  opacity: 1;
  visibility: visible;
}

.marker-tooltip h3 {
  margin: 0 0 8px;
  font-size: 14px;
  color: #4caf50;
}

.marker-tooltip p {
  margin: 4px 0;
  font-size: 12px;
  color: #555;
}



/* 适配不同屏幕尺寸 */
@media (max-width: 768px) {
  .home {
    flex-direction: column;
  }
}

.draggable {
  position: absolute;
  cursor: move;
  border: 1px solid #ccc;
  padding: 10px;
  background-color: #f0f0f0;
}

/* 右侧面板样式 */
.right-panel {
  flex: 1;
  background-color: white;
  box-shadow: -2px 0 10px rgba(0, 0, 0, 0.1);
  overflow-y: auto;
}

.plan-header {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
  background-color: #4caf50;
  color: white;
}

.plan-header h2 {
  margin: 0;
  font-size: 1.3em;
}

.plan-content {
  flex-direction: column; /* Stack items vertically */
  align-items: center;
  justify-content: center;
  min-height: 80%;
}

.empty-plan-message {
  color: #999;
  font-style: italic;
  text-align: center;
  padding: 20px;
}
/* 右侧面板样式 */
.right-panel {
  flex: 1;
  background-color: white;
  box-shadow: -2px 0 10px rgba(0, 0, 0, 0.1);
  overflow-y: auto;
}

.plan-header {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
  background-color: #4caf50;
  color: white;
}

.plan-header h2 {
  margin: 0;
  font-size: 1.3em;
}

.empty-plan-message {
  color: #999;
  font-style: italic;
  text-align: center;
  padding: 20px;
}

</style>
