<template>
  <div class="dashboard">
    <!-- 用户信息大卡片 -->
    <el-row :gutter="20">
      <el-col :span="24">
        <el-card :body-style="{ padding: '20px' }">
          <div class="user-card">
            <div class="user-avatar">
              <img :src="avatar + '?imageView2/1/w/80/h/80'" alt="User Avatar" />
            </div>
            <div class="user-info">
              <p class="user-name">admin</p>
              <p class="user-email">admin@example.com</p>
            </div>
          </div>
        </el-card>
      </el-col>
    </el-row>

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
          <div ref="followingChart" style="height: 400px;"></div>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card :body-style="{ padding: '20px' }">
          <div ref="followedChart" style="height: 400px;"></div>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card :body-style="{ padding: '20px' }">
          <div ref="longTextChart" style="height: 400px;"></div>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card :body-style="{ padding: '20px' }">
          <div ref="lateNightContentChart" style="height: 400px;"></div>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card :body-style="{ padding: '20px' }">
          <div ref="followingRatioChart" style="height: 400px;"></div>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card :body-style="{ padding: '20px' }">
          <div ref="forwardingChart" style="height: 400px;"></div>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card :body-style="{ padding: '20px' }">
          <div ref="forwardedChart" style="height: 400px;"></div>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card :body-style="{ padding: '20px' }">
          <div ref="forwardingRatioChart" style="height: 400px;"></div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import * as echarts from 'echarts';
import { mapGetters } from 'vuex';
import axios from 'axios';

