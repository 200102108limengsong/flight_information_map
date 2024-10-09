<template>
  <div id="chart" style="width: 600px; height: 400px;"></div>
</template>

<script>
import * as echarts from 'echarts';

export default {
  name: 'LineChart',
  data() {
    return {
      // 图表实例
      chartInstance: null,
    };
  },
  mounted() {
    this.initChart();
  },
  beforeDestroy() {
    if (!this.chartInstance) {
      return;
    }
    this.chartInstance.dispose();
    this.chartInstance = null;
  },
  methods: {
    initChart() {
      this.chartInstance = echarts.init(document.getElementById('chart'));

      const option = {
        legend: {},
        toolbox: {
          show: true,
          feature: {
            dataZoom: {
              yAxisIndex: 'none'
            },
            dataView: { readOnly: false },
            magicType: { type: ['line', 'bar'] },
            restore: {},
            saveAsImage: {}
          }
        },
        title: {
          text: '示例折线图'
        },
        tooltip: {
          trigger: 'axis'
        },
        legend: {
          data: ['数据系列A', '数据系列B']
        },
        xAxis: {
          type: 'category',
          boundaryGap: false,
          data: ['周一', '周二', '周三', '周四', '周五', '周六', '周日']
        },
        yAxis: {
          type: 'value'
        },
        series: [
          {
            name: '数据系列A',
            type: 'line',
            stack: '总量',
            areaStyle: {},
            data: [120, 132, 101, 134, 90, 230, 210]
          },
          {
            name: '数据系列B',
            type: 'line',
            stack: '总量',
            data: [220, 182, 191, 234, 290, 330, 310]
          }
        ]
      };

      this.chartInstance.setOption(option);
    }
  }
};
</script>

<style scoped></style>