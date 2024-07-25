<template>
  <div class="app-container">
    <div class="search-container">
      <el-input
        class="search-username"
        placeholder="Search by Username"
        v-model="searchQueryUsername"
        clearable
        @clear="clearSearch"
      />
      <el-button type="primary" @click="handleSearch('username')" class="search-button">Search</el-button>
    </div>
    <!-- 数据源为filteredItems -->
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
          <span v-html="scope.row.username"></span>
        </template>
      </el-table-column>
      <el-table-column label="用户ID">
        <template slot-scope="scope">
          {{ scope.row.userId }}
        </template>
      </el-table-column>
      <el-table-column label="关注数" width="110" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.followingCount }}</span>
        </template>
      </el-table-column>
      <el-table-column label="被关注数" width="110" align="center">
        <template slot-scope="scope">
          {{ scope.row.followedCount }}
        </template>
      </el-table-column>
      <el-table-column class-name="status-col" label="贴文数" width="110" align="center">
        <template slot-scope="scope">
          {{ scope.row.postCount }}
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
          userId: '12345',
          followingCount: 15,
          followedCount: 22,
          postCount: 11,
          location: 'New York'
        },
        {
          username: 'Jane Smith',
          userId: '67890',
          followingCount: 20,
          followedCount: 30,
          postCount: 8,
          location: 'Los Angeles'
        }
      ],
      cachedData: null,
      highlightedUsername: ''
    }
  },
  created() {
    this.fetchData()
  },
  computed: {
    filteredItems() {  //  filteredItems 计算属性会遍历 this.list 数组，并对每个元素进行映射。映射后的每个元素都会包含原始数据（通过 ...item 展开操作符）以及一个经过 highlightText 方法处理后的 username 字段。
      return this.list.map(item => ({
        ...item,
        username: this.highlightText(item.username, this.highlightedUsername)
      }))
    }
  },
  methods: {
    async fetchData() {
      if (this.cachedData) {
        // 如果缓存中有数据，使用缓存的数据
        this.list = this.cachedData
        return // 直接返回，不再发送请求
      }

      this.listLoading = true
      try {
        const response = await axios.get('http://localhost:8080/user/getAllBaseUserInfo', {
          headers: {
            'Authorization': 'Bearer YOUR_ACCESS_TOKEN'
          }
        })
        console.log('API Response:', response.data)
        if (response.data.code === 200) {
          this.list = response.data.data
          this.cachedData = response.data.data // 缓存数据
        } else {
          console.warn('API Error:', response.data.message)
          this.list = this.defaultData
          this.cachedData = this.defaultData // 缓存默认数据
        }
      } catch (error) {
        console.error('Error loading data:', error)
        this.list = this.defaultData
        this.cachedData = this.defaultData // 缓存默认数据
      } finally {
        this.listLoading = false
      }
    },

    clearSearch() {
      this.searchQueryUsername = ''
      this.highlightedUsername = ''
      this.list = this.cachedData // 重置为缓存数据
    },

    async handleSearch(type) {
      console.log('Search type:', type)
      this.currentSearchType = type

      if (type === 'username' && this.searchQueryUsername) {
        this.listLoading = true
        this.highlightedUsername = this.searchQueryUsername // 设置加粗关键词

        try {
          // 使用实际的 POST 请求向后端发送 JSON 数据
          const response = await axios.post('http://localhost:8080/user/getBaseUserInfo', {
            username: this.searchQueryUsername
          }, {
            headers: {
              'Authorization': 'Bearer YOUR_ACCESS_TOKEN',
              'Content-Type': 'application/json' // 确保请求头为 JSON
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
    },

    highlightText(text, query) {
      if (!query) return text
      const regex = new RegExp(`(${query})`, 'gi')
      return text.replace(regex, '<b>$1</b>')
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
