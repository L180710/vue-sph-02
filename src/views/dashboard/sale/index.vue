<template>
  <el-card class="box-card" style="margin: 10px 0">
    <div slot="header" class="clearfix">
      <el-tabs class="tab" v-model="activeName">
        <el-tab-pane label="销售额" name="sale"></el-tab-pane>
        <el-tab-pane label="访问量" name="visit"></el-tab-pane>
      </el-tabs>
      <!-- 头部右侧内容 -->
      <div class="right">
        <span @click="setDay">今日</span>
        <span @click="setWeek">本周</span>
        <span @click="setMonth">本月</span>
        <span @click="setYear">本年</span>
        <el-date-picker
          v-model="date"
          class="date"
          type="datetimerange"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
          :default-time="['12:00:00']"
          value-format="yyyy-MM-dd"
        >
        </el-date-picker>
      </div>
    </div>
    <div>
      <el-row :gutter="10">
        <el-col :span="16">
          <!-- 容器 -->
          <div class="charts" ref="charts"></div>
        </el-col>
        <el-col :span="8">
          <h3>门店{{title}}排名</h3>
          <ul>
            <li>
              <span class="rindex">1</span>
              <span>肯德基</span>
              <span class="rvalue">1234</span>
            </li>
            <li>
              <span class="rindex">2</span>
              <span>肯德基</span>
              <span class="rvalue">1234</span>
            </li>
            <li>
              <span class="rindex">3</span>
              <span>肯德基</span>
              <span class="rvalue">1234</span>
            </li>
            <li>
              <span>4</span>
              <span>肯德基</span>
              <span class="rvalue">1234</span>
            </li>
            <li>
              <span>5</span>
              <span>肯德基</span>
              <span class="rvalue">1234</span>
            </li>
            <li>
              <span>6</span>
              <span>肯德基</span>
              <span class="rvalue">1234</span>
            </li>
            <li>
              <span>7</span>
              <span>肯德基</span>
              <span class="rvalue">1234</span>
            </li>
          </ul>
        </el-col>
      </el-row>
    </div>
  </el-card>
</template>

<script>
import * as echarts from "echarts";
import dayjs from "dayjs";
export default {
  name: "",
  data() {
    return {
      activeName: "sale",
      myEcharts: null,
      date: []
    };
  },
  computed: {
    // 计算属性 - 标题
    title() {
      return this.activeName == 'sale' ? '销售额' : '访问量';
    }
  },
  watch: {
    title() {
      // 重新修改图标配置数据
      this.myEcharts.setOption({
        title: {
          text: this.title
        }
      })
    }
  },
  mounted() {
    // 初始化 echarts 实例
    this.myEcharts = echarts.init(this.$refs.charts);
    this.myEcharts.setOption({
      title: {
        text: this.title
      },
      tooltip: {
        trigger: "axis",
        axisPointer: {
          type: "shadow",
        },
      },
      grid: {
        left: "3%",
        right: "4%",
        bottom: "3%",
        containLabel: true,
      },
      xAxis: [
        {
          type: "category",
          data: ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"],
          axisTick: {
            alignWithLabel: true,
          },
        },
      ],
      yAxis: [
        {
          type: "value",
        },
      ],
      series: [
        {
          name: "Direct",
          type: "bar",
          barWidth: "60%",
          data: [10, 52, 200, 334, 390, 330, 220],
        },
      ],
    });
  },
  methods: {
    // 本天
    setDay() {
      const day = dayjs().format('YYYY-MM-DD');
      this.date = [day, day];
    },
    // 本周
    setWeek() {
      const start = dayjs().day(1).format('YYYY-MM-DD');
      const end = dayjs().day(7).format('YYYY-MM-DD');
      this.date = [start, end];
    },
    // 本月
    setMonth() {
      const start = dayjs().startOf('month').format('YYYY-MM-DD');
      const end = dayjs().endOf('month').format('YYYY-MM-DD');
      this.date = [start, end];
    },
    // 本年
    setYear() {
      const start = dayjs().startOf('year').format('YYYY-MM-DD');
      const end = dayjs().endOf('year').format('YYYY-MM-DD');
      this.date = [start, end];
    }
  },
};
</script>

<style scoped>
.clearfix {
  position: relative;
  display: flex;
  justify-content: space-between;
}

.tab {
  width: 100%;
}

.right {
  position: absolute;
  right: 0;
}

.date {
  width: 260px;
  margin: 0 20px;
}

.right span {
  margin: 10px;
}

.charts {
  width: 100%;
  height: 300px;
}

ul {
  list-style: none;
  width: 100%;
  height: 300px;
  padding: 0px;
}

ul li {
  height: 8%;
  margin: 10px 0;
}

.rindex {
  float: left;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #000;
  color: #fff;
  text-align: center;
}

.rvalue {
  float: right;
}
</style>
