<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input v-model="listQuery.po" placeholder="po" style="width: 200px;" class="filter-item" @keyup.enter.native="handleFilter" />
      <el-input v-model="listQuery.sku" placeholder="sku" style="width: 200px;" class="filter-item" @keyup.enter.native="handleFilter" />
      <el-button v-waves class="filter-item" type="primary" icon="el-icon-search" @click="handleFilter">
        查询
      </el-button>
      <el-button class="filter-item" style="margin-left: 10px;" type="primary" icon="el-icon-edit" @click="handleCreate">
        新增库存
      </el-button>
      <!-- <el-button :loading="loading" class="filter-item" style="margin-left:10px;" type="primary" icon="el-icon-edit" @click="handleUpload">
        导入
      </el-button> -->
      <br/>


<!-- <el-upload
 class="upload-demo"
 ref="upload"
 action="doUpload"
 :limit="1"
 :file-list="fileList"
 :before-upload="beforeUpload"
 :on-success="uploadSucess">
 <el-button slot="trigger" size="small" type="primary">选取文件</el-button>
 <!-- <a href="./static/moban.xlsx" rel="external nofollow" download="模板"><el-button size="small" type="success">下载模板</el-button></a> -->
 <!-- <el-button style="margin-left: 10px;" size="small" type="success" @click="submitUpload">上传到服务器</el-button> -->
 <!-- <div slot="tip" class="el-upload__tip">只能上传excel文件，且不超过5MB</div> -->
 <!-- <div slot="tip" class="el-upload-list__item-name">{{fileName}}</div>

</el-upload> 
      <br/>  -->
<!-- <span slot="footer" class="dialog-footer">
 <el-button @click="visible = false">取消</el-button>
 <el-button type="primary" @click="submitUpload()">确定</el-button>
