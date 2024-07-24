<template>
  <div class="app-container">
    <div class="search-container">
      <el-input
        class="ID"
        placeholder="Search by ID"
        v-model="searchQueryID"
        clearable
        @input="handleSearch('ID')"
      />
      <el-button type="primary" @click="handleSearch" class="search-button">Search</el-button>
    </div>
    <el-table
      v-loading="listLoading"
      :data="filteredItems"
      element-loading-text="Loading"
      border
      fit
      highlight-current-row
    >
      <el-table-column align="center" label="用户名">
        <template slot-scope="scope">
          {{ scope.row.username }}
        </template>
      </el-table-column>
      <el-table-column label="用户ID" width="110" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.user_id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="post_id" width="110" align="center">
        <template slot-scope="scope">
          {{ scope.row.post_id }}
        </template>
      </el-table-column>
      <el-table-column label="文本内容">
        <template slot-scope="scope">
          {{ scope.row.Hit_sentence }}
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>

import axios from 'axios'

export default {
  data() {
    return {
      list: [], // 确保list初始化为空数组
      listLoading: true,
      // searchQueryID: '',
      // searchQueryTitle: '',
      // searchQueryAuthor: '',
      currentSearchType: '',
      defaultData: [ // 假数据
        {
          username: 'John Doe',
          user_id: '12345',
          post_id: '1',
          Hit_sentence: '文本内容'
        },
        {
          username: 'John Doe',
          user_id: '23456',
          post_id: '2',
          Hit_sentence: '文本内容'
        }
      ]
    }
  },
  created() {
    this.fetchData()
  },
  computed: {
    filteredItems() {
      let filteredList = this.list

      if (this.searchQueryID) {
        filteredList = filteredList.filter(item =>
          item.userid && item.userid.toString().includes(this.searchQueryID)
        )
      }

      if (this.searchQueryTitle) {
        filteredList = filteredList.filter(item =>
          item.username && item.username.toLowerCase().includes(this.searchQueryTitle.toLowerCase())
        )
      }

      if (this.searchQueryAuthor) {
        filteredList = filteredList.filter(item =>
          item.location && item.location.toLowerCase().includes(this.searchQueryAuthor.toLowerCase())
        )
      }
      return filteredList
    }
  },
  methods: {
    async fetchData() {
      this.listLoading = true
      try {
        const response = await axios.get('/api/post/getPostInfo', {
          headers: {
            'Authorization': 'Bearer YOUR_ACCESS_TOKEN' // 如果需要身份验证
          }
        })
        console.log('API Response:', response.data) // 调试输出响应
        if (response.data.code === 200) {
          this.list = response.data.data // 使用实际数据
        } else {
          console.warn('API Error:', response.data.message)
          this.list = this.defaultData // 使用假数据
        }
      } catch (error) {
        console.error('Error loading data:', error)
        this.list = this.defaultData // 使用假数据
      } finally {
        this.listLoading = false
      }
    },
    handleSearch(type) {
      console.log('Search type:', type)
      this.currentSearchType = type
    },
    async load() {
      try {
        const response = await axios.get('/user/getBaseUserInfo', {
          params: {
            username: 'John Doe' // 模拟的查询参数
          },
          headers: {
            'Authorization': 'Bearer YOUR_ACCESS_TOKEN' // 如果需要身份验证
          }
        })
        console.log('Data loaded from /user/getBaseUserInfo:', response.data)
        this.tableData = response.data
      } catch (error) {
        console.error('Error loading data from /user/getBaseUserInfo:', error)
      }
    }
  }
}
</script>

<style scoped>
.app-container {
  padding: 20px;
}

.search-container {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  margin-bottom: 20px;
  gap: 10px;
}

.search-button {
  margin-left: 10px;
}
</style>
