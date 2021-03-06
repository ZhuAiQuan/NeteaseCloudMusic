<template>
  <div class="album-newest">
    <div class="warp">
      <div class="content">
        <ul class="list" ref="listRef" :style="listStyle">
          <li
            class="item"
            v-for="(item, index) in listData"
            :key="index"
            ref="liRef"
          >
            <div class="item-img">
              <img class="img" :src="`${item.picUrl}?param=100y100`" alt="" />
              <i class="icon"></i>
            </div>
            <div class="title" :title="item?.name">{{ item?.name }}</div>
            <div class="name">{{ item?.artist?.name }}</div>
          </li>
        </ul>
      </div>
      <button class="btn left-btn" @click="albumPrev"></button>
      <button class="btn right-btn" @click="albumNext"></button>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, reactive, computed } from 'vue';
import { throttle } from 'lodash';
import { albumNewest } from '@api/home';
import { ResponseType } from '@/types/types';

interface ListOffest {
  index: number;
  transform: number;
  duration: number;
}

export default defineComponent({
  setup() {
    // 新碟上架数据
    const listData = ref<unknown[]>([]);
    function getListData() {
      albumNewest().then((res: ResponseType) => {
        if (res.code === 200) {
          // 截取前十项，并复制为二倍模板
          listData.value = res?.albums.slice(0, 10);
          listData.value.unshift(...listData.value);
        }
      });
    }
    getListData();

    // 列表偏移样式
    const listOffest = reactive<ListOffest>({
      index: 0,
      transform: 0,
      duration: 0
    });

    const listStyle = computed(() => {
      return {
        transform: `translate3d(-${listOffest.transform}px, 0, 0)`,
        transition: `${listOffest.duration}s`
      };
    });

    const listRef = ref<HTMLElement>();
    const liRef = ref<HTMLElement>();

    // 上一轮
    const albumPrev = throttle(
      function() {
        const li = liRef.value as HTMLElement;
        if (listOffest.index == 0) {
          // 瞬间返回二倍模板，并在短暂延迟后执行动画
          listOffest.index = 2;
          listOffest.duration = 0;
          setTimeout(() => {
            listOffest.index--;
            listOffest.duration = 1.5;
            listOffest.transform = li.clientWidth * listOffest.index * 5;
          });
        } else {
          listOffest.index--;
          listOffest.duration = 1.5;
        }
        listOffest.transform = li.clientWidth * listOffest.index * 5;
      },
      1500,
      {
        leading: true, // 点击第一下是否执行
        trailing: false // 节流时间内，多次点击，节流结束后，是否执行一次
      }
    );

    // 下一轮
    const albumNext = throttle(
      function() {
        const li = liRef.value as HTMLElement;
        // 瞬间返回二倍模板，并在短暂延迟后执行动画
        if (listOffest.index == 2) {
          listOffest.index = 0;
          listOffest.duration = 0;
          setTimeout(() => {
            listOffest.index++;
            listOffest.duration = 1.5;
            listOffest.transform = li.clientWidth * listOffest.index * 5;
          });
        } else {
          listOffest.index++;
          listOffest.duration = 1.5;
        }
        listOffest.transform = li.clientWidth * listOffest.index * 5;
      },
      1500,
      {
        leading: true, // 点击第一下是否执行
        trailing: false // 节流时间内，多次点击，节流结束后，是否执行一次
      }
    );
    return {
      listData,
      listRef,
      liRef,
      listStyle,
      albumPrev,
      albumNext
    };
  }
});
</script>

<style lang="less" scoped>
@import './album-newest.less';
</style>
