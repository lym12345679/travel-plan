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
  imageUrl: string // 新增图片链接字段
}

interface Spot {
  id: number
  name: string
  location: string
  rating: number
  position: Position
  imageUrl: string // 新增图片链接字段
}

interface Transportation {
  id: number
  type: string
  route: string
  frequency: string
  position: Position
  imageUrl: string // 新增图片链接字段
}


// 模拟数据
const hotels: Hotel[] = [
  { id: 10001, name: '速8酒店(上海松江新城嘉和广场店)', location: '上海市松江区文诚路338弄D区1号楼4层', price: '¥680/晚', position: { x: 25, y: 35 } ,
    imageUrl: 'http://store.is.autonavi.com/showpic/21216203233ff277087519bbf77cdc59?type=pic'},
  { id: 10002, name: '速8酒店(上海浦东机场晨阳路店)', location: '上海市浦东新区川南奉公路1312号', price: '¥520/晚', position: { x: 74, y: 26 } ,
    imageUrl: 'http://store.is.autonavi.com/showpic/a7e0bf71b9845b0713240320df64149b?type=pic'},
  { id: 10003, name: '速8酒店(上海金山城市沙滩店)', location: '上海市金山区金山卫镇学府路551号', price: '¥520/晚', position: { x: 30, y: 60 } ,
    imageUrl: 'http://store.is.autonavi.com/showpic/b3295ca75973cc7a603eb9037c93de18?type=pic'},
  { id: 10004, name: '99优选酒店(上海北外滩延吉中路地铁站店)', location: '上海市杨浦区延吉中路95号248幢', price: '¥520/晚', position: { x: 65, y: 33 } ,
    imageUrl: 'http://store.is.autonavi.com/showpic/0676ebd7c6009b5ed906a00da77a8401?type=pic'},
  { id: 10005, name: '上海君然酒店(秀沿路地铁站店)', location: '上海市浦东新区康桥镇川周公路3251弄9号', price: '¥520/晚', position: { x: 80, y: 70 } ,
    imageUrl: 'http://store.is.autonavi.com/showpic/9e015361b5f79b75abf763cdae9345cb?type=pic'},
  { id: 10006, name: '如家华驿酒店(上海金山城市沙滩店)', location: '上海市金山区象州路128号(停车场内)', price: '¥520/晚', position: { x: 75, y: 68 } ,
    imageUrl: 'http://store.is.autonavi.com/showpic/5554f9b939e2369f188c6dfa99cef5b1?type=pic'},
  { id: 10007, name: '柏友青年酒店', location: '上海市闵行区吴中路1099号2号楼4-5层', price: '¥520/晚', position: { x: 55, y: 17 } ,
    imageUrl: 'http://store.is.autonavi.com/showpic/ec1b61b5827dea874a00961b95bcad87?type=pic'},
  { id: 10008, name: '如家酒店·neo(上海闵行开发区北桥地铁站店)', location: '上海市闵行区沪闵路2660号(北桥地铁站2号口步行270米)', price: '¥520/晚', position: { x: 15, y: 20 } ,
    imageUrl: 'http://store.is.autonavi.com/showpic/1f8a4cf99808badd219c1fbb8e781e6b?type=pic'},
  { id: 10009, name: '99优选酒店(上海虹桥机场沪青平公路店)', location: '上海市闵行区沪青平公路航东路788弄9号', price: '¥520/晚', position: { x: 50, y: 50 } ,
    imageUrl: 'http://store.is.autonavi.com/showpic/7a12e7d9b7ecf461c2149167ba89e2e4?type=pic'},
  { id: 10010, name: '布丁酒店(上海浦东机场店)', location: '上海市浦东新区江镇水闸南路10号', price: '¥520/晚', position: { x: 50, y: 78 } ,
    imageUrl: 'http://store.is.autonavi.com/showpic/e243d19a70ff2da9a2fb5a70e52dc955?type=pic'},
  { id: 10011, name: '格林豪泰智选酒店(上海罗泾店)', location: '上海市宝山区罗泾镇新川沙路518号', price: '¥520/晚', position: { x: 23, y: 82 } ,
    imageUrl: 'http://store.is.autonavi.com/showpic/75cc0546306e9f325b61274ab7df14d7?type=pic'},
]

