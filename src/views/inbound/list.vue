<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input  v-model="listQuery.title"  placeholder="po" style="width: 200px;" class="filter-item" @keyup.enter.native="handleFilter" />
      <el-input  v-model="listQuery.sku"  placeholder="sku" style="width: 200px;" class="filter-item" @keyup.enter.native="handleFilter" />
      <el-button v-waves class="filter-item" type="primary" icon="el-icon-search" @click="handleFilter">
        查询
      </el-button>
      <el-button class="filter-item" style="margin-left: 10px;" type="primary" icon="el-icon-edit" @click="handleCreate">
        新增库存
      </el-button>
      <el-button class="filter-item" style="margin-left: 10px;" type="primary" icon="el-icon-edit" @click="handleCreate">
        导入
      </el-button>      
    </div>

    <el-table :key="tableKey" v-loading="listLoading" :data="list" border fit highlight-current-row style="width: 85%;" @sort-change="sortChange">
      <el-table-column label="PO" prop="id" sortable="custom" align="center" width="120" :class-name="getSortClass('po')">
        <template slot-scope="scope">
          <span>{{ scope.row.po }}</span>
        </template>
      </el-table-column>
      <el-table-column label="SKU" width="150px" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.sku }}</span>
        </template>
      </el-table-column>
      <el-table-column label="入仓号" width="150px">
        <template slot-scope="scope">
          <span>{{ scope.row.inboundNo }}</span>
        </template>
      </el-table-column>
      <el-table-column label="物料号" width="110px" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.customsMeterialNo }}</span> 
        </template>
      </el-table-column>
      <el-table-column label="总库存毛重" width="110px" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.stockGw }}</span> 
        </template>
      </el-table-column>
      <el-table-column label="供应商名称" width="110px" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.supplierName }}</span> 
        </template>
      </el-table-column>
      <el-table-column label="so" width="110px" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.so }}</span> 
        </template>
      </el-table-column>
      <el-table-column label="收货日期" width="110px" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.rcvdDate }}</span> 
        </template>
      </el-table-column>            
      <el-table-column label="实收箱数" width="110px" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.rcvdCtns }}</span> 
        </template>
      </el-table-column>
      <el-table-column label="实收件数" width="110px" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.rcvdPcs }}</span> 
        </template>
      </el-table-column>
      <el-table-column label="报关单号" width="110px" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.merchName }}</span> 
        </template>
      </el-table-column>
      <el-table-column label="一箱几件" width="110px" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.itemsPerBox }}</span> 
        </template>
      </el-table-column> 

      <el-table-column label="Actions" align="center" width="200" class-name="small-padding fixed-width">
        <template slot-scope="{row}">
          <el-button type="primary" size="mini"  @click="handleUpdate(row)">
            修改
          </el-button>
          <el-button type="primary" size="mini" align="center" @click="handleUpdate(row)">
            扣减
          </el-button>

        </template>
      </el-table-column>
    </el-table>

    <pagination v-show="total>0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit" @pagination="getList" />

    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
      <el-form ref="dataForm" :rules="rules" :model="temp" label-position="left" label-width="120px" style="width: 1000px; margin-left:30px;">
        <el-row>
          <el-col :span="10" >
            <el-form-item label="PO:" prop="title" class="postInfo-container-item">
              <el-input v-model="temp.po" />
            </el-form-item>
          </el-col>
          <el-col :span="10">
            <el-form-item label="SKU:" prop="title" class="postInfo-container-item">
              <el-input v-model="temp.sku" />
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="10" >
            <el-form-item label="入仓号:" prop="title" class="postInfo-container-item">
              <el-input v-model="temp.inboundNo" />
            </el-form-item>
          </el-col>
          <el-col :span="10">
            <el-form-item label="物料号:" prop="title" class="postInfo-container-item">
              <el-input v-model="temp.title" />
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="10" >
            <el-form-item label="报关单件净重:" prop="title" class="postInfo-container-item">
              <el-input v-model="temp.title" />
            </el-form-item>
          </el-col>
          <el-col :span="10">
            <el-form-item label="每箱件数:" prop="title" class="postInfo-container-item">
              <el-input v-model="temp.title" />
            </el-form-item>
          </el-col>
        </el-row>
        
      </el-form>

      <div slot="footer"  align="center" class="dialog-footer">
        <el-button  @click="dialogFormVisible = false">
          Cancel
        </el-button>
        <el-button type="primary" @click="dialogStatus==='create'?createData():updateData()">
          Confirm
        </el-button>
      </div>
    </el-dialog>

  </div>
  