</span> -->

      <!-- <el-upload
      style="display: inline; margin-left: 10px;margin-right: 10px;"
      action="http://localhot:9080/wms-core/stock/upload"
      ref="fileupload"
      multiple="false"
      name="excelFile"
      :
      :headers="headers"
      :on-preview="handlePreview"
      :on-error="uploadFalse"
      :on-success="uploadSucess"
      :auto-upload="false"
      :show-file-list="false"
      :before-upload="beforeUpload">
       <el-button slot="trigger" size="small" type="primary">选取文件</el-button>

      <el-button class="filter-item" style="margin-left:10px;"  type="primary" icon="el-icon-edit"  @click="handleUpload">导入</el-button>
      </el-upload> -->

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
          <el-button type="primary" size="mini" @click="handleUpdate(row)">
            修改
          </el-button>
          <el-button type="primary" size="mini" align="center" @click="handleUpdate(row)">
            出库
          </el-button>

        </template>
      </el-table-column>
    </el-table>

    <pagination v-show="total>0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit" @pagination="getList" />

    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible" width="60%" >
      <el-form ref="dataForm" :rules="rules" :model="temp" label-position="left" label-width="120px" style="width: 1200px; margin-left:30px;">
        <el-row>
          <el-col :span="80">
            <el-form-item label="PO:" prop="po" >
              <el-input v-model="temp.po" />
            </el-form-item>
          </el-col>
          <el-col :span="80">
            <el-form-item label="SKU:" prop="sku" >
              <el-input v-model="temp.sku" />
            </el-form-item>
          </el-col>
          <el-col :span="80">
            <el-form-item label="SO:" prop="so" >
              <el-input v-model="temp.so" />
            </el-form-item>
          </el-col>          
        </el-row>
        <el-row>
          <el-col :span="80">
            <el-form-item label="物料号:"  >
              <el-input v-model="temp.customsMeterialNo" />
            </el-form-item>
          </el-col>
          <el-col :span="80">
            <el-form-item label="实收箱数:" >
              <el-input v-model="temp.rcvdCtns" />
            </el-form-item>
          </el-col>
          <el-col :span="80">
            <el-form-item label="实收件数:" >
              <el-input v-model="temp.rcvdPcs" />
            </el-form-item>
          </el-col>            
        </el-row>
        <el-row>
          <el-col :span="80">
            <el-form-item label="单箱毛重:"  >
              <el-input v-model="temp.gwPerBoxActul" />
            </el-form-item>
          </el-col>
          <el-col :span="80">
            <el-form-item label="单箱净重:">
              <el-input v-model="temp.custsDeclaPieceWeigh" />
            </el-form-item>
          </el-col>
          <el-col :span="80">
            <el-form-item label="报关单件净重:"  >
              <el-input v-model="temp.boxLengthActul" />
            </el-form-item>
          </el-col>                  
        </el-row>
        <el-row>
          <el-col :span="80">
            <el-form-item label="长(CM):" >
              <el-input v-model="temp.boxLengthActul" />
            </el-form-item>
          </el-col>
          <el-col :span="80">
            <el-form-item label="宽(CM):" >
              <el-input v-model="temp.boxWidthActul" />
            </el-form-item>
          </el-col>
          <el-col :span="80">
            <el-form-item label="高(CM):" >
              <el-input v-model="temp.boxHighActul" />
            </el-form-item>
          </el-col>                  
        </el-row>
        <el-row>
          <el-col :span="80">
            <el-form-item label="成交数量:"  >
              <el-input v-model="temp.declaCount" />
            </el-form-item>
          </el-col>
          <el-col :span="80">
            <el-form-item label="申报单位:" >
              <el-input v-model="temp.declaUnit" />
            </el-form-item>
          </el-col>
          <el-col :span="80">
            <el-form-item label="申报单价:"  >
              <el-input v-model="temp.declaUnitPrice" />
            </el-form-item>
          </el-col>                  
        </el-row>
        <el-row>
          <el-col :span="80">
            <el-form-item label="币制:"  >
              <el-input v-model="temp.declaCurrency" />
            </el-form-item>
          </el-col>
          <el-col :span="80">
            <el-form-item label="商品编码:"  >
              <el-input v-model="temp.customsMerchNo" />
            </el-form-item>
          </el-col>
          <el-col :span="80">
            <el-form-item label="商品名称:"  >
              <el-input v-model="temp.merchName" />
            </el-form-item>
          </el-col>                  
        </el-row>
        <el-row>
          <el-col :span="80">
            <el-form-item label="原产国:"  >
              <el-input v-model="temp.originCountry" />
            </el-form-item>
          </el-col>
          <el-col :span="80">
            <el-form-item label="目的国:"  >
              <el-input v-model="temp.destCountry" />
            </el-form-item>
          </el-col>
          <el-col :span="80">
            <el-form-item label="仓位:" >
              <el-input v-model="temp.warehousePosition" />
            </el-form-item>
          </el-col>                  
        </el-row>
        <el-row>
          <el-col :span="80">
            <el-form-item label="入仓号"  >
              <el-input v-model="temp.inboundNo" />
            </el-form-item>
          </el-col>                
          <el-col :span="80">
            <el-form-item label="供应商名称"  >
              <el-input v-model="temp.supplierName" />
            </el-form-item>
          </el-col>                
          <el-col :span="80">
            <el-form-item label="备注:"  >
              <el-input v-model="temp.remarks" />
            </el-form-item>
          </el-col>
          
        </el-row>

      </el-form>

      <div slot="footer" align="center" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">
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
import { fetchList, fetchPv, addStock, updateArticle } from '@/api/article'
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
        po: [{ required: true, message: 'title is required', trigger: 'blur' }],
        sku: [{ required: true, message: 'title is required', trigger: 'blur' }]
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
          addStock(this.temp).then(() => {
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
    beforeUpload(file) {
      alert('aa');
  //debugger
  console.log(file,'文件');
  this.files = file;
  const extension = file.name.split('.')[1] === 'xls'
  const extension2 = file.name.split('.')[1] === 'xlsx'
  const isLt2M = file.size / 1024 / 1024 < 5
  if (!extension && !extension2) {
   this.$message.warning('上传模板只能是 xls、xlsx格式!')
   return
  }
  if (!isLt2M) {
   this.$message.warning('上传模板大小不能超过 5MB!')
   return
  }
  this.fileName = file.name;
  return false // 返回false不会自动上传
    },
    uploadSucess({ results, header }) {
            alert('uploadSucess') ;
      this.tableData = results
      this.tableHeader = header
    },
    uploadFalse(response, file, fileList) {
      this.$message.error("文件上传失败！");
    },
    handleUpload(){
      alert('aaaa');
      alert('bb'+$refs.fileupload.filename);
      this.$refs.fileupload.submit();
    },
    handlePreview(){
      alert('handlerpre');
    },
    uploadFalse(){
      alert('uploadFalse') ;
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

<style scoped>
.excel-upload-input{
  display: none;
  z-index: -9999;
}

</style>