<template>
  <!-- 容器 -->
  <div class="charts" ref="charts"></div>
</template>

<script>
// 引入 echarts
import * as echarts from "echarts";
export default {
  name: "",
  mounted() {
    // 初始化 echarts 实例
    let lineCharts = echarts.init(this.$refs.charts);
    // 配置数据
    lineCharts.setOption({
      xAxis: {
        show: false,
        min: 0,
        max: 100
      },
      yAxis: {
        show: false,
        type: "category",
      },
      // 系列
      series: [
        {
          type: "bar",
          data: [78],
          barWidth: 10,
          color: 'yellowgreen',
          showBackground: true,
          backgroundStyle: {color: '#eee'},
          // 文本
          label: {
            show: true,
            formatter: "|",
            position: 'right'
          }
        },

      ],
      // 布局调试
      grid: {
        left: 0,
        top: 0,
        right: 0,
        bottom: 0,
      },
    });
  },
};
</script>

<style>
.charts {
  width: 100%;
  height: 100%;
}
</style>
