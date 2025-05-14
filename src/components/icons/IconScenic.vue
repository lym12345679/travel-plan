<script>
defineProps({
  name: {
    type: String,
    default: 'IconScenic'
  }
});
export default {
  data() {
    return {
      dragging: false,  // 是否正在拖拽
      offsetX: 0,  // 鼠标按下时距离元素左上角的偏移
      offsetY: 0,  // 鼠标按下时距禋元素左上角的偏移
      posX: 0,  // 元素左上角相对于父元素的位置
      posY: 0   // 元素左上角相对于父元素的位置
    };
  },
  methods: {
    startDragging(e) {
      this.dragging = true;
      this.offsetX = e.clientX - this.posX;
      this.offsetY = e.clientY - this.posY;
    },
    dragging(e) {
      if (this.dragging) {
        this.posX = e.clientX - this.offsetX;
        this.posY = e.clientY - this.offsetY;
      }
    },
    stopDragging() {
      this.dragging = false;
    }
  }
};
</script>


<template>
  <div
    class="draggable"
    :style="{ top: posY + 'px', left: posX + 'px' }"
    @mousedown="startDragging"
    @mousemove="dragging"
    @mouseup="stopDragging"
  >
    Drag me!
  </div>
</template>


<style scoped>
.draggable {
  position: absolute;
  cursor: move;
  border: 1px solid #ccc;
  padding: 10px;
  background-color: #f0f0f0;
}
</style>