</template>


<script>
import { fetchList, fetchPv, createArticle, updateArticle } from '@/api/article'
import waves from '@/directive/waves' // waves directive
import { parseTime } from '@/utils'
import Pagination from '@/components/Pagination' // secondary package based on el-pagination

const calendarTypeOptions = [
  { key: 'CN', display_name: 'China' },
  { key: 'US', display_name: 'USA' },
  { key: 'JP', display_name: 'Japan' },
  { key: 'EU', display_name: 'Eurozone' }
]

// arr to obj, such as { CN : "China", US : "USA" }
const calendarTypeKeyValue = calendarTypeOptions.reduce((acc, cur) => {
  acc[cur.key] = cur.display_name
  return acc
}, {})

export default {
  name: 'ComplexTable',
  components: { Pagination },
  directives: { waves },
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: 'success',
        draft: 'info',
        deleted: 'danger'
      }
      return statusMap[status]
    },
    typeFilter(type) {
      return calendarTypeKeyValue[type]
    }
  },
  data() {
    return {
      tableKey: 0,
      list: null,
      total: 0,
      listLoading: true,  
      listQuery: {
        page: 1,
        limit: 20,
        importance: undefined,
        title: undefined,
        type: undefined,
        sort: '123123'
      },
      importanceOptions: [1, 2, 3],
      calendarTypeOptions,
      sortOptions: [{ label: 'ID Ascending', key: '+id' }, { label: 'ID Descending', key: '-id' }],
      statusOptions: ['published', 'draft', 'deleted'],
      showReviewer: false,
      temp: {
        id: undefined,
        importance: 1,
        remark: '',
        timestamp: new Date(),
        title: '',
        type: '',
        status: 'published'
      },
      dialogFormVisible: false,
      dialogStatus: '',
      textMap: {
        update: 'Edit',
        create: 'Create'
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
        this.list = response.data.items
        this.total = response.data.total
        // Just to simulate the time of the request
        setTimeout(() => {
          this.listLoading = false
        }, 1.5 * 1000)
      })
    },
    handleFilter() {
      this.listQuery.page = 1
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
        importance: 1,
        remark: '',
        timestamp: new Date(),
        title: '',
        status: 'published',
        type: ''
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
            for (const v of this.list) {
              if (v.id === this.temp.id) {
                const index = this.list.indexOf(v)
                this.list.splice(index, 1, this.temp)
                break
              }
            }
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
    handleDelete(row) {
      this.$notify({
        title: 'Success',
        message: 'Delete Successfully',
        type: 'success',
        duration: 2000
      })
      const index = this.list.indexOf(row)
      this.list.splice(index, 1)
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
        const data = this.formatJson(filterVal, this.list)
        excel.export_json_to_excel({
          header: tHeader,
          data,
          filename: 'table-list'
        })
        this.downloadLoading = false
      })
    },
    formatJson(filterVal, jsonData) {
      return jsonData.map(v => filterVal.map(j => {
        if (j === 'timestamp') {
          return parseTime(v[j])
        } else {
          return v[j]
        }
      }))
    },
    getSortClass: function(key) {
      const sort = this.listQuery.sort
      return sort === `+${key}`
        ? 'ascending'
        : sort === `-${key}`
          ? 'descending'
          : ''
    }
  }
}
</script>
