
<template>
 
  <el-card class="voca-card" :class="{ pc: isPC() }" v-loading.fullscreen.lock="fullscreenLoading"
    element-loading-text="应用正在加载中..." element-loading-background="#eee" style="display: inline-block; height:800px;width:500px; overflow: auto;  " >
    <template #header>
      <div class="voca-card-head">
        <div class="voca-card-name">
          <span class="voca-card-name--inner" contenteditable="true" @blur="getSearchText($event, bookItem)">{{
            bookItem?.n }}</span>
          <el-button :icon="RefreshLeft" round plain type="success" v-show="bookItemBeforeSearch"
            @click="restoreBookItem">还原</el-button>
        </div>

        <el-button :icon="Setting" plain type="primary" circle class="voca-card-handle" @click="OpenSetting"></el-button>
      </div>
    </template>
    <div class="voca-card-body" @dblclick="getWordItem">
      <div class="voca-card-item" :class="{ 'toggle-by-btn': basicData.toggleWordWay === 'button' }">
        <ConciseWord v-if="basicData?.showMode === 'concise'" :basic-data="basicData" :book-item="bookItem" />
        <WordCom v-else :basic-data="basicData" :book-item="bookItem" />
      </div>
      <div v-if="basicData.toggleWordWay === 'button'" class="voca-card-toggle-btn">
        <el-button @click="getWordItemByBtn('left')">上一个</el-button>
        <el-button type="primary" plain @click="getWordItemByBtn('right')">下一个</el-button>

      </div>
    </div>
    
  </el-card>


  <SettingCom  :class="{ pc: isPC() }" :visible="drawer" @handleDrawer="handleDrawer" />


  
</template>

<script setup>
import {
  ref,
  reactive,
  watchEffect,
  watch,
  onMounted,
  onUnmounted,
  toRaw,
} from "vue";
import { ElNotification, ElMessage, ElLoading } from "element-plus";
import { Setting, RefreshLeft } from "@element-plus/icons-vue";
import { storeToRefs } from "pinia";

import { useWordItem } from "../../hooks/useWordItem";

import SettingCom from "../setting/index.vue";
import WordCom from "../../components/word.vue";
import ConciseWord from "../../components/conciseWord.vue";

import { useBookStore } from "../../stores/books";

import { setNotify } from "../../utils/element-plus";
import { isPC } from "../../utils/common";

// mounted函数必须在异步调用之前，不然不生效
onMounted(() => {
  window.addEventListener("keyup", arrowRightGetData);
});

onUnmounted(() => {
  window.removeEventListener("keyup", arrowRightGetData);
});

// 今日学习数据

let useBook = useBookStore();

let { basicData } = storeToRefs(useBook);

let {
  bookItem,
  bookItemBeforeSearch,
  fullscreenLoading,
  drawer,
  handleDrawer,
  getDataTest,
  getSearchText,
} = useWordItem();

function OpenSetting() {
  drawer.value = true;
  bookItemBeforeSearch.value = null;
}

function restoreBookItem() {
  bookItem.value = toRaw(bookItemBeforeSearch.value);
  bookItemBeforeSearch.value = null;
}

function getWordItem(e) {
  if (bookItemBeforeSearch.value) {
    setNotify("点击单词还原按钮后，再去切换单词");
    return false;
  }
  if (basicData.value.toggleWordWay === 'button') {
    return false
  }
  let windowClientY = document.body.clientHeight;
  let windowClientX = document.body.clientWidth;
  let hasGet = windowClientX < e.clientX * 2;
  if (!hasGet) {
    getDataTest(false);
    return false;
  }
  getDataTest();
}

function getWordItemByBtn(direction) {
  if (
    bookItemBeforeSearch.value
  ) {
    setNotify("点击单词还原按钮后，再去切换单词");
    return false;
  }
  if (basicData.value.toggleWordWay !== 'button') {
    return false
  }
  if (direction === "right") {
    getDataTest();
  }
  if (direction === "left") {
    getDataTest(false);
  }
}

function arrowRightGetData(e) {
  if (
    bookItemBeforeSearch.value &&
    ["ArrowRight", "ArrowLeft"].includes(e.code)
  ) {
    setNotify("点击单词还原按钮后，再去切换单词");
    return false;
  }
  if (e.code === "ArrowRight") {
    getDataTest();
  }
  if (e.code === "ArrowLeft") {
    getDataTest(false);
  }
}
</script>

<style scoped lang="scss">
.pc {
  max-width: 520px;
}

.voca {
  &-card {
    height: 100%;
    margin: 0 auto;
    width: 50%;
    :deep(.el-card__header) {
      background: #ffffff;
      color: rgb(39, 67, 141);
      padding: 10px 20px;
    }

    :deep(.el-card__body) {
      height: calc(100% - 86px);
      overflow: auto;
      border: solid rgb(39, 67, 141);
      border-width: 5px 2px 2px;
      padding: 0px;
      margin: 10px 20px;
    }

    &-name {
      font-size: 2em;
      display: inline-block;
    }

    &-name--inner {
      padding-right: 12px;
      display: inline-block;
      font-weight: 900;
    }

    &-head {
      display: inline-flex;
      flex-flow: row;
      justify-content: space-between;
      align-items: center;
      width: 100%;
    }

    &-handle {
      text-align: right;
      background-color: #ffffff;
      border-color: rgb(39, 67, 141);
      color: rgb(39, 67, 141);

      &:hover {
        box-shadow: 0px 0px 12px rgba(0, 0, 0, 0.12);
        color: #ffffff;
        border-color: rgb(39, 67, 141);
        background: rgb(39, 67, 141);
      }
    }

    &-body {
      height: 100%;
      overflow: hidden;

      .left,
      .right {
        width: 100%;
        height: 100%;
      }
    }

    &-item {
      height: 100%;
      width: 100%;
      overflow: auto;
      margin: 6px 0;
      padding: 0 6px;
      &.toggle-by-btn {
        height: calc(100% - 3em - 12px);
      }
    }

    &-toggle-btn {
      height: 3em;
      width: 100%;
      display: inline-flex;
      flex-wrap: nowrap;
      justify-content: center;
      align-items: center;
      .el-button {
        flex: 1;
        border-radius: unset;
        height: 100%;

        &:last-child {
          background: rgb(39, 67, 141);
          color: white;
          transition: all 0.2s;

          &:hover {
            background: rgb(39, 67, 141);
            color: rgb(39, 67, 141);
          }
        }

      }
      .el-button+.el-button {
        margin-left: 0;
      }
    }
  }
}</style>