const spots: Spot[] = [
  { id: 20001, name: '上海迪士尼度假区', location: '上海市浦东新区川沙新镇黄赵路310号(迪士尼地铁站1号口步行360米)', rating: 4.7, position: { x: 70, y: 60 } ,
    imageUrl: 'http://store.is.autonavi.com/showpic/0ed6a4415156ded208b6b8830970191d'},
  { id: 20002, name: '外滩', location: '上海市黄浦区中山东一路49号', rating: 4.7, position: { x: 40, y: 60 } ,
    imageUrl: 'http://store.is.autonavi.com/showpic/b9c402b7d34ea98654cc915e567761dd'},
  { id: 20003, name: '东方明珠广播电视塔', location: '上海市浦东新区世纪大道1号', rating: 4.7, position: { x: 43, y: 27 } ,
    imageUrl: 'http://store.is.autonavi.com/showpic/73078b8c6c35ed97cf0c11c0cb65b32f'},
  { id: 20004, name: '川沙古镇', location: '上海市浦东新区东城壕路8弄1号', rating: 4.5, position: { x: 80, y: 50 } ,
    imageUrl: 'https://aos-comment.amap.com/B0FFKPFGVE/comment/4b01252a7c24468d74a8a9f2b6ccba1d_2048_2048_80.jpg'},
  { id: 20005, name: '上海世纪公园', location: '上海市浦东新区锦绣路1001号', rating: 4.5, position: { x: 50, y: 38 } ,
    imageUrl: 'http://aos-cdn-image.amap.com/sns/ugccomment/cf673112-ca06-4486-8c2d-7939c26b417e.jpg'},
  { id: 20006, name: '武康路历史文化名街', location: '上海市徐汇区上海市湖南路街道武康路与湖南路交叉口', rating: 4.3, position: { x: 30, y: 40 } ,
    imageUrl: 'https://aos-comment.amap.com/B0FFIK0Z21/comment/0137AF31_FBC8_4319_AD94_B8C7EFE9FEA7_L0_001_1512_2016_1738127719706_97726227.jpg'},
  { id: 20007, name: '上海豫园', location: '上海市黄浦区福佑路168号', rating: 4.0, position: { x: 20, y: 80 } ,
    imageUrl: 'http://store.is.autonavi.com/showpic/faeb0264854cd82fbb315cb2ccacea0f'},
  { id: 20008, name: '上海滨江森林公园', location: '上海市浦东新区高桥镇凌桥高沙滩3号', rating: 4.4, position: { x: 77, y: 23 } ,
    imageUrl: 'https://aos-comment.amap.com/B00155FHMD/comment/content_media_external_images_media_1000032730_ss__1733575930263_86880803.jpg'},
  { id: 20009, name: '1933老场坊', location: '上海市虹口区沙泾路10号', rating: 4.5, position: { x: 15, y: 25 } ,
    imageUrl: 'http://store.is.autonavi.com/showpic/6a976ce1765588fe56d1bbc21c1fa0e2'},
  { id: 20010, name: '浦东金海湿地公园', location: '上海市浦东新区曹路镇民唐路198号', rating: 4.2, position: { x: 50, y: 78 } ,
    imageUrl: 'https://aos-comment.amap.com/B00156ECD1/comment/content_media_external_images_media_80219_ss__1741407710503_34240895.jpg'},
  { id: 20011, name: '上海世博园', location: '上海市浦东新区上海市上钢新村街道上南路与博成路交叉口东南约60米', rating: 4.5, position: { x: 85, y: 55 } ,
    imageUrl: 'https://aos-comment.amap.com/B0FFJGWFDS/comment/041f14a66173d4484e90a08c5b0834a3_2048_2048_80.jpg'},
]