export default {
  computed: {
    ...mapGetters(['avatar'])
  },
  data() {
    return {
      userInfo: {
        name: 'admin',
        email: 'zhangsan@example.com',
      },
      infoData: [],
      followingData: [],
      followedData: [],
      longTextData: [],
      lateNightContentData: [],
      followingRatioData: [],
      forwardingData: [],
      forwardedData: [],
      forwardingRatioData: [],
    };
  },
  async mounted() {
    await this.fetchData(); // 确保数据获取完成后再初始化图表
    this.initCharts();
  },
  methods: {
    async fetchData() {
      try {
        const response = await axios.get('http://localhost:8080/user/getExtraUserInfo');
        const data = response.data;

        // 基本信息
        this.infoData = [
          { label: '用户总数', value: data.baseInfo.usernum, icon: 'el-icon-user-solid', unit: '人' },
          { label: '贴文总数', value: data.baseInfo.postnum, icon: 'el-icon-s-comment', unit: '篇' },
          { label: '涉及国家', value: data.baseInfo.Country, icon: 'el-icon-s-promotion', unit: '个' },
          { label: '涉及平台数', value: (data.baseInfo.platform && data.baseInfo.platform.length), icon: 'el-icon-platform-eleme', unit: '个' }
        ];

        // 关注数数据
        this.followingData = data.followingInfo.ArrayListX.map((count, index) => ({
          value: data.followingInfo.ArrayListY[index],
          name: `关注数 ${count}`
        }));

        // 粉丝数数据
        this.followedData = data.followedInfo.ArrayListX.map((count, index) => ({
          value: data.followedInfo.ArrayListY[index],
          name: `粉丝数 ${count}`
        }));

        // 长内容占比数据
        this.longTextData = data.longTextInfo.ArrayListX.map((percentage, index) => ({
          value: data.longTextInfo.ArrayListY[index],
          name: `长内容占比 ${percentage}`
        }));

        // 深夜发布贴文占比数据
        this.lateNightContentData = data.lateNightContentRatioInfo.ArrayListX.map((ratio, index) => ({
          value: data.lateNightContentRatioInfo.ArrayListY[index],
          name: `比例 ${ratio}`
        }));

        // 关注比例数据
        this.followingRatioData = data.followingRatioInfo.ArrayListX.map((ratio, index) => ({
          value: data.followingRatioInfo.ArrayListY[index],
          name: `比例 ${ratio}`
        }));

        // 转发数数据
        this.forwardingData = data.forwardingInfo.ArrayListX.map((count, index) => ({
          value: data.forwardingInfo.ArrayListY[index],
          name: `转发数 ${count}`
        }));

        // 已转发数数据
        this.forwardedData = data.forwardedInfo.ArrayListX.map((count, index) => ({
          value: data.forwardedInfo.ArrayListY[index],
          name: `已转发数 ${count}`
        }));

        // 转发比例数据
        this.forwardingRatioData = data.forwardingRatioInfo.ArrayListX.map((ratio, index) => ({
          value: data.forwardingRatioInfo.ArrayListY[index],
          name: `转发比例 ${ratio}`
        }));

      } catch (error) {
        console.error('用户数据请求失败:', error);

        // 使用默认数据
        this.infoData = [
          { label: '用户总数', value: 0, icon: 'el-icon-user-solid', unit: '人' },
          { label: '贴文总数', value: 0, icon: 'el-icon-s-comment', unit: '篇' },
          { label: '涉及国家', value: 0, icon: 'el-icon-s-promotion', unit: '个' },
          { label: '涉及平台数', value: 0, icon: 'el-icon-platform-eleme', unit: '个' }
        ];

        this.followingData = [
          { value: 1500, name: '账号A' },
          { value: 1200, name: '账号B' },
          { value: 1800, name: '账号C' },
          { value: 1100, name: '账号D' },
          { value: 1300, name: '账号E' }
        ];

        this.followedData = [
          { value: 8000, name: '账号A' },
          { value: 7000, name: '账号B' },
          { value: 9000, name: '账号C' },
          { value: 6000, name: '账号D' },
          { value: 8000, name: '账号E' }
        ];

        this.longTextData = [
          { value: 30, name: '账号A' },
          { value: 40, name: '账号B' },
          { value: 50, name: '账号C' },
          { value: 60, name: '账号D' },
          { value: 70, name: '账号E' }
        ];

        this.lateNightContentData = [
          { value: 20, name: '账号A' },
          { value: 30, name: '账号B' },
          { value: 25, name: '账号C' },
          { value: 15, name: '账号D' },
          { value: 10, name: '账号E' }
        ];

        this.followingRatioData = [
          { value: 20, name: '账号A' },
          { value: 30, name: '账号B' },
          { value: 25, name: '账号C' },
          { value: 15, name: '账号D' },
          { value: 10, name: '账号E' }
        ];

        this.forwardingData = [
          { value: 100, name: '账号A' },
          { value: 150, name: '账号B' },
          { value: 200, name: '账号C' },
          { value: 250, name: '账号D' },
          { value: 300, name: '账号E' }
        ];

        this.forwardedData = [
          { value: 50, name: '账号A' },
          { value: 75, name: '账号B' },
          { value: 100, name: '账号C' },
          { value: 125, name: '账号D' },
          { value: 150, name: '账号E' }
        ];

        this.forwardingRatioData = [
          { value: 20, name: '账号A' },
          { value: 30, name: '账号B' },
          { value: 40, name: '账号C' },
          { value: 50, name: '账号D' },
          { value: 60, name: '账号E' }
        ];
      }
    },

    initCharts() {
      this.initBarChart(this.$refs.followingChart, '账号关注数', this.followingData.map(d => d.name), this.followingData.map(d => d.value));
      this.initBarChart(this.$refs.followedChart, '账号粉丝数', this.followedData.map(d => d.name), this.followedData.map(d => d.value), true);
      this.initLineChart(this.$refs.longTextChart, '长内容占比', this.longTextData.map(d => d.name), this.longTextData.map(d => d.value));
      this.initPieChart(this.$refs.lateNightContentChart, '深夜发布贴文占比', this.lateNightContentData);
      this.initPieChart(this.$refs.followingRatioChart, '关注比例分布', this.followingRatioData);
      this.initBarChart(this.$refs.forwardingChart, '转发情况', this.forwardingData.map(d => d.name), this.forwardingData.map(d => d.value));
      this.initBarChart(this.$refs.forwardedChart, '已转发情况', this.forwardedData.map(d => d.name), this.forwardedData.map(d => d.value));
      this.initLineChart(this.$refs.forwardingRatioChart, '转发比例', this.forwardingRatioData.map(d => d.name), this.forwardingRatioData.map(d => d.value));
    },

    initBarChart(element, title, xAxisData, seriesData, isHorizontal = false) {
      const chart = echarts.init(element);
      chart.setOption({
        title: { text: title, left: 'center' },
        tooltip: {},
        xAxis: isHorizontal ? {} : { data: xAxisData },
        yAxis: isHorizontal ? { data: xAxisData } : {},
        series: [{
          name: title,
          type: 'bar',
          data: seriesData,
          barWidth: '40%', // 设置柱子的宽度
          label: {
            show: true,
            position: isHorizontal ? 'inside' : 'top'
          }
        }]
      });
    },
    initLineChart(element, title, xAxisData, seriesData) {
      const chart = echarts.init(element);
      chart.setOption({
        title: { text: title, left: 'center' },
        tooltip: { trigger: 'axis' },
        xAxis: { type: 'category', data: xAxisData },
        yAxis: { type: 'value', axisLabel: { formatter: '{value}%' } },
        series: [{
          name: title,
          type: 'line',
          data: seriesData
        }]
      });
    },
    initPieChart(element, title, seriesData) {
      const chart = echarts.init(element);
      chart.setOption({
        title: { text: title, left: 'center' },
        tooltip: { trigger: 'item' },
        legend: { orient: 'vertical', left: 'left' },
        series: [{
          name: title,
          type: 'pie',
          radius: '50%',
          data: seriesData,
          emphasis: {
            itemStyle: {
              shadowBlur: 10,
              shadowOffsetX: 0,
              shadowColor: 'rgba(0, 0, 0, 0.5)'
            }
          }
        }]
      });
    }
  }
}
</script>


<style scoped lang="scss">
.dashboard {
  background-color: #f0f2f5; /* 浅灰背景 */
  padding: 30px;

  .el-row {
    margin-bottom: 10px; /* 行底部间距 */
  }

  .el-col {
    margin-bottom: 20px; /* 每个图表底部间距，增大上下间隔 */
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

  /* 添加图表的外层容器间隔 */
  .chart-row > .el-col {
    margin-top: 10px;
    margin-bottom: 10px;
  }
}
</style>
