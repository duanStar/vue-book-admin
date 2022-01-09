<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input v-model="listQuery.title" placeholder="书名" style="width: 200px;margin-left: 10px;" class="filter-item" clearable @keyup.enter.native="handleFilter" @clear="handleFilter" @blur="handleFilter" />
      <el-input v-model="listQuery.author" placeholder="作者" style="width: 200px;margin-left: 10px;" class="filter-item" clearable @keyup.enter.native="handleFilter" @clear="handleFilter" @blur="handleFilter" />
      <el-select v-model="listQuery.category" placeholder="分类" clearable class="filter-item" style="margin-left: 10px;" @change="handleFilter">
        <el-option v-for="item in categoryList" :key="item.value" :value="item.label" :label="item.label + '(' + item.num + ')'" />
      </el-select>
      <el-button v-waves class="filter-item" type="primary" icon="el-icon-search" style="margin-left: 10px;" @click="handleFilter">查询</el-button>
      <el-button class="filter-item" type="primary" icon="el-icon-edit" @click="handlerCreate">新增</el-button>
      <el-checkbox v-model="showCover" class="filter-item" style="margin-left: 10px;" @change="changShowCover">显示封面</el-checkbox>
      <el-table :key="tableKey" v-loading="loading" :data="list" border fit highlight-current-row style="width: 100%;" :default-sort="defaultSort" @sort-change="sortChange">
        <el-table-column label="ID" prop="id" sortable="custom" align="center" width="80" />
        <el-table-column label="书名" align="center" width="150">
          <template slot-scope="{ row: { titleWrapper } }">
            <span v-html="titleWrapper" />
          </template>
        </el-table-column>
        <el-table-column label="作者" align="center" width="150">
          <template slot-scope="{ row: { authorWrapper } }">
            <span v-html="authorWrapper" />
          </template>
        </el-table-column>
        <el-table-column label="出版社" prop="publisher" align="center" width="150" />
        <el-table-column label="分类" prop="categoryText" align="center" width="100" />
        <el-table-column label="语言" prop="publisher" align="center" />
        <el-table-column v-if="showCover" label="封面" align="center" width="150">
          <template slot-scope="scope">
            <a :href="scope.row.cover" target="_blank">
              <img :src="scope.row.cover" style="width: 120px;height: 180px;">
            </a>
          </template>
        </el-table-column>
        <el-table-column label="文件名" prop="fileName" align="center" width="100" />
        <el-table-column label="文件路径" align="center" width="100">
          <template slot-scope="{ row: { filePath } }">
            <span>{{ filePath | valueFilter }}</span>
          </template>
        </el-table-column>
        <el-table-column label="封面路径" align="center" width="100">
          <template slot-scope="{ row: { coverPath } }">
            <span>{{ coverPath | valueFilter }}</span>
          </template>
        </el-table-column>
        <el-table-column label="解压路径" align="center" width="100">
          <template slot-scope="{ row: { unzipPath } }">
            <span>{{ unzipPath | valueFilter }}</span>
          </template>
        </el-table-column>
        <el-table-column label="上传人" align="center" width="100">
          <template slot-scope="{ row: { createUser } }">
            <span>{{ createUser | valueFilter }}</span>
          </template>
        </el-table-column>
        <el-table-column label="上传时间" align="center" width="100">
          <template slot-scope="{ row: { createDt } }">
            <span>{{ createDt | timeFilter }}</span>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="120" fixed="right" align="center">
          <template slot-scope="{ row }">
            <el-button type="text" icon="el-icon-edit" @click="handleUpdate(row)">编辑</el-button>
            <el-button type="text" icon="el-icon-delete" style="color: #f56c6c" @click="handleDelete(row)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <Pagination v-show="total > 0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.pageSize" @pagination="refresh" />
    </div>
  </div>
</template>

<script>
import Pagination from '@/components/Pagination'
import waves from '@/directive/waves'
import { getCategory, listBook, deleteBook } from '@/api/book'
import { parseTime } from '@/utils'

export default {
  name: 'List',
  components: {
    Pagination
  },
  directives: {
    waves
  },
  filters: {
    valueFilter(val) {
      return val || '无'
    },
    timeFilter(val) {
      return val ? parseTime(val, '{y}-{m}-{d} {h}:{m}:{s}') : '无'
    }
  },
  data() {
    return {
      listQuery: {
      },
      showCover: false,
      categoryList: [],
      tableKey: 0,
      loading: true,
      list: [],
      total: 0,
      defaultSort: {}
    }
  },
  created() {
    this.parseQuery()
  },
  mounted() {
    this.getList()
    this.getCategoryList()
  },
  beforeRouteUpdate(to, from, next) {
    if (to.path === from.path) {
      const newQuery = Object.assign({}, to.query)
      const oldQuery = Object.assign({}, from.query)
      if (JSON.stringify(newQuery) !== JSON.stringify(oldQuery)) {
        this.getList()
      }
    }
    next()
  },
  methods: {
    parseQuery() {
      const query = Object.assign({}, this.$route.query)
      let sort = '+id'
      const listQuery = {
        page: 1,
        pageSize: 20,
        sort
      }
      if (query) {
        query.page && (query.page = +query.page)
        query.pageSize && (query.pageSize = +query.pageSize)
        query.sort && (sort = query.sort)
      }
      const symbol = sort[0]
      const column = sort.substring(1)
      this.defaultSort = {
        prop: column,
        order: symbol === '+' ? 'ascending' : 'descending'
      }
      this.listQuery = {
        ...listQuery,
        ...query
      }
    },
    getCategoryList() {
      getCategory().then(res => {
        this.categoryList = res.data
      })
    },
    wrapperKeyWork(key, value) {
      function highlight(val) {
        return `<span style="color: #1890ff;">${val}</span>`
      }
      const queryKey = this.listQuery[key]
      if (!queryKey) {
        return value
      } else {
        return value.replace(new RegExp(queryKey, 'ig'), v => highlight(v))
      }
    },
    getList() {
      this.loading = true
      listBook(this.listQuery).then(res => {
        const { data: { list, count }} = res
        this.total = count
        this.list = list
        this.loading = false
        this.list.forEach(book => {
          book.titleWrapper = this.wrapperKeyWork('title', book.title)
          book.authorWrapper = this.wrapperKeyWork('author', book.author)
        })
      })
    },
    refresh() {
      this.$router.push(({
        path: '/book/list',
        query: this.listQuery
      }))
    },
    handleFilter() {
      this.listQuery.page = 1
      this.refresh()
    },
    handlerCreate() {
      this.$router.push({ name: 'bookCreate' })
    },
    changShowCover(value) {
      this.showCover = value
    },
    sortChange(data) {
      const { prop, order } = data
      this.defaultSort = data
      this.sortBy(prop, order)
    },
    sortBy(prop, order) {
      if (order === 'ascending') {
        this.listQuery.sort = `+${prop}`
      } else {
        this.listQuery.sort = `-${prop}`
      }
      this.handleFilter()
    },
    handleUpdate(book) {
      const { fileName } = book
      this.$router.push(`/book/edit/${fileName}`)
    },
    handleDelete(book) {
      const { fileName } = book
      this.$confirm('此操作将永久删除该电子书，是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        deleteBook(fileName).then(res => {
          this.$notify({
            title: '成功',
            message: res.msg || '删除成功',
            type: 'success',
            duration: 2000
          })
          this.handleFilter()
        })
      })
    }
  }
}
</script>

<style lang="scss" scoped>

</style>
