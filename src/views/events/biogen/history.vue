<template>
  <div class="app-container historic-data">
    <h4></h4>
    <el-table :key='tableKey' :data="list" v-loading="listLoading" element-loading-text="loading" border fit
              highlight-current-row
              style="width: 100%">

      <el-table-column label="Date Changed" align="center">
        <template slot-scope="scope">
          <span class="link-type">{{scope.row.history_date}}</span>
        </template>
      </el-table-column>

      <el-table-column width="150" align="center" label="changed by">
        <template slot-scope="scope">
          <span>{{scope.row.history_user }}</span>
        </template>
      </el-table-column>

      <el-table-column label="actions" width="340" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button type="primary" size="mini" @click="handlePreview(scope.row)">view</el-button>
        </template>
      </el-table-column>

    </el-table>
    <div class="pagination-container" style="margin-top: 25px">
      <el-pagination background @current-change="handleCurrentChange"
                     :current-page="listQuery.page" :page-size="listQuery.limit"
                     layout="total, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </div>
    <el-dialog
      title="Event Data"
      :visible="dialogVisible"
      :show-close="false"
    >
      <table align="center" class="historic-data-table">
        <tr>
          <td><strong>Name</strong></td>
          <td>{{ eventObj.name }}</td>
        </tr>
        <tr>
          <td><strong>Phone</strong></td>
          <td>{{ eventObj.phone }}</td>
        </tr>
        <tr>
          <td><strong>Email</strong></td>
          <td>{{ eventObj.email }}</td>
        </tr>
        <tr>
          <td><strong>Requestor Name</strong></td>
          <td>{{ eventObj.requestor_name }}</td>
        </tr>
        <tr>
          <td><strong>Date</strong></td>
          <td>{{ eventObj.date }}</td>
        </tr>
        <tr>
          <td><strong>Time</strong></td>
          <td>{{ eventObj.time }}</td>
        </tr>
        <tr>
          <td><strong>Period</strong></td>
          <td>{{ eventObj.period }}</td>
        </tr>
        <tr>
          <td><strong>Timezone</strong></td>
          <td>{{ eventObj.timezone }}</td>
        </tr>
        <tr>
          <td><strong>Duration</strong></td>
          <td> {{eventObj.duration }} minutes</td>
        </tr>
        <tr>
          <td><strong>Presenters</strong></td>
          <td>
            <div v-for="presenter in eventObj.presenters">
              {{presenter.name}} - {{presenter.email}}
            </div>
          </td>
        </tr>
        <tr>
          <td><strong>Participants Count</strong></td>
          <td>{{ eventObj.participants_count }}</td>
        </tr>
        <tr>
          <td><strong>Presenters Count</strong></td>
          <td>{{ eventObj.presenters_count }}</td>
        </tr>
        <tr>
          <td><strong>Program Meeting Id</strong></td>
          <td> {{eventObj.program_meeting_id }}</td>
        </tr>
        <tr>
          <td><strong>Notes</strong></td>
          <td>{{ eventObj.notes }}</td>
        </tr>
        <tr>
          <td><strong>EOD / Webcast</strong></td>
          <td> {{ eventObj.eod_webcast }}</td>
        </tr>
        <tr>
          <td><strong>MS / SMA</strong></td>
          <td> {{ eventObj.ms_sma }}</td>
        </tr>
        <tr>
          <td><strong>Slide Deck Name</strong></td>
          <td> {{eventObj.slide_deck_name }}</td>
        </tr>
        <tr>
          <td><strong>Slide Deck Id</strong></td>
          <td>{{ eventObj.slide_deck_id }}</td>
        </tr>
        <tr>
          <td><strong>Status</strong></td>
          <td>{{ eventObj.status }}</td>
        </tr>
        <tr>
          <td><strong>Project</strong></td>
          <td>{{ eventObj.project_code }}</td>
        </tr>
      </table>
      <span slot="footer" class="dialog-footer">
          <el-button @click="dialogVisible = false">Cancel</el-button>
          <el-button type="primary" @click="handleRevert">Revert</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  import { fetchEventHistory, revertEvent } from '@/api/biogenEvent'

  const defaultEventObj = {
    name: '',
    phone: '',
    email: '',
    requestor_name: '',
    date: '',
    time: '',
    period: '',
    timezone: 'US/Eastern',
    duration: '',
    presenters: [],
    participants_count: 10,
    presenters_count: 1,
    program_meeting_id: '',
    notes: '',
    eod_webcast: '',
    ms_sma: '',
    project: '',
    slide_deck_name: '',
    slide_deck_id: '',
    status: 'new',
    history_user: '',
    history_id: '',
    history_date: '',
    project_code: ''
  }
  export default {
    data() {
      return {
        tableKey: 0,
        list: null,
        total: null,
        listLoading: true,
        listQuery: {
          page: 1,
          limit: 10
        },
        dialogVisible: false,
        eventObj: Object.assign({}, defaultEventObj)
      }
    },
    created() {
      this.getList()
    },
    methods: {
      getList() {
        this.listLoading = true
        fetchEventHistory(this.$route.params.eventId).then(response => {
          this.list = response.results
          this.total = response.count
          this.listLoading = false
          console.log(response)
        })
      },
      handleCurrentChange(val) {
        this.listQuery.page = val
        this.getList()
      },
      handlePreview(row) {
        this.eventObj = row
        this.dialogVisible = true
      },
      handleRevert() {
        this.$confirm('This will permanently revert the event data. Continue?', 'Warning', {
          confirmButtonText: 'OK',
          cancelButtonText: 'Cancel',
          type: 'warning'
        }).then(() => {
          revertEvent(this.$route.params.eventId, { history_id: this.eventObj.history_id })
            .then(() => {
              this.$notify({
                title: 'Success',
                message: 'Successfully reverted event data',
                type: 'success',
                duration: 3000
              })
              this.$router.go(-1)
            })
        })
      }
    }
  }
</script>