const transportations: Transportation[] = [
  { id: 30001, type: '源深体育中心(地铁站)', route: '上海市浦东新区6号线', frequency: '每15分钟一班', position: { x: 40, y: 45 } ,
    imageUrl: 'https://aos-comment.amap.com/BV10039836/comment/B552D39A_282A_4CDA_A955_F18BE084734F_L0_001_1502_2000_1747215646792_76943368.jpg'},
  { id: 30002, type: '杨高中路(地铁站)', route: '上海市浦东新区18号线;9号线', frequency: '每10分钟一班', position: { x: 55, y: 65 } ,
    imageUrl: 'https://aos-comment.amap.com/BV10039884/comment/a7aee2f8284698465406e5645d05799d_2048_2048_80.jpg'},
  { id: 30003, type: '民生路(地铁站)', route: '上海市浦东新区18号线;6号线', frequency: '每20分钟一班', position: { x: 20, y: 22 } ,
    imageUrl: 'https://aos-comment.amap.com/BV10039837/comment/content_media_external_images_media_1000072461_ss__1745998885755_88881053.jpg'},
  { id: 30004, type: '北洋泾路(地铁站)', route: '上海市浦东新区6号线', frequency: '每15分钟一班', position: { x: 17, y: 65 } ,
    imageUrl: 'https://aos-comment.amap.com/BV10039838/comment/42bceb36317794e8a015d3359f29a5f1_2048_2048_80.jpg'},
  { id: 30005, type: '芳甸路(地铁站)', route: '上海市浦东新区9号线', frequency: '每10分钟一班', position: { x: 70, y: 24 } ,
    imageUrl: 'https://aos-comment.amap.com/BV10960994/comment/%5B2024-12-26-06-35-46%5DECD359D3-334D-4362-A486-9B85F7A4DA2_1735166154097_65508737.jpg'},
  { id: 30006, type: '向城路(地铁站)', route: '上海市浦东新区4号线', frequency: '每8分钟一班', position: { x: 77, y: 53 } ,
    imageUrl: 'http://store.is.autonavi.com/showpic/e2b05e45b1f398c6ed616b7231f66716'},
  { id: 30007, type: '张杨路桃林路(公交站)', route: '上海市浦东新区339路;736路;783路;790路;791路;961路;961路(区间);浦东106路', frequency: '每20分钟一班', position: { x: 38, y: 82 } ,
    imageUrl: 'https://aos-comment.amap.com/BV10025028/headerImg/053b6b288568206460ee5698dfbf9f3b_2048_2048_80.jpg'},
  { id: 30008, type: '民生路灵山路(公交站)', route: '上海市浦东新区1024路;130路;181路;609路;638路;794路;公利1号线', frequency: '每15分钟一班', position: { x: 33, y: 32 } ,
    imageUrl: 'https://aos-comment.amap.com/BV10024819/comment/874a08eb68122cdca7aa95baf9303589_2048_2048_80.jpg'},
  { id: 30009, type: '平江路枫林路(公交站)', route: '上海市徐汇区957路', frequency: '每23分钟一班', position: { x: 68, y: 40 } ,
    imageUrl: 'http://store.is.autonavi.com/showpic/b19c51a90db1c3b02b4387f802f0c1e7'},
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
  imgUrl: string
}

const selectedInfo = ref({
  listInfo: {
    uid:0,
    name:'',
    type: '',
    id: 0,
    location: '',
    imgUrl: '',
  },
})

const travelInfoList = ref<listInfo[]>([])

//拖拽功能
const startDraggingSpotIcon = (spot:Spot,ev:events.Event) => {
  selectedInfo.value.listInfo = {
    uid:++currentUid.value,
    name: spot.name,
    type: 'spot',
    id: spot.id,
    location: spot.location,
    imgUrl: spot.imageUrl,
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
    imgUrl: hotel.imageUrl,
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
    imgUrl: transportation.imageUrl,
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
    imgUrl: item.imgUrl,
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
    imgUrl: '',
  }
}
function onDrop(ev:events.Event ) {
  ev.preventDefault();
  var data=selectedInfo.value.listInfo;
  console.log(data);
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
    imgUrl: '',
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
  <div class="home" style="touch-action: none;">
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
            <img :src="hotel.imageUrl" alt="酒店图片" class="tooltip-image" /> <!-- 添加图片展示 -->
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
            <img :src="spot.imageUrl" alt="景点图片" class="tooltip-image" /> <!-- 添加图片展示 -->
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
            <img :src="transport.imageUrl" alt="交通图片" class="tooltip-image" /> <!-- 添加图片展示 -->
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
            :name="item.name" :type="item.type" :location="item.location" :id="item.id" :image-url="item.imgUrl"
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
  position: absolute; /* 确保可以动态更新位置 */
  touch-action: none; /* 禁用默认触摸行为 */
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

.tooltip-image {
  width: 100%;
  height: 100px;
  object-fit: cover;
  border-radius: 4px;
  margin-bottom: 8px;
}

</style>
