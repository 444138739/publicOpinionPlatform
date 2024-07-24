<template>
    <div class="dashboard">
        <!-- 用户信息大卡片 -->
      <!-- <el-row :gutter="20">
        
        <el-col :span="24">
          <el-card :body-style="{ padding: '20px' }">
            <div class="user-card">
              <div class="user-avatar">
                <img :src="avatar+'?imageView2/1/w/80/h/80'" alt="User Avatar">
              </div>
              <div class="user-info">
                <p class="user-name">admin</p>
                <p class="user-email">admin@example.com</p>
              </div>
            </div>
          </el-card>
        </el-col>
      </el-row> -->
  
      <!-- 统计卡片 -->
      <el-row :gutter="20">
        <el-col :span="6" v-for="data in infoData" :key="data.label">
          <el-card :body-style="{ padding: '20px' }">
            <div class="stat-card">
              <div class="stat-icon">
                <i :class="data.icon"></i>
              </div>
              <div class="stat-info">
                <p class="stat-value">{{ data.value }}</p>
                <p class="stat-label">{{ data.label }}</p>
              </div>
            </div>
          </el-card>
        </el-col>
      </el-row>
  
      <el-row :gutter="20" class="chart-row">
      <el-col :span="12">
        <el-card :body-style="{ padding: '20px' }">
          <div ref="barChart" style="height: 400px;"></div>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card :body-style="{ padding: '20px' }">
          <div ref="horizontalBarChart" style="height: 400px;"></div>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card :body-style="{ padding: '20px' }">
          <div ref="lineChart" style="height: 400px;"></div>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card :body-style="{ padding: '20px' }">
          <div ref="pieChart" style="height: 400px;"></div>
        </el-card>
      </el-col>
    </el-row>


    </div>
  </template>
  
<script>

import * as echarts from 'echarts';
import { mapGetters } from 'vuex';

export default {
    computed: {
    ...mapGetters([
        'avatar'
    ])
    },
    data() {
        return {
        userInfo: {
            name: 'admin',
            email: 'zhangsan@example.com',
        },
        infoData: [
            { label: '用户总数', value: '2000', icon: 'el-icon-user-solid', unit: '人' },
            { label: '贴文总数', value: '850', icon: 'el-icon-s-comment', unit: '篇' },
            { label: '涉及国家', value: '15', icon: 'el-icon-s-promotion', unit: '个' },
            { label: '涉及平台数', value: '3', icon: 'el-icon-platform-eleme', unit: '个' }
        ]
        };
    },

    // Echarts画图
    mounted() {
    this.initBarChart();
    this.initHorizontalBarChart();
    this.initLineChart();
    this.initPieChart();
    },
    methods: {
    initBarChart() {
      const chart = echarts.init(this.$refs.barChart);
      chart.setOption({
        title: {
          text: '账号关注数'
        },
        tooltip: {},
        xAxis: {
          data: ['账号A', '账号B', '账号C', '账号D', '账号E']
        },
        yAxis: {},
        series: [{
          name: '关注数',
          type: 'bar',
          data: [1500, 1200, 1800, 1100, 1300]
        }]
      });
    },
    initHorizontalBarChart() {
      const chart = echarts.init(this.$refs.horizontalBarChart);
      chart.setOption({
        title: {
          text: '账号粉丝数'
        },
        tooltip: {},
        xAxis: {},
        yAxis: {
          data: ['账号A', '账号B', '账号C', '账号D', '账号E']
        },
        series: [{
          name: '粉丝数',
          type: 'bar',
          data: [8000, 7000, 9000, 6000, 8000],
          barWidth: '40%', // 设置柱子的宽度
          label: {
            show: true,
            position: 'inside'
          }
        }]
      });
    },
    initLineChart() {
      const chart = echarts.init(this.$refs.lineChart);
      chart.setOption({
        title: {
          text: '长内容占比'
        },
        tooltip: {
          trigger: 'axis'
        },
        xAxis: {
          type: 'category',
          data: ['账号A', '账号B', '账号C', '账号D', '账号E']
        },
        yAxis: {
          type: 'value',
          max: 100,
          min: 0,
          axisLabel: {
            formatter: '{value}%'
          }
        },
        series: [{
          name: '长内容占比',
          type: 'line',
          data: [30, 40, 50, 60, 70]
        }]
      });
    },
    initPieChart() {
      const chart = echarts.init(this.$refs.pieChart);
      chart.setOption({
        title: {
          text: '深夜发布贴文占比'
        },
        tooltip: {
          trigger: 'item'
        },
        legend: {
          orient: 'vertical',
          left: 'left'
        },
        series: [
          {
            name: '深夜发布贴文占比',
            type: 'pie',
            radius: '50%',
            data: [
              { value: 20, name: '账号A' },
              { value: 30, name: '账号B' },
              { value: 25, name: '账号C' },
              { value: 15, name: '账号D' },
              { value: 10, name: '账号E' }
            ],
            emphasis: {
              itemStyle: {
                shadowBlur: 10,
                shadowOffsetX: 0,
                shadowColor: 'rgba(0, 0, 0, 0.5)'
              }
            }
          }
        ]
      });
    }
  },
}
</script>

<style scoped lang="scss">
.dashboard {
background-color: #f0f2f5; /* 浅灰背景 */
padding: 30px;

.el-row {
    margin-bottom: 20px;
}

.user-card {
    display: flex;
    align-items: center;
    .user-avatar {
    img {
        width: 60px;
        height: 60px;
        border-radius: 50%;
    }
    }
    .user-info {
    margin-left: 12px;
    .user-name {
        font-size: 20px;
        font-weight: bold;
        color: #333;
    }
    .user-email {
        font-size: 14px;
        color: #666;
    }
    }
}

.stat-card {
    display: flex;
    align-items: center;
    .stat-icon {
    font-size: 40px;
    color: #409EFF; /* Element UI 默认的蓝色主题色 */
    }
    .stat-info {
    margin-left: 12px;
    .stat-value {
        font-size: 24px;
        font-weight: bold;
        color: #333;
    }
    .stat-label {
        font-size: 16px;
        color: #666;
    }
    }
}

.chart-row {
    background-color: #fff; /* 为图表行设置白色背景 */
    padding: 20px;
    border-radius: 4px;
}
}
</style>
