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
        class="Author"
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
      <el-table-column align="center" label="ID" width="95">
        <template slot-scope="scope">
          {{ scope.row.ID }}
        </template>
      </el-table-column>
      <el-table-column label="Title">
        <template slot-scope="scope">
          {{ scope.row.title }}
        </template>
      </el-table-column>
      <el-table-column label="Author" width="110" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.author }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Pageviews" width="110" align="center">
        <template slot-scope="scope">
          {{ scope.row.pageviews }}
        </template>
      </el-table-column>
      <el-table-column class-name="status-col" label="Status" width="110" align="center">
        <template slot-scope="scope">
          <el-tag :type="statusFilter(scope.row.status)">{{ scope.row.status }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column align="center" prop="created_at" label="Display_time" width="200">
        <template slot-scope="scope">
          <i class="el-icon-time" />
          <span>{{ scope.row.display_time }}</span>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import { getList } from '@/api/table'
import request from '@/utils/request'

export default {
  data() {
    return {
      list: [], // 确保list初始化为空数组
      listLoading: true,
      searchQueryID: '',
      searchQueryTitle: '',
      searchQueryAuthor: '',
      currentSearchType: ''
    }
  },
  created() {
    this.fetchData()
  },
  computed: {
    filteredItems() {
      let filteredList = this.list

      if (this.currentSearchType === 'ID' && this.searchQueryID) {
        filteredList = filteredList.filter(item =>
          item.ID && item.ID.toString().includes(this.searchQueryID)
        )
      }

      if (this.currentSearchType === 'Title' && this.searchQueryTitle) {
        filteredList = filteredList.filter(item =>
          item.title && item.title.toLowerCase().includes(this.searchQueryTitle.toLowerCase())
        )
      }

      if (this.currentSearchType === 'Author' && this.searchQueryAuthor) {
        filteredList = filteredList.filter(item =>
          item.author && item.author.toLowerCase().includes(this.searchQueryAuthor.toLowerCase())
        )
      }

      return filteredList
    }
  },
  methods: {
    fetchData() {
      this.listLoading = true
      getList().then(response => {
        console.log('Data loaded:', response.data.items)
        this.list = response.data.items
        this.listLoading = false
      }).catch(error => {
        console.error('Error loading data:', error)
        this.listLoading = false
      })
    },
    handleSearch(type) {
      console.log('Search type:', type)
      this.currentSearchType = type
    },
    load() {
      request.get('/admin').then(res => {
        if (res.code === '0') {
          console.log('Data loaded from /admin:', res.data)
          this.tableData = res.data
        } else {
          console.error('Error loading data from /admin:', res)
        }
      }).catch(error => {
        console.error('Error loading data from /admin:', error)
      })
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
