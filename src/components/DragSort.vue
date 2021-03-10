<template>
  <div>
    <div
      class="drag-container"
      :style="{ height: `${containerHeight}px` }"
      @mousedown.prevent="dragDown($event)"
      @mousemove.prevent="dragMove($event)"
      @mouseup.prevent="dragUp($event)"
    >
      <div
        v-for="(item, index) in list"
        class="drag-container-item"
        :class="{ active: index == active, ani: isAni }"
        :key="item.id"
        :style="{ transform: `translate(${draw(index).x}, ${draw(index).y}` }"
      >
        <slot :item="item">{{}}</slot>
      </div>
    </div>
    <div>
      <button @click="sort(1, 4)">交换位置</button>
    </div>
  </div>
</template>
<script>
import { ref, reactive, defineComponent, onMounted } from "vue";

export default defineComponent({
  name: "DragSort",
  props: {
    list: {
      type: Object,
    },
    col: {
      type: Number,
      default: 4,
    },
    w: {
      type: Number,
      default: 150,
    },
    h: {
      type: Number,
      default: 150,
    },
    margin: {
      type: Number,
      default: 20,
    },
  },
  setup(props) {
    let wrapEl = null;
    onMounted(() => {
      wrapEl = document.querySelector(".drag-container");
      window.addEventListener("mouseup", () => {
        if (isMove.value) {
          var el = document.querySelectorAll(".drag-container-item");
          var index = dragIndex.value;
          el[index].style.transform =
            "translate(" + draw(index).x + "," + draw(index).y + ")";
          clear();
        }
      });
    });

    const containerHeight = ref(0);
    const containerWidth = ref(0);
    const active = ref(null);
    const isMove = ref(false);
    const isDown = ref(false);
    const isAni = ref(false);
    const dragIndex = ref(0);
    const dragMoveDistance = {
      x: 0,
      y: 0,
    };

    const col = props.col;
    const itemWidth = props.w;
    const itemHeight = props.h;
    const margin = props.margin;
    const imgList = props.list;
    const draw = (index) => {
      let x = (index % col) * itemWidth + ((index % col) + 1) * margin + "px";
      let y =
        Math.floor(index / col) * itemHeight +
        (Math.floor(index / col) + 1) * margin +
        "px";
      return {
        x,
        y,
      };
    };
    const dragEnterIndex = ref(null);
    const dragMoveIndexComputed = (x, y) => {
      let xIndex = Math.ceil(x / (itemWidth + margin));
      let Yindex = Math.ceil(y / (itemHeight + margin));
      //拖拽进入区域 索引
      let dragMoveIndex = xIndex + (Yindex - 1) * col;

      return dragMoveIndex > imgList.length ? imgList.length : dragMoveIndex;
    };
    containerHeight.value =
      Math.ceil(imgList.length / col) * (itemHeight + margin) + margin;
    containerWidth.value = itemWidth * col + margin * (col + 1);
    const sort = (start, end) => {
      let startEl = imgList[start],
        endEl = imgList[end];
      if (start < end) {
        imgList.splice(end, 0, startEl);
        imgList.splice(start, 1);
      } else {
        if (end <= 1) {
          imgList.splice(0, 0, startEl);
        } else {
          imgList.splice(end - 1, 0, startEl);
        }
        imgList.splice(start + 1, 1);
      }
    };
    const dragDown = (e) => {
      let x = e.pageX - wrapEl.offsetLeft;
      let y = e.pageY - wrapEl.offsetTop;
      dragMoveDistance.x = x;
      dragMoveDistance.y = y;
      active.value = dragMoveIndexComputed(x, y) - 1;
      isDown.value = true;
      dragIndex.value = dragMoveIndexComputed(x, y) - 1;
    };
    const dragMove = (e) => {
      let x = e.pageX - wrapEl.offsetLeft;
      let y = e.pageY - wrapEl.offsetTop;
      if (
        (Math.abs(x - dragMoveDistance.x) > 50 ||
          Math.abs(y - dragMoveDistance.y) > 50) &&
        isDown.value
      ) {
        isMove.value = true;
      }
      if (isMove.value) {
        var index = dragIndex.value;
        var el = document.querySelectorAll(".drag-container-item");
        if (
          x >= 0 &&
          x < containerWidth.value &&
          y >= 0 &&
          y < containerHeight.value
        ) {
          // 容器范围内
          var translateX = x - itemWidth / 2 + "px";
          var translateY = y - itemHeight / 2 + "px";
          el[index].style.transform =
            "translate(" + translateX + "," + translateY + ")";
        } else {
        //   el[index].style.transform =
        //     "translate(" + draw(index).x + "," + draw(index).y + ")";
        }

        if (index != dragMoveIndexComputed(x, y) - 1) {
          dragEnterIndex.value = dragMoveIndexComputed(x, y);
        }

        isAni.value = false;
      }
    };
    const dragUp = (e) => {
      if (isMove.value) {
        let x = e.pageX - wrapEl.offsetLeft;
        let y = e.pageY - wrapEl.offsetTop;
        if (
          x >= 0 &&
          x < containerWidth.value &&
          y >= 0 &&
          y < containerHeight.value
        ) {
          // 容器范围内
          dragEnterIndex.value = dragMoveIndexComputed(x, y);
          console.log(dragEnterIndex.value, dragIndex.value);
          isAni.value = true;
          sort(dragIndex.value, dragEnterIndex.value);
        }
      }
      clear();
    };
    const clear = () => {
      // 初始化状态
      dragMoveDistance.x = 0;
      dragMoveDistance.y = 0;
      dragEnterIndex.value = null;
      isDown.value = false;
      isMove.value = false;
      active.value = null;
    };
    return {
      imgList,
      containerHeight,
      containerWidth,
      isMove,
      isDown,
      isAni,
      dragIndex,
      dragMove,
      active,
      dragEnterIndex,
      draw,
      sort,
      dragDown,
      dragMove,
      dragUp,
    };
  },
});
</script>
<style  scoped>
.drag-container {
  position: relative;
  border: 1px solid #ff0000;
  width: 800px;
}
.drag-container-item {
  position: absolute;
  top: 0;
  left: 0;
  /* transition: all .1s; */
  cursor: pointer;
  user-select: none;
  box-sizing: border-box;
}
.drag-container .drag-container-item.active {
  opacity: 0.8;
  z-index: 99;
  border: none !important;
}
.drag-container .drag-container-item.enter {
  opacity: 0.6;
}
.drag-container .drag-container-item.ani {
  transition: all 0.2s;
}
</style>