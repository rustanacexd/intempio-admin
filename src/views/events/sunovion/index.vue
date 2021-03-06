<template>
  <div class="app-container calendar-list-container">
    <div class="filter-container" style="margin-bottom: 25px">
      <el-input @keyup.enter.native="handleFilter" style="width: 200px;" class="filter-item" placeholder="Search"
                v-model="listQuery.search">
      </el-input>

      <el-select clearable class="filter-item" style="width: 130px" v-model="listQuery.status" placeholder="status">
        <el-option v-for="item in  statusOptions" :key="item.key" :label="item.display_name"
                   :value="item.key">
        </el-option>
      </el-select>
      <el-select @change='handleFilter' class="filter-item" v-model="listQuery.ordering">
        <el-option v-for="item in sortOptions" :key="item.key" :label="item.label" :value="item.key">
        </el-option>
      </el-select>
      <el-button class="filter-item" type="primary" v-waves icon="el-icon-search" @click="handleFilter">Search
      </el-button>
      <el-button class="filter-item" style="margin-left: 10px;" @click="handleCreate" type="primary"
                 icon="el-icon-edit" v-if="!isClient">Add
      </el-button>
    </div>

    <el-table :key='tableKey' :data="list" v-loading="listLoading" element-loading-text="loading" border fit
              highlight-current-row
              style="width: 100%">
      <el-table-column align="center" label="Event ID" width="300" v-if="!isClient">
        <template slot-scope="scope">
          <span class="link-type" style="cursor: pointer" @click="handleUpdate(scope.row)">{{scope.row.id}}</span>
        </template>
      </el-table-column>

      <el-table-column label="Event Name">
        <template slot-scope="scope">
          <span class="link-type" style="cursor: pointer" @click="handleUpdate(scope.row)">{{scope.row.name}}</span>
        </template>
      </el-table-column>

      <el-table-column width="200" align="center" label="date created">
        <template slot-scope="scope">
          <span>{{scope.row.created }}</span>
        </template>
      </el-table-column>

      <el-table-column class-name="status-col" label="status" width="100" align="center">
        <template slot-scope="scope">
          <el-tag :type="scope.row.status | statusFilter">{{scope.row.status}}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="actions" width="350" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button type="primary" size="mini" @click="handleUpdate(scope.row)">
            {{isClient ? 'View' : 'Edit'}}
          </el-button>
          <template v-if="!isClient">
            <el-button size="mini" type="danger" @click="handleDelete(scope.row)">delete</el-button>
            <el-button v-if="scope.row.status === 'reviewed'" size="mini" type="success"
                       @click="handleSubmitToKF(scope.row)">submit to kf
            </el-button>
            <el-button size="mini" type="warning" @click="handleClone(scope.row)">clone</el-button>
          </template>
        </template>
      </el-table-column>
    </el-table>

    <div class="pagination-container" style="margin-top: 25px">
      <el-pagination background @current-change="handleCurrentChange"
                     :current-page="listQuery.page" :page-sizes="[10,20,30, 50]" :page-size="listQuery.limit"
                     layout="total, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </div>


  </div>
</template>

<script>
  import { fetchList, deleteEvent, createEvent, submitToKf } from '@/api/sunovionEvent'
  import waves from '@/directive/waves'

  const statusOptions = [
    { key: 'new', display_name: 'new' },
    { key: 'reviewed', display_name: 'reviewed' },
    { key: 'accepted', display_name: 'accepted' },
    { key: 'canceled', display_name: 'canceled' }
  ]

  const sortOptions = [
    { label: 'created descending', key: '-created' },
    { label: 'created ascending', key: 'created' },
    { label: 'modified descending', key: '-modified' },
    { label: 'modified ascending', key: 'modified' },
    { label: 'reviewed_at descending', key: '-reviewed_at' },
    { label: 'reviewed_at ascending', key: 'reviewed_at' },
    { label: 'accepted_at descending', key: '-accepted_at' },
    { label: 'accepted_at ascending', key: 'accepted_at' }
  ]

  export default {
    directives: {
      waves
    },
    computed: {
      isClient() { return this.$store.getters.isClient }
    },
    data() {
      return {
        tableKey: 0,
        list: null,
        total: null,
        listLoading: true,
        listQuery: {
          search: '',
          page: 1,
          ordering: '-created',
          status: '',
          limit: 10
        },
        sortOptions,
        statusOptions,
        textMap: {
          update: 'Edit',
          create: 'Create'
        }
      }
    },
    filters: {
      statusFilter(status) {
        const statusMap = {
          accepted: 'success',
          new: 'warning',
          reviewed: 'primary',
          canceled: 'danger'
        }
        return statusMap[status]
      }
    },
    created() {
      this.getList()
    },
    methods: {
      getList() {
        this.listLoading = true
        fetchList(this.listQuery).then(response => {
          this.list = response.results
          this.total = response.count
          this.listLoading = false
        })
      },
      handleFilter() {
        this.listQuery.page = 1
        this.getList()
      },
      handleCurrentChange(val) {
        this.listQuery.page = val
        this.getList()
      },
      handleDelete(row) {
        this.$confirm('This will permanently delete the event. Continue?', 'Warning', {
          confirmButtonText: 'OK',
          cancelButtonText: 'Cancel',
          type: 'warning'
        }).then(() => {
          this.listLoading = true
          deleteEvent(row.id).then(() => {
            this.listLoading = false
            this.getList()
            this.$notify({
              title: 'Success',
              message: `deleted ${row.name}`,
              type: 'success',
              duration: 3000
            })
          })
        }).catch(() => {
          this.$notify({
            title: 'Info',
            message: `Delete Canceled`,
            type: 'info',
            duration: 3000
          })
        })
      },
      handleCreate() {
        this.$router.push({ name: 'sunovionCreateEvent' })
      },
      handleUpdate(row) {
        this.$router.push({
          name: 'sunovionEditEvent',
          params: { eventId: row.id }
        })
      },
      handleClone(row) {
        row.name = row.name + ' Copy'
        row.status = 'new'
        this.loading = true
        createEvent(row).then(data => {
          this.loading = false
          this.$router.push({
            name: 'sunovionEditEvent',
            params: { eventId: data.id }
          })
        })
      },
      handleSubmitToKF(row) {
        this.listLoading = true
        submitToKf(row.id).then(() => {
          this.$notify({
            title: 'Success',
            message: 'Successfully submitted to Kissflow',
            type: 'success',
            duration: 3000
          })
          this.listLoading = false
        })
      }
    }
  }
</script>
