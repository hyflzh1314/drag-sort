<template>
  <div>
    <h1>拖拽排序，自定义拖拽内容，可增删</h1>
    <div>
      <button @click="add">增加图片</button>
      <button @click="done">获取最新数组</button>
    </div>
    <drag-sort
      v-model:itemList="itemList"
      :col="3"
      :itemWidth="150"
      :itemHeight="170"
      :margin="10"
      v-model:isDrag="isDrag"
    >
      <template v-slot:default="slot">
        <div class="drag-item">
          <span class="del" @click="del(slot.item)">x</span>
          <img :src="slot.item.url" @click="imgClick($event)" />
          <p @click="pClick(slot.item.id)">{{ slot.item.id }}</p>
        </div>
      </template>
    </drag-sort>

    <div style="height: 400px"></div>
  </div>
</template>

<script>
import { reactive, ref, watch } from "vue";
import DragSort from "@component/DragSort.vue";
export default {
  components: {
    DragSort,
  },
  setup(props) {
    const data = [
      {
        url:
          "http://img.zhuge.com/wolongyun/61b63259bd7a0e1ee22ad5b9d6ac53e6.jpeg",
        id: 1,
      },
      {
        url:
          "http://img.zgsta.zhuge.com/193-2-1-2-102/c0a337073a8707234bf27736c3d30611_addfinger.png",
        id: 2,
      },
      {
        url:
          "http://img.zgsta.zhuge.com/193-2-1-4-102/3f68538de26a804267b08931781dcc33_addfinger.png",
        id: 3,
      },
      {
        url:
          "http://img.zgsta.zhuge.com/193-2-1-4-102/e40b8ae50ee1089d710eea19a9d6ebc2_addfinger.png",
        id: 4,
      },
      {
        url:
          "http://img.zgsta.zhuge.com/193-2-1-7-102/97d9067230e8e127053841073cea73e4_addfinger.png",
        id: 5,
      },
    ];
    const state = reactive({ count: 5 });
    const isDrag = ref(false);
    const itemList = reactive(data);
    const add = () => {
      state.count++;
      itemList.push({
        url:
          "http://img.zgsta.zhuge.com/193-2-1-7-102/97d9067230e8e127053841073cea73e4_addfinger.png",
        id: state.count,
      });
    };
    Array.prototype.remove = function (item) {
      let index = this.indexOf(item);
      if (index > -1) {
        this.splice(index, 1);
      }
    };
    const del = (item) => {
      console.log(item);
      itemList.remove(item);
    };

    const done = () => {
      console.log(itemList);
      console.log(isDrag);
    };

    const imgClick = event => {
      if (isDrag.value) {
        isDrag.value = false;
        return false
      } 
      console.log(event.target.src);
    };
    const pClick = id => {
      alert(id)
    }
    return {
      state,
      itemList,
      add,
      del,
      done,
      imgClick,
      isDrag,
      pClick
    };
  },
};
</script>

<style scoped>
a {
  color: #42b983;
}

.drag-item {
  width: 150px;
  height: auto;
  position: relative;
}
.drag-item .del {
  position: absolute;
  right: -10px;
  top: -10px;
  width: 20px;
  height: 20px;
  text-align: center;
  line-height: 18px;
  background-color: #ddd;
  border-radius: 50%;
  cursor: pointer;
  color: #999;
}
.drag-item img {
  width: 100%;
  height: 150px;
}
.drag-item p {
  line-height: 20px;
  background: #eee;
  margin: 0;
}
</style>