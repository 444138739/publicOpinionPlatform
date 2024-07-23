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
      <el-input
        class="Title"
        placeholder="Search by title"
        v-model="searchQueryTitle"
        clearable
        @input="handleSearch('Title')"
      />
      <el-input
        class="Author+"
        placeholder="Search by Author"
        v-model="searchQueryAuthor"
        clearable
        @input="handleSearch('Author')"
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
      <el-table-column align="center" label="用户名" width="95">
        <template slot-scope="scope">
          {{ scope.row.username }}
        </template>
      </el-table-column>
      <el-table-column label="用户ID">
        <template slot-scope="scope">
          {{ scope.row.userid }}
        </template>
      </el-table-column>
      <el-table-column label="关注数" width="110" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.following_count }}</span>
        </template>
      </el-table-column>
      <el-table-column label="被关注数" width="110" align="center">
        <template slot-scope="scope">
          {{ scope.row.followed_count }}
        </template>
      </el-table-column>
      <el-table-column class-name="status-col" label="贴文数" width="110" align="center">
        <template slot-scope="scope">
          <el-tag :type="statusFilter(scope.row.post_count)">{{ scope.row.post_count }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column align="center" prop="created_at" label="地点" width="200">
        <template slot-scope="scope">
          <i class="el-icon-time" />
          <span>{{ scope.row.location}}</span>
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
      searchQueryID: '',
      searchQueryTitle: '',
      searchQueryAuthor: '',
      currentSearchType: '',
      defaultData: [ // 假数据
        {
          username: 'John Doe',
          userid: '12345',
          following_count: 15,
          followed_count: 22,
          post_count: 11,
          location: 'New York'
        },
        {
          username: 'Jane Smith',
          userid: '67890',
          following_count: 20,
          followed_count: 30,
          post_count: 8,
          location: 'Los Angeles'
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
        const response = await axios.get('/api/user/getBaseUserInfo', {
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
    },

    statusFilter(status) {
      const statusMap = {
        published: 'success',
        draft: 'gray',
        deleted: 'danger'
      }
      return statusMap[status]
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
