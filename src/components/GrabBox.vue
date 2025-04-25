<script>
import { useMouse } from '../hooks/mouse.js'
const { x, y } = useMouse()
export default {
  data() {
    return {
      isPressed: false,
      mouseCoords : useMouse()
    };
  },
  methods: {
    changeColor(pressed) {
      this.isPressed = pressed;
    },
  },
};
</script>

<template>
 <div
    @mousedown="changeColor(true)"
    @mouseup="changeColor(false)"
    :style="{ backgroundColor: isPressed ? 'lightblue' : 'white' , position : isPressed ? 'absolute' : relative, bottom :x , left : y }"
    class="box"
 ></div>
</template>

<style scoped>
.box {
  width: 100px;
  height: 100px;
  border-radius: 10px;
  background-color: #f09029;
  
}
.box:hover {
    cursor: pointer;
  } 
</style>