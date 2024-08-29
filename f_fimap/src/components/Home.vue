<template>
  
  <div>
    <el-container class="Main_interface">

      <el-aside :class="{ hidden: !showAside }">

        <el-container class="Logo_container">
          <img src="/Plane_icon.svg" alt="Logo" class="logo">
          <span class="logo-text-content">Flight Information MAP</span>
          
          <span>V0.1  2024.08</span>
        </el-container>

        <el-container class="Help_container">

          <el-icon color="#409efc" class="no-inherit">
            <InfoFilled />
          </el-icon>
          <span class="help-text" @click="openMapPage">地图说明</span>

          <el-icon color="#409efc" class="no-inherit">
            <QuestionFilled />
          </el-icon>
          <span class="help-text" @click="openHelpPage">使用帮助</span>

          <el-icon color="#409efc" class="no-inherit">
            <Connection />
          </el-icon>
          <span class="help-text" @click="openOurPage">开发人员</span>
        </el-container>

        <el-container class="Search_container">
          
          <img src="/weather.svg" width="40"height='35'>

          <el-select v-model="value" clearable placeholder="实时气象图层" style="width: 240px">
            <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" />
          </el-select>
          
          <el-select v-model="selectedMap" placeholder="3D/2D" style="width: 240px; ">
                    <el-option label="2D地图" value="leaflet"></el-option>
                    <el-option label="3D地图" value="cesium"></el-option>
                </el-select>

                


        </el-container >

        <el-container class="Information_container">
          <div><information /></div>
          
        </el-container>

      </el-aside>

      <el-main>

        <el-switch v-model="showAside" :active-action-icon="View" :inactive-action-icon="Hide" />

        <div>
          <component :is="currentMapComponent" :current-layer="value" :number="test"  />
          
        </div>
      </el-main>

    </el-container>
  </div>
  
</template>

<script setup>
import { ref, computed } from 'vue';
import Leaflet_Map from './Leaflet_Map.vue';
import Cesium_Map from './Cesium_Map.vue';
import { Hide, View } from '@element-plus/icons-vue'
import information from './infomation.vue'


//切换地图
const selectedMap = ref('leaflet');
const currentMapComponent = computed(() => {
    return selectedMap.value === 'leaflet' ? Leaflet_Map : Cesium_Map;
});


const test = ref('1111');
//天气控件
const value = ref('raincls')
const options = [
{
    value: 'cloudscls',
    label: '云层',
  },
  {
    value: 'raincls',
    label: '降雨',
  },
  {
    value: 'wind',
    label: '风速',
  },
  {
    value:'pressurecntr',
    label:'气压',
  }
  
]


//侧边栏
const showAside = ref(true);

function openMapPage() {
      // 使用 window.open 打开新的网页
      const helpPageUrl = './src/components/Map.html'; // 替换为你的 HTML 文件路径
      window.open(helpPageUrl, '_blank'); // '_blank' 表示在新标签页打开
}

function openHelpPage() {
      // 使用 window.open 打开新的网页
      const helpPageUrl = './src/components/Help.html'; // 替换为你的 HTML 文件路径
      window.open(helpPageUrl, '_blank'); // '_blank' 表示在新标签页打开
}

function openOurPage() {
      // 使用 window.open 打开新的网页
      
      const helpPageUrl = './src/components/resume.html';
      window.open(helpPageUrl, '_blank'); // '_blank' 表示在新标签页打开
}

</script>

<style scoped>
.Main_interface {

  background-color: #003663;
  color: #f9f4f4;
  /* 其他定位样式 */
  box-shadow: 0px -1px 1px 1px #00254b;
  height: 97.8vh;
  padding-right: 10px;
  padding-left: 10px;
  width: 100%;
  position: relative;
  display: flex
    /* 垂直居中 Logo 和文字 
  /* 默认值，但明确设置可以增加可读性 */
}

.Main_interface .el-aside {
  margin-right: 15px;
  background-color: #c6d5e1;
  width: 25%;
  height: 100%;
  position: relative;
  
}

.Main_interface .el-main {
  width: 73%;
  height: 100%;
  background-color: #e6ecf1;
  top: 0;
  right: 0;
  padding: 5px;
  overflow: hidden;
  /* 隐藏任何溢出的内容 */
  position: relative;
  /* 确保 .el-main 有定位 */
}

.Main_interface .el-main .el-switch {
  position: absolute;
  top: 10px;
  /* 距离顶部的距离 */
  right: 10px;
  /* 距离右侧的距离 */
  z-index: 1000;
  /* 设置一个较高的 z-index 值 */
}

.hidden {
  display: none;
}

.Logo_container {
  max-width: 100%;
  position: relative;
  padding-bottom: 20px;
  height: 15%;

}

.logo {

  position: relative;
  top: 3px;
  width: 6em;
  height: 6em;
  margin-right: 8px;
}

.logo-text-content {
  font-size: 1.9em;
  /* 调整文字大小 */
  font-weight: bold;
  /* 使文字加粗 */
  font-family: Arial, sans-serif;
}

.Help_container {
  max-width: 100%;
  justify-content: center; /* 水平居中对齐 */
  align-items: center; 
}

.help-text {
  font-size: 1em;
  padding-right: 15px;

}

.Search_container {
  background-color: #08a37f;
  height: 17%;
}

.Information_container {
  position: absolute;
  /* 绝对定位 */
  bottom: 0;
  /* 贴紧底部 */
  left: 0;
  /* 贴紧左侧 */
  width: 100%;
  /* 占据 100% 的宽度 */
  height: 65%;
  /* 占据 60% 的高度 */
  background-color: #04637e;
  /* 示例背景颜色 */
}
</style>
