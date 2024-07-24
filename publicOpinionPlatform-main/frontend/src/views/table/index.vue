<template>
  <div class="app-container">
    <div class="search-container">
      <el-input
        class="search-username"
        placeholder="Search by Username"
        v-moadddel="searchQueryUsername"
        clearable
      />
      <el-button type="primary" @click="handleSearch('username')" class="search-button">Search</el-button>
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
          {{ scope.row.post_count }}
        </template>
      </el-table-column>
      <el-table-column align="center" prop="created_at" label="地点" width="200">
        <template slot-scope="scope">
          <i class="el-icon-time" />
          <span>{{ scope.row.location }}</span>
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
      list: [],
      listLoading: true,
      searchQueryUsername: '',
      currentSearchType: '',
      defaultData: [
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
      return this.list
    }
  },
  methods: {
    async fetchData() {
      this.listLoading = true
      try {
        const response = await axios.get('/user/getBaseUserInfo', {
          headers: {
            'Authorization': 'Bearer YOUR_ACCESS_TOKEN'
          }
        })
        console.log('API Response:', response.data)
        if (response.data.code === 200) {
          this.list = response.data.data
        } else {
          console.warn('API Error:', response.data.message)
          this.list = this.defaultData
        }
      } catch (error) {
        console.error('Error loading data:', error)
        this.list = this.defaultData
      } finally {
        this.listLoading = false
      }
    },

    async handleSearch(type) {
      console.log('Search type:', type)
      this.currentSearchType = type

      if (type === 'username' && this.searchQueryUsername) {
        this.listLoading = true

        try {
          // 使用实际的 POST 请求向后端发送 JSON 数据
          const response = await axios.post('user/searchByUsername', {
            username: this.searchQueryUsername
          }, {
            headers: {
              'Authorization': 'Bearer YOUR_ACCESS_TOKEN',
              'Content-Type': 'application/json'// 确保请求头为 JSON
            }
          })

          console.log('Search API Response:', response.data)

          if (response.data.code === 200) {
            this.list = response.data.data
          } else {
            console.warn('Search API Error:', response.data.message)
            this.list = this.defaultData.filter(item =>
              item.username.toLowerCase().includes(this.searchQueryUsername.toLowerCase())
            )
          }
        } catch (error) {
          console.error('Error searching data:', error)
          // 使用模拟数据进行回退
          this.list = this.defaultData.filter(item =>
            item.username.toLowerCase().includes(this.searchQueryUsername.toLowerCase())
          )
        } finally {
          this.listLoading = false
        }
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
