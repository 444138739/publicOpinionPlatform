<template>
  <div class="app-container">
    <div class="search-container">
      <el-input
        placeholder="Search by title"
        v-model="searchQuery"
        clearable
        @input="handleSearch"
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
          {{ scope.$index }}
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
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: 'success',
        draft: 'gray',
        deleted: 'danger'
      }
      return statusMap[status]
    }
  },
  data() {
    return {
      list: null,
      listLoading: true,
      searchQuery: ''
    }
  },
  created() {
    this.fetchData()
    this.handleSearch()
  },
  computed: {
    filteredItems() {
      if (!this.searchQuery) {
        return this.list
      }
      return this.list.filter(item =>
        item.title.toLowerCase().includes(this.searchQuery.toLowerCase())
      )
    }
  },
  methods: {
    fetchData() {
      this.listLoading = true
      getList().then(response => {
        this.list = response.data.items
        this.listLoading = false
      })
    },
    handleSearch() {
      this.searchQuery = this.searchQuery.trim()
      // The search logic is handled by the computed property 'filteredItems'
    },
    load() {
      request.get('/admin').then(res => {
        if (res.code === '0') {
          this.tableData = res.data;
        } else {

        }
      })
    }
  }
}
</script>

<style scoped>
.app-container{
  padding:20px;
}

.search-container{
  display:flex;
  justify-content:flex-end;
  align-items:stretch;
  margin-bottom:20px;
}

.search-input{
  width: 300px;
}

.search-button{
  margin-left: 10px;
}

</style>

