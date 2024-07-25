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
    <el-table
      v-loading="listLoading"
      :data="filteredItems"
      element-loading-text="Loading"
      border
      fit
      highlight-current-row
    >
      <el-table-column label="平台" width="80">
        <template slot-scope="scope">
          {{ scope.row.platform }}
        </template>
      </el-table-column>
      <el-table-column align="center" label="发帖人用户ID" width="140">
        <template slot-scope="scope">
          <span>{{ scope.row.postId }}</span>

        </template>
      </el-table-column>
      <el-table-column label="发帖人用户名" width="120" align="center">
        <template slot-scope="scope">
          <span v-html=" scope.row.postUsername "></span>
        </template>
      </el-table-column>
      <el-table-column label="贴文类型" width="100" align="center">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.postedId" class="forward-tag" type="primary">
            <el-icon class="el-icon-share"></el-icon>  转发
          </el-tag>
          <el-tag v-else class="original-tag" type="success">
            <el-icon class="el-icon-document"></el-icon>  原发
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column label="文本内容" >
        <template slot-scope="scope">
          {{ scope.row.hitSentence }}
        </template>
      </el-table-column>
      <el-table-column label="语言" width="80">
        <template slot-scope="scope">
          {{ scope.row.language }}
        </template>
      </el-table-column>

      <el-table-column align="center" prop="date" label="发布时间" width="180">
        <template slot-scope="scope">
          <i class="el-icon-time" />
          <span>{{ scope.row.date }}</span>
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
      searchQueryUsername: '',
      currentSearchType: '',
      defaultData: [ // 假数据
        {
          platform: 'Twitter',
          postUsername: 'John Doe',
          postId: '12345',
          postedUsername: 'Join',
          postedId: '12345',
          hitSentence: '文本内容',
          language: 'English',
          date: '2022-10-12 10:10:10'
        },
        {
          platform: 'Twitter',
          postUsername: 'John Doe',
          postId: '12345',
          postedUsername: 'Join',
          postedId: '12345',
          hitSentence: '文本内容',
          language: 'English',
          date: '2022-10-12 10:10:10'
        },
      ],
      cachedData: null,
      highlightedUsername: ''
    }
  },
  created() {
    this.fetchData()
  },
  computed: {
    filteredItems() {
      return this.list.map(item => ({
        ...item,
        postUsername: this.highlightText(item.postUsername, this.highlightedUsername)  //  username的取值需要在这里获取  对应传回的数据库中的值
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
        const response = await axios.get('http://localhost:8080/post/getAllPostInfo', {
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
          const response = await axios.post('http://localhost:8080/post/getPostInfoByUsername', {
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
      console.log(text)
      const regex = new RegExp(`(${query})`, 'gi')
      return text.replace(regex, '<span style="background-color: #D3E3FD; font-weight: bold; color: black;">$1</span>')
      //return text.replace(regex,'<b>$1</b>');
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

/* 标签字体加粗 */
.el-tag {
  font-weight: bold;
}
</style>
