<template>
  <div class="dashboard">
    <!-- 用户信息大卡片 -->
    <el-row :gutter="20">
      <el-col :span="24">
        <el-card :body-style="{ padding: '20px' }">
          <div class="platformTitle">
            NH-数据舆论分析系统
          </div>
        </el-card>
      </el-col>
    </el-row>

    <!-- 统计卡片 -->

    <el-row :gutter="20">
      <el-col :span="6" v-for="data in infoData" :key="data.label">
        <el-card :body-style="{ padding: '20px', height: '150px' }"> <!-- 设置固定高度 -->
          <div class="stat-card">
            <div class="stat-icon">
              <i :class="data.icon"></i>
            </div>
            <div class="stat-info">
              <div v-if="Array.isArray(data.value)">
                <ul class="plat-content">
                  <li v-for="platform in data.value" :key="platform.name">
                    <span class="platform-name">{{ platform.name }}: </span>
                    <span class="platform-value">{{ platform.value }}{{ data.unit }}</span>
                  </li>
                </ul>
                <p class="plat-label">{{ data.label }}</p>
              </div>
              <div v-else>
                <p class="stat-value">{{ data.value }}{{ data.unit }}</p>
                <p class="stat-label">{{ data.label }}</p>
              </div>
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
        const platformCount = {
          'Twitter': 0,
          'Facebook': 0,
          'Instagram': 0,
          'TikTok': 0,
        };
        data.baseInfo.platform.forEach(platform => {
          if (platformCount[platform] !== undefined) {
            platformCount[platform]++;
          }
        });

        // 基本信息
        this.infoData = [
          { label: '用户总数', value: data.baseInfo.usernum, icon: 'el-icon-user-solid', unit: '人' },
          { label: '贴文总数', value: data.baseInfo.postnum, icon: 'el-icon-s-comment', unit: '篇' },
          { label: '涉及国家', value: data.baseInfo.Country, icon: 'el-icon-s-promotion', unit: '个' },
          { label: '涉及平台数', value: Object.keys(platformCount).map(key => `${key} ${platformCount[key]}`).join('; '), icon: 'el-icon-platform-eleme', unit: '个' }
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
          { label: '用户总数', value: 3, icon: 'el-icon-user-solid', unit: '人' },
          { label: '贴文总数', value: 2, icon: 'el-icon-s-comment', unit: '篇' },
          { label: '涉及国家', value: 1, icon: 'el-icon-s-promotion', unit: '个' },
          {
            label: '涉及平台数',
            value: [
              { name: 'Twitter', value: 0 },
              { name: 'Facebook', value: 0 },
              { name: 'Instagram', value: 0 },
              { name: 'TikTok', value: 0 },
            ],
            icon: 'el-icon-platform-eleme',
            unit: '个'
          }
        ];

        this.followingData = [
          { value: 1500, name: '关注数0-49' },
          { value: 1200, name: '50-99' },
          { value: 1800, name: '100-149' },
          { value: 1100, name: '150-199' },
          { value: 1300, name: '200以上' }
        ];

        this.followedData = [
          { value: 1500, name: '0-99' },
          { value: 1200, name: '100-199' },
          { value: 1800, name: '200-299' },
          { value: 1100, name: '300-399' },
          { value: 1300, name: '400以上' }
        ];


        this.longTextData = [
          { value: 30, name: '长内容占比 0-0.1' },
          { value: 40, name: '长内容占比 0.1-0.2' },
          { value: 50, name: '长内容占比 0.2-0.3' },
          { value: 60, name: '长内容占比 0.3-0.4' },
          { value: 70, name: '长内容占比 0.4-0.5' },
          { value: 80, name: '长内容占比 0.5-0.6' },
          { value: 90, name: '长内容占比 0.6-0.7' },
          { value: 100, name: '长内容占比 0.7-0.8' },
          { value: 110, name: '长内容占比 0.8-0.9' },
          { value: 120, name: '长内容占比 0.9-1.0' }
        ];

        this.lateNightContentData = [
          { value: 20, name: '比例 0-0.1' },
          { value: 30, name: '比例 0.1-0.2' },
          { value: 25, name: '比例 0.2-0.3' },
          { value: 15, name: '比例 0.3-0.4' },
          { value: 10, name: '比例 0.4-0.5' },
          { value: 5, name: '比例 0.5-0.6' },
          { value: 10, name: '比例 0.6-0.7' },
          { value: 15, name: '比例 0.7-0.8' },
          { value: 20, name: '比例 0.8-0.9' },
          { value: 25, name: '比例 0.9-1.0' }
        ];

        this.followingRatioData = [
          { value: 20, name: '比例 0-0.1' },
          { value: 30, name: '比例 0.1-0.2' },
          { value: 25, name: '比例 0.2-0.3' },
          { value: 15, name: '比例 0.3-0.4' },
          { value: 10, name: '比例 0.4-0.5' },
          { value: 5, name: '比例 0.5-0.6' },
          { value: 10, name: '比例 0.6-0.7' },
          { value: 15, name: '比例 0.7-0.8' },
          { value: 20, name: '比例 0.8-0.9' },
          { value: 25, name: '比例 0.9-1.0' }
        ];

        this.forwardingData = [
          { value: 100, name: '转发数 0' },
          { value: 150, name: '转发数 1' },
          { value: 200, name: '转发数 2' },
          { value: 250, name: '转发数 3' },
          { value: 300, name: '转发数 4' }
        ];

        this.forwardedData = [
          { value: 50, name: '已转发数 0' },
          { value: 75, name: '已转发数 1' },
          { value: 100, name: '已转发数 2' },
          { value: 125, name: '已转发数 3' },
          { value: 150, name: '已转发数 4' }
        ];

        this.forwardingRatioData = [
          { value: 20, name: '转发比例 0-0.1' },
          { value: 30, name: '转发比例 0.1-0.2' },
          { value: 40, name: '转发比例 0.2-0.3' },
          { value: 50, name: '转发比例 0.3-0.4' },
          { value: 60, name: '转发比例 0.4-0.5' },
          { value: 70, name: '转发比例 0.5-0.6' },
          { value: 80, name: '转发比例 0.6-0.7' },
          { value: 90, name: '转发比例 0.7-0.8' },
          { value: 100, name: '转发比例 0.8-0.9' },
          { value: 110, name: '转发比例 0.9-1.0' }
        ];
      }
    },

    initCharts() {
      this.initColorfulBarChart(this.$refs.followingChart, '关注数', this.followingData.map(d => d.name), this.followingData.map(d => d.value));
      // this.initBarChart(this.$refs.followingChart, 'Following Distribution', followingInfo.ArrayListX, followingInfo.ArrayListY, false, '关注人数', '人数', '', '人');
      this.initColorfulBarChart(this.$refs.followedChart, '粉丝数', this.followedData.map(d => d.name), this.followedData.map(d => d.value), true);
      this.initLineChart(this.$refs.longTextChart, '长内容占比', this.longTextData.map(d => d.name), this.longTextData.map(d => d.value));
      this.initPieChart(this.$refs.lateNightContentChart, '深夜发布贴文占比', this.lateNightContentData);
      this.initCircleChart(this.$refs.followingRatioChart, '关注比例分布', this.followingRatioData);
      this.initBarChart(this.$refs.forwardingChart, '转发情况', this.forwardingData.map(d => d.name), this.forwardingData.map(d => d.value));
      this.initGreenBarChart(this.$refs.forwardedChart, '已转发情况', this.forwardedData.map(d => d.name), this.forwardedData.map(d => d.value));
      this.initLineChart(this.$refs.forwardingRatioChart, '转发比例', this.forwardingRatioData.map(d => d.name), this.forwardingRatioData.map(d => d.value));
    },
    // initBarChart(element, title, xAxisData, seriesData, isHorizontal = false) {
    //   const chart = echarts.init(element);
    //   chart.setOption({
    //     title: { text: title, left: 'center' },
    //     tooltip: {},
    //     xAxis: isHorizontal ? {} : { data: xAxisData },
    //     yAxis: isHorizontal ? { data: xAxisData } : {},
    //     series: [{
    //       name: title,
    //       type: 'bar',
    //       data: seriesData,
    //       barWidth: '40%', // 设置柱子的宽度
    //       label: {
    //         show: true,
    //         position: isHorizontal ? 'inside' : 'top'
    //       }
    //     }]
    //   });
    // },
    initBarChart(element, title, xAxisData, seriesData, isHorizontal = false, xAxisLabel = 'X轴', yAxisLabel = 'Y轴', xAxisUnit = '', yAxisUnit = '') {
      const chart = echarts.init(element);
      chart.setOption({
        title: { text: title, left: 'center' },
        tooltip: {},
        xAxis: isHorizontal ? {
          name: yAxisLabel,
          nameLocation: 'middle',
          nameGap: 30,
          axisLabel: {
            formatter: '{value} ' + yAxisUnit
          },
          data: xAxisData
        } : {
          name: xAxisLabel,
          nameLocation: 'middle',
          nameGap: 30,
          axisLabel: {
            formatter: '{value} ' + xAxisUnit
          },
          data: xAxisData
        },
        yAxis: isHorizontal ? {
          name: xAxisLabel,
          nameLocation: 'middle',
          nameGap: 30,
          axisLabel: {
            formatter: '{value} ' + xAxisUnit
          }
        } : {
          name: yAxisLabel,
          nameLocation: 'middle',
          nameGap: 30,
          axisLabel: {
            formatter: '{value} ' + yAxisUnit
          }
        },
        series: [{
          name: title,
          type: 'bar',
          data: seriesData,
          barWidth: '40%', // 设置柱子的宽度
          itemStyle: {
            color: 'green' // 设置柱子的颜色为绿色
          },
          label: {
            show: true,
            position: isHorizontal ? 'inside' : 'top'
          }
        }]
      });
    },
    initGreenBarChart(element, title, xAxisData, seriesData, isHorizontal = false) {
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
          itemStyle: {
            color:'rgb(68,189,68)',
          },
          barWidth: '40%', // 设置柱子的宽度
          label: {
            show: true,
            position: isHorizontal ? 'inside' : 'top'
          }
        }]
      });
    },
    initColorfulBarChart(element, title, xAxisData, seriesData, isHorizontal = false) {
      const chart = echarts.init(element);
      const colors = [
        '#5470C6', '#91CC75', '#FAC858', '#EE6666', '#73C0DE', '#3BA272', '#FC8452', '#9A60B4', '#EA7CCC'
      ]; // 颜色数组，可以根据需要添加更多颜色

      const seriesDataWithColors = seriesData.map((value, index) => ({
        value,
        itemStyle: { color: colors[index % colors.length] } // 轮流使用颜色数组中的颜色
      }));

      chart.setOption({
        title: { text: title, left: 'center' },
        tooltip: {},
        xAxis: isHorizontal ? {} : { data: xAxisData },
        yAxis: isHorizontal ? { data: xAxisData } : {},
        series: [{
          name: title,
          type: 'bar',
          data: seriesDataWithColors,
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
    },
    initCircleChart(element,title,seriesData){
      const chart = echarts.init(element);
      chart.setOption({
        title: { text:title, left: 'center'},
        tooltip: { trigger: 'item' },
        legend: {
          bottom: '5%',
          left: 'center'
        },
        series: [
          {
            name: title,
            type: 'pie',
            radius: ['40%', '70%'],
            avoidLabelOverlap: false,
            data: seriesData,
            label: {
              show: false,
              position: 'center'
            },
            emphasis: {
              label: {
                show: true,
                fontSize: 40,
                fontWeight: 'bold'
              }
            },
            labelLine: {
              show: false
            }
          }]
      })
    },
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

  .platformTitle {
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 30px;
    font-weight: bold;
    font-family: Arial,serif;
    padding:1px;
  }

  .stat-card {
    display: flex;
    align-items: center;

    .stat-icon{
      font-size: 40px;
      color: #409EFF; /* Element UI 默认的蓝色主题色 */
      margin-bottom: 20px;
    }

    .stat-info {
      margin-left: 12px;

      .stat-value {
        font-size: 24px;
        font-weight: bold;
        color: #333;
      }
      .plat-content{
        letter-spacing: revert;
        font-family:"Microsoft YaHei UI",serif;
        font-weight:bold;
        margin-top:10px;
      }
      .platform-name {
        margin-right: 10px; /* 增加name和value之间的间距 */
      }
      .platform-value {
        font-size: 16px;
        color: #333;
      }
      .plat-label {
          margin-left: 50px;
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
