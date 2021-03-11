<template>
  <div>
    <div
      class="drag-container"
      :style="{ height: `${containerHeight}px`, width: `${containerWidth}px` }"
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
    </div>
  </div>
</template>
<script>
import { ref, reactive, defineComponent, onMounted, watch } from "vue";

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
    setup(props, ctx) {
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
            // 计算父容器宽高
            containerHeight.value = Math.ceil(imgList.length / col) * (itemHeight + margin) + margin;
            containerWidth.value = itemWidth * col + margin * (col + 1);
        });

        const col = props.col;
        const itemWidth = props.w;
        const itemHeight = props.h;
        const margin = props.margin;
        const imgList = props.list;

        const containerHeight = ref(0);
        const containerWidth = ref(0);
        const active = ref(null);
        const isMove = ref(false);
        const isDown = ref(false);
        const isAni = ref(false);
        // 当前拖拽元素
        const dragIndex = ref(0);
        // 拖拽进入区域元素
        const dragEnterIndex = ref(null);
        // 拖拽距离
        const dragMoveDistance = {
            x: 0,
            y: 0,
        };

        
        
        const draw = (index) => {
            // 绘制每个元素的translate偏移位置
            let x = (index % col) * itemWidth + ((index % col) + 1) * margin + "px";
            let y = Math.floor(index / col) * itemHeight + (Math.floor(index / col) + 1) * margin + "px";
            return {
                x,
                y,
            };
        };
        const dragMoveIndexComputed = (x, y) => {
            let xIndex = Math.ceil(x / (itemWidth + margin));
            let Yindex = Math.ceil(y / (itemHeight + margin));
            //拖拽进入区域 索引
            let dragMoveIndex = xIndex + (Yindex - 1) * col;

            return dragMoveIndex > imgList.length ? imgList.length : dragMoveIndex;
        };
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

        const mousePosition = (pageX, pageY) => {
            // 计算鼠标在容器内的坐标
            let x = pageX - wrapEl.offsetLeft;
            let y = pageY - wrapEl.offsetTop;

            return {
                x,
                y
            }
        }

        const dragDown = (e) => {
            let { x, y } = mousePosition(e.pageX, e.pageY);
            dragMoveDistance.x = x;
            dragMoveDistance.y = y;
            active.value = dragMoveIndexComputed(x, y) - 1;
            isDown.value = true;
            dragIndex.value = dragMoveIndexComputed(x, y) - 1;
        };
        const dragMove = (e) => {
            let { x, y } = mousePosition(e.pageX, e.pageY);
            // 判断是否拖拽操作
            if (
            (Math.abs(x - dragMoveDistance.x) > 20 ||
                Math.abs(y - dragMoveDistance.y) > 20) &&
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
                let { x, y } = mousePosition(e.pageX, e.pageY);
                if (
                    x >= 0 &&
                    x < containerWidth.value &&
                    y >= 0 &&
                    y < containerHeight.value
                ) {
                    // 容器范围内
                    dragEnterIndex.value = dragMoveIndexComputed(x, y);
                    isAni.value = true;
                    sort(dragIndex.value, dragEnterIndex.value);
                    ctx.emit('update:list', imgList)
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

        watch(
            imgList,
            (list, prevList) => {
                containerWidth.value = itemWidth * col + margin * (col + 1);
                containerHeight.value = Math.ceil(list.length / col) * (itemHeight + margin) + margin;
            }
        )


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