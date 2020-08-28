<template>
  <div class="app-container">
    <!-- 查询项 -->
    <div class="filter-container">
      <el-input v-model="listQuery.symbol" placeholder="交易品种代码" style="width: 200px;" class="filter-item" @keyup.enter.native="handleFilter" />
      <el-select v-model="listQuery.side" placeholder="多空方向" clearable class="filter-item" style="width: 130px;margin-left: 10px;">
        <el-option v-for="item in sideOptions" :key="item.key" :label="item.display_name" :value="item.key" />
      </el-select>
      <el-select v-model="listQuery.status" placeholder="状态" clearable class="filter-item" style="width: 130px;margin-left: 10px;">
        <el-option v-for="item in statusOptions" :key="item.key" :label="item.display_name" :value="item.key" />
      </el-select>
      <el-button class="filter-item" style="margin-left: 10px;" type="primary" icon="el-icon-search" @click="handleFilter">
        查询
      </el-button>
      <el-button class="filter-item" type="primary" icon="el-icon-edit" @click="handleCreate">
        新增
      </el-button>
      <el-button v-waves :loading="downloadLoading" class="filter-item" type="primary" icon="el-icon-download" @click="handleDownload">
        导出
      </el-button>
    </div>

    <!-- 数据列表 -->
    <el-table
      :key="tableKey"
      v-loading="listLoading"
      :data="list"
      border
      fit
      highlight-current-row
      style="width: 100%;"
      @sort-change="sortChange"
    >
      <el-table-column label="入场日期" width="120px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.entryDate | parseTime('{y}-{m}-{d}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="品种代码" width="120px">
        <template slot-scope="{row}">
          <span>{{ row.symbol }}</span>
        </template>
      </el-table-column>
      <el-table-column label="多空方向" width="100px" align="center">
        <template slot-scope="{row}">
          <el-tag :type="row.side | colorFilter">
            {{ row.side | sideFilter }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column label="进场点" width="100px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.entryPrice }}</span>
        </template>
      </el-table-column>
      <el-table-column label="入场理由" width="150px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.entryReason }}</span>
        </template>
      </el-table-column>
      <el-table-column label="时间级别" width="100px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.timeLevel }}</span>
        </template>
      </el-table-column>
      <el-table-column label="仓位" width="100px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.position }}</span>
        </template>
      </el-table-column>
      <el-table-column label="止损点" align="center" width="100px">
        <template slot-scope="{row}">
          <span>{{ row.slPrice }}</span>
        </template>
      </el-table-column>
      <el-table-column label="止盈TP1" width="100px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.spPriceOne }}</span>
        </template>
      </el-table-column>
      <el-table-column label="止盈TP2" width="100px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.spPriceTwo }}</span>
        </template>
      </el-table-column>
      <el-table-column label="盈亏比" width="100px" align="center">
        <template slot-scope="{row}">
          <span>≈ {{ row.pnlRatio }}</span>
        </template>
      </el-table-column>
      <el-table-column label="已实现盈亏" width="100px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.realizedPnl }}</span>
        </template>
      </el-table-column>
      <el-table-column label="计价单位" width="100px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.valuationUnit | valuationUnitFilter }}</span>
        </template>
      </el-table-column>
      <el-table-column label="状态" class-name="status-col" width="110px">
        <template slot-scope="{row}">
          <el-tag :type="row.status | colorFilter">
            {{ row.status | statusFilter }}
          </el-tag>
        </template>
      </el-table-column>

      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template slot-scope="{row,$index}">
          <el-button type="primary" size="mini" @click="handleUpdate(row)">
            编辑
          </el-button>
          <el-button size="mini" type="danger" @click="handleDelete(row,$index)">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 数据列表分页 -->
    <pagination v-show="total>0" :total="total" :page.sync="listQuery.current" :limit.sync="listQuery.size" @pagination="getList" />
    <!-- 新增和编辑页面 -->
    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
      <el-form ref="dataForm" :rules="rules" :model="temp" label-position="right" label-width="100px">
        <el-row span="25">
          <el-col span="12">
            <el-form-item label="入场日期" prop="entryDate">
              <el-date-picker v-model="temp.entryDate" type="datetime" placeholder="请选择日期" />
            </el-form-item>
          </el-col>
          <el-col span="12">
            <el-form-item label="多空方向" prop="side">
              <el-select v-model="temp.side" class="filter-item">
                <el-option v-for="item in sideOptions" :key="item.key" :label="item.display_name" :value="item.key" />
              </el-select>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row span="25">
          <el-col span="12">
            <el-form-item label="品种代码" prop="symbol">
              <el-input class="form-item" v-model="temp.symbol" />
            </el-form-item>
          </el-col>
          <el-col span="12">
            <el-form-item label="入场点" prop="entryPrice">
              <el-input v-model="temp.entryPrice" />
            </el-form-item>
          </el-col>
        </el-row>
        <el-row span="25">
          <el-col span="24">
            <el-form-item label="入场理由" prop="entryReason">
              <el-input v-model="temp.entryReason" :autosize="{ minRows: 2, maxRows: 4}" type="textarea" />
            </el-form-item>
          </el-col>
        </el-row>
        <el-form-item label="时间级别" prop="timeLevel">
          <el-input v-model="temp.timeLevel" />
        </el-form-item>
        <el-row span="25">
          <el-col span="12">
            <el-form-item label="仓位" prop="position">
              <el-input v-model="temp.position" />
            </el-form-item>
          </el-col>
          <el-col span="12">
            <el-form-item label="止损点" prop="slPrice">
              <el-input v-model="temp.slPrice" />
            </el-form-item>
          </el-col>
        </el-row>
        <el-row span="25">
          <el-col span="12">
            <el-form-item label="止盈TP1" prop="spPriceOne">
              <el-input v-model="temp.spPriceOne" />
            </el-form-item>
          </el-col>
          <el-col span="12">
            <el-form-item label="止盈TP2" prop="spPriceTwo">
              <el-input v-model="temp.spPriceTwo" />
            </el-form-item>
          </el-col>
        </el-row>
        <el-row span="25">
          <el-col span="12">
            <el-form-item label="盈亏比" prop="pnlRatio">
              <el-input v-model="temp.pnlRatio" />
            </el-form-item>
          </el-col>
          <el-col span="12">
            <el-form-item label="已实现盈亏" prop="realizedPnl">
              <el-input v-model="temp.realizedPnl" />
            </el-form-item>
          </el-col>
        </el-row>
        <el-row span="25">
          <el-col span="12">
            <el-form-item label="计价单位" prop="valuationUnit">
              <el-select v-model="temp.valuationUnit" class="filter-item">
                <el-option v-for="item in valuationUnitOptions" :key="item.key" :label="item.display_name" :value="item.key" />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col span="12">
            <el-form-item label="状态">
              <el-select v-model="temp.status" class="filter-item">
                <el-option v-for="item in statusOptions" :key="item.key" :label="item.display_name" :value="item.key" />
              </el-select>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">
          取消
        </el-button>
        <el-button type="primary" @click="dialogStatus==='create'?createData():updateData()">
          保存
        </el-button>
      </div>
    </el-dialog>

    <el-dialog :visible.sync="dialogPvVisible" title="Reading statistics">
      <el-table :data="pvData" border fit highlight-current-row style="width: 100%">
        <el-table-column prop="key" label="Channel" />
        <el-table-column prop="pv" label="Pv" />
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="dialogPvVisible = false">Confirm</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { fetchList, fetchPv, createArticle, updateArticle, removeTradePlan } from '@/api/trade-plan'
import waves from '@/directive/waves' // waves directive
import { parseTime } from '@/utils'
import Pagination from '@/components/Pagination' // secondary package based on el-pagination

const statusOptions = [
  { key: 1, display_name: '挂单中' },
  { key: 2, display_name: '持仓中' },
  { key: 3, display_name: '已平仓' },
  { key: 4, display_name: '已撤销' }
]
const valuationUnitOptions = [
  { key: 1, display_name: 'USDT' },
  { key: 2, display_name: 'CNY' },
  { key: 3, display_name: 'USD' }
]
const sideOptions = [
  { key: 1, display_name: '多' },
  { key: 2, display_name: '空' },
  { key: 3, display_name: '现货' }
]

// arr to obj, such as { CN : "China", US : "USA" }
const statusKeyValue = statusOptions.reduce((acc, cur) => {
  acc[cur.key] = cur.display_name
  return acc
}, {})

const valuationUnitKeyValue = valuationUnitOptions.reduce((acc, cur) => {
  acc[cur.key] = cur.display_name
  return acc
}, {})

const sideKeyValue = sideOptions.reduce((acc, cur) => {
  acc[cur.key] = cur.display_name
  return acc
}, {})

export default {
  name: 'TradePlan',
  components: { Pagination },
  directives: { waves },
  filters: {
    colorFilter(value) {
      const colorMap = {
        1: 'success',
        2: 'danger',
        3: 'info',
        4: 'warning'
      }
      return colorMap[value]
    },
    statusFilter(status) {
      return statusKeyValue[status]
    },
    valuationUnitFilter(valuationUnit) {
      return valuationUnitKeyValue[valuationUnit]
    },
    sideFilter(side) {
      return sideKeyValue[side]
    }
  },
  data() {
    return {
      tableKey: 0,
      list: null,
      total: 0,
      listLoading: true,
      listQuery: {
        current: 1,
        size: 10,
        importance: undefined,
        symbol: undefined,
        status: undefined,
        side: undefined,
        sort: '+id'
      },
      importanceOptions: [1, 2, 3],
      sortOptions: [{ label: 'ID Ascending', key: '+id' }, { label: 'ID Descending', key: '-id' }],
      statusOptions,
      valuationUnitOptions,
      sideOptions,
      showReviewer: false,
      temp: {
        id: undefined,
        entryDate: new Date(),
        symbol: '',
        timeLevel: '',
        entryPrice: null,
        side: '',
        entryReason: '',
        position: null,
        slPrice: null,
        spPriceOne: null,
        spPriceTwo: null,
        pnlRatio: null,
        realizedPnl: null,
        valuationUnit: null,
        status: 1
      },
      dialogFormVisible: false,
      dialogStatus: '',
      textMap: {
        update: '编辑交易计划',
        create: '创建交易计划'
      },
      dialogPvVisible: false,
      pvData: [],
      rules: {
        type: [{ required: true, message: 'type is required', trigger: 'change' }],
        timestamp: [{ type: 'date', required: true, message: 'timestamp is required', trigger: 'change' }],
        title: [{ required: true, message: 'title is required', trigger: 'blur' }]
      },
      downloadLoading: false
    }
  },
  created() {
    this.getList()
  },
  methods: {
    getList() {
      this.listLoading = true
      fetchList(this.listQuery).then(response => {
        this.listLoading = false
        this.list = response.data.records
        this.total = response.data.total
      })
    },
    handleFilter() {
      this.listQuery.current = 1
      this.getList()
    },
    handleModifyStatus(row, status) {
      this.$message({
        message: '操作Success',
        type: 'success'
      })
      row.status = status
    },
    sortChange(data) {
      const { prop, order } = data
      if (prop === 'id') {
        this.sortByID(order)
      }
    },
    sortByID(order) {
      if (order === 'ascending') {
        this.listQuery.sort = '+id'
      } else {
        this.listQuery.sort = '-id'
      }
      this.handleFilter()
    },
    resetTemp() {
      this.temp = {
        id: undefined,
        entryDate: new Date(),
        symbol: '',
        timeLevel: '',
        entryPrice: null,
        side: '',
        entryReason: '',
        position: null,
        slPrice: null,
        spPriceOne: null,
        spPriceTwo: null,
        pnlRatio: null,
        realizedPnl: null,
        valuationUnit: null,
        status: 1
      }
    },
    handleCreate() {
      this.resetTemp()
      this.dialogStatus = 'create'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    createData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.temp.id = parseInt(Math.random() * 100) + 1024 // mock a id
          this.temp.author = 'vue-element-admin'
          createArticle(this.temp).then(() => {
            this.list.unshift(this.temp)
            this.dialogFormVisible = false
            this.$notify({
              title: 'Success',
              message: 'Created Successfully',
              type: 'success',
              duration: 2000
            })
          })
        }
      })
    },
    handleUpdate(row) {
      this.temp = Object.assign({}, row) // copy obj
      this.temp.timestamp = new Date(this.temp.timestamp)
      this.dialogStatus = 'update'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    updateData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          const tempData = Object.assign({}, this.temp)
          tempData.timestamp = +new Date(tempData.timestamp) // change Thu Nov 30 2017 16:41:05 GMT+0800 (CST) to 1512031311464
          updateArticle(tempData).then(() => {
            const index = this.list.findIndex(v => v.id === this.temp.id)
            this.list.splice(index, 1, this.temp)
            this.dialogFormVisible = false
            this.$notify({
              title: 'Success',
              message: 'Update Successfully',
              type: 'success',
              duration: 2000
            })
          })
        }
      })
    },
    handleDelete(row, index) {
      removeTradePlan(row.id).then(response => {
        this.$notify({
          title: 'Success',
          message: 'Delete Successfully',
          type: 'success',
          duration: 2000
        })
        this.list.splice(index, 1)
      })
    },
    handleFetchPv(pv) {
      fetchPv(pv).then(response => {
        this.pvData = response.data.pvData
        this.dialogPvVisible = true
      })
    },
    handleDownload() {
      this.downloadLoading = true
      import('@/vendor/Export2Excel').then(excel => {
        const tHeader = ['timestamp', 'title', 'type', 'importance', 'status']
        const filterVal = ['timestamp', 'title', 'type', 'importance', 'status']
        const data = this.formatJson(filterVal)
        excel.export_json_to_excel({
          header: tHeader,
          data,
          filename: 'table-list'
        })
        this.downloadLoading = false
      })
    },
    formatJson(filterVal) {
      return this.list.map(v => filterVal.map(j => {
        if (j === 'timestamp') {
          return parseTime(v[j])
        } else {
          return v[j]
        }
      }))
    },
    getSortClass: function(key) {
      const sort = this.listQuery.sort
      return sort === `+${key}` ? 'ascending' : 'descending'
    }
  }
}
</script>
