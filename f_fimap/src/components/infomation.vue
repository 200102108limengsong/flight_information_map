<template>
  <div>
     
      <el-row>
        
      <el-col :span="24">
        <el-table :data="flightData"  stripe border >
          <el-table-column v-for="(column, index) in columns" :key="index" :prop="column.prop" :label="column.label" :width="column.width" />
        </el-table>
      </el-col>
    </el-row>
    
    
    
    
  </div>
</template>

<script setup>
import { onMounted, ref,watch } from 'vue';

const props = defineProps({
  airinformationdata: {
      type: Object,
      required: true,
    },
});
watch(() => props.airinformationdata, (newVal) => {
  console.log('Received new data:', newVal);
  // 在这里你可以访问到传递过来的 filteredAircraft 数据
  // 根据需要进行更新或处理
});
// 初始化信息展示数据
const flightData = ref([]);
const columns = ref([
  
  { prop: 'callsign', label: '呼号' },
  { prop: 'origin_country', label: '国家'},
  { prop: 'longitude', label: '经度' },
  { prop: 'latitude', label: '维度'},
  { prop: 'baro_altitude', label: '高度', },
  { prop: 'velocity', label: '速度'},
  { prop: 'true_track', label: '方向'},
  { prop: 'vertical_rate', label: '垂直速度'},
]);

// 在组件加载时获取数据
onMounted(() => {
  fetchData();
 
});

// 数据获取函数
async function fetchData() {
  /*try {
    const response = await fetch('http://localhost:8000/api/planesnow.geojson/?format=json');
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();

    // 只保留前50条记录
    //const limitedData = data.features.slice(0, 50).map(feature => feature.properties);

    // 直接将每个特征的 properties 对象添加到 flightData
    flightData.value = data.map(feature => feature.properties);
  } catch (error) {
    console.error('Error fetching data:', error);
  }*/

  

}
</script>

<style scoped>
/* 你可以在这里添加特定的样式 */

.el-table {
  font-size: 12px; /* 你可以根据需要调整这个值 */
  max-width: 373px;
  overflow-x: auto; /* 当内容超出最大宽度时显示水平滚动条 */
  height: 475px;
}

/* 如果你想要进一步调整头部样式 */
.el-table th div {
  font-weight: bold; /* 头部字体加粗 */
}
/* 如果你想要调整单元格的内边距 */
.el-table td,
.el-table th {
  padding: 8px 0; /* 你可以根据需要调整这个值 */
  
}
.el-card{
  font-size: 20px;
  font-weight: bold;
}
</style>