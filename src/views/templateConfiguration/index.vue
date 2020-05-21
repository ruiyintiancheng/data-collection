/*
 * @Author: 1k 
 * @Date: 2019-08-02 15:02:12 
 * @Last Modified by: wk
 * @Last Modified time: 2020-05-21 10:17:36
 * @Description:  excel模板配置
 */
 <template>
  <div class="notifacation-ma">
    <div class="base-row">
      <div class="row-botton clearfix">
        <div class="row-title">
          <svg-icon icon-class="search" />
          <span>筛选查询</span>
        </div>
        <div class="row-option">
          <!-- <a href="javascript:void(0)" class="button" @click="searchOption">查询</a> -->
          <el-button icon="el-icon-search"
                     @click="searchOption"
                     type="primary">查询</el-button>
          <a @click="searchToggle=false"
             v-if="searchToggle">
            <svg-icon icon-class="up" />&nbsp;收起</a>
          <a @click="searchToggle=true"
             v-else>
            <svg-icon icon-class="down" />&nbsp;展开</a>
        </div>
      </div>
      <div v-show="searchToggle"
           class="form-search">
        <el-form ref="form"
                 :inline="true"
                 :model="searchData"
                 class="demo-table-expand">
          <div class="input-both-3">
            <el-form-item>
              <span class="input-label">模板名称</span>
              <el-input size="medium "
                        class="form-input"
                        v-model="searchData.collectTemplateName"
                        clearable></el-input>
            </el-form-item>
          </div>
          <div class="input-both-3">
            <el-form-item>
              <span class="input-label">表名</span>
              <el-input size="medium "
                        class="form-input"
                        v-model="searchData.tableName"
                        clearable></el-input>
            </el-form-item>
          </div>
          <!-- <div class="input-both-3">
            <el-form-item>
              <span class="input-label">通知类型:</span>
              <el-select class="form-input"
                         v-model="searchData.noticeType"
                         size="small"
                         clearable>
                <el-option label="全部"
                           value="0"></el-option>
                <el-option label="开始计算"
                           value="1"></el-option>
                <el-option label="计算完成"
                           value="2"></el-option>
                <el-option label="计算失败"
                           value="3"></el-option>
              </el-select>
            </el-form-item>
          </div> -->
        </el-form>
      </div>
    </div>
    <div class="base-row">
      <div class="row-botton clearfix">
        <div class="row-title">
          <svg-icon icon-class="ul" />
          <span>数据列表</span>
        </div>
        <div class="row-option">
          <el-button icon="el-icon-upload2"
                     @click="openUpdateFile">模板上传</el-button>
          <el-button icon="el-icon-plus"
                     @click="openColumnStep">添加</el-button>
          <a @click="tableToggle=false"
             v-if="tableToggle">
            <svg-icon icon-class="up" />&nbsp;收起</a>
          <a @click="tableToggle=true"
             v-else>
            <svg-icon icon-class="down" />&nbsp;展开</a>
        </div>
      </div>
      <div v-show="tableToggle">
        <!-- 表格 -->
        <el-table ref="basicTable"
                  :data="configData"
                  style="width: 100%;"
                  v-loading="listLoading"
                  element-loading-text="给我一点时间"
                  :height="getDefaultHeight()"
                  stripe
                  border
                  highlight-current-row
                  fit
                  header-cell-class-name="ai-el-table-header">
          <el-table-column width=50
                           type="index"
                           align="center"
                           fixed="left"
                           label="序号"></el-table-column>
          <el-table-column :min-width="100"
                           align='center'
                           label="模板名称">
            <template slot-scope="scope">
              {{scope.row.collectTemplateName}}
            </template>
          </el-table-column>
          <el-table-column :min-width="100"
                           align='center'
                           label="数据源名称">
            <template slot-scope="scope">
              {{scope.row.dsName}}
            </template>
          </el-table-column>
          <el-table-column width="80"
                           align='center'
                           label="用户名">
            <template slot-scope="scope">
              {{scope.row.schemaName}}
            </template>
          </el-table-column>
          <el-table-column :min-width="100"
                           align='center'
                           label="表名">
            <template slot-scope="scope">
              {{scope.row.tableName}}
            </template>
          </el-table-column>
          <el-table-column width="90"
                           align='center'
                           label="模板类型">
            <template slot-scope="scope">
              <!-- {{options.collectTemplateType[scope.row.field_convert_map.collectTemplateType]}} -->
              {{scope.row.field_convert_map.collectTemplateType}}
            </template>
          </el-table-column>
          <el-table-column width="240"
                           label="操作"
                           fixed="right"
                           align="center">
            <template slot-scope="scope">
              <el-button plain
                         type="info"
                         size="mini"
                         @click="addParam(scope.row)">查看</el-button>
              <el-button plain
                         type="primary"
                         size="mini"
                         @click="openColumn(scope.row)">修改</el-button>
              <el-button type="danger"
                         plain
                         size="mini"
                         @click="handleDelete(scope.row)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
        <el-pagination background
                       @size-change="handleSizeChange"
                       @current-change="handleCurrentChange"
                       :current-page="pageNo"
                       :total="total"
                       layout="total, sizes, prev, pager, next, jumper"
                       :page-sizes="[10,15,20]"
                       :page-size="pageSize">
        </el-pagination>
      </div>
    </div>

    <el-dialog :close-on-click-modal="false"
               v-el-drag-dialog
               :title="updateFormData.collectTemplateName"
               append-to-body
               :visible.sync="inputParamFormVisible"
               width="80%"
               custom-class="dialog-default">
      <div class="dialog-contant-default-addOrUpdate">
        <div>
          <!-- sheet索引:
          <el-input :placeholder="updateFormData.sheetIndex "
                    :disabled="true"
                    size="small">
          </el-input> -->
          <el-form ref="form"
                   :inline="true"
                   :model="searchData"
                   class="demo-table-expand">
            <el-form-item>
              <span class="input-label">数据源名称:</span>
              <el-input size="small"
                        class="form-input"
                        v-model="updateFormData.dsName"
                        clearable
                        :disabled="true"></el-input>
            </el-form-item>
            <el-form-item>
              <span class="input-label">用户名:</span>
              <el-input size="small"
                        class="form-input"
                        v-model="updateFormData.schemaName"
                        clearable
                        :disabled="true"></el-input>
            </el-form-item>
            <el-form-item>
              <span class="input-label">表名:</span>
              <el-input size="small"
                        class="form-input"
                        v-model="updateFormData.tableName"
                        clearable
                        :disabled="true"></el-input>
            </el-form-item>
            <el-form-item>
              <span class="input-label">模板类型:</span>
              <el-input size="small"
                        class="form-input"
                        v-model="options.collectTemplateType[updateFormData.collectTemplateType]"
                        clearable
                        :disabled="true"></el-input>
            </el-form-item>
            <el-form-item v-show="updateFormData.collectTemplateType === '2'">
              <span class="input-label">sheet索引:</span>
              <el-input size="small"
                        class="form-input"
                        v-model="updateFormData.sheetIndex"
                        clearable
                        :disabled="true"></el-input>
            </el-form-item>

            <el-form-item v-show="updateFormData.collectTemplateType === '2'">
              <span class="input-label">开始行数:</span>
              <el-input size="small"
                        class="form-input"
                        v-model="updateFormData.startRow"
                        clearable
                        :disabled="true"></el-input>
            </el-form-item>
            <el-form-item v-show="updateFormData.collectTemplateType === '1'">
              <span class="input-label">分隔符:</span>
              <el-input size="small"
                        class="form-input"
                        v-model="updateFormData.delimiter"
                        clearable
                        :disabled="true"></el-input>
            </el-form-item>
            <el-form-item v-show="updateFormData.collectTemplateType === '1'">
              <span class="input-label">文本识别符:</span>
              <el-input size="small"
                        class="form-input"
                        v-model="updateFormData.textIdentifier"
                        clearable
                        :disabled="true"></el-input>
            </el-form-item>
          </el-form>
        </div>
        <el-table ref="basicTable"
                  :data="updateFormData.templateColumns"
                  style="width: 100%;"
                  v-loading="listLoading"
                  element-loading-text="给我一点时间"
                  :height="getDefaultHeight()"
                  stripe
                  border
                  highlight-current-row
                  fit
                  header-cell-class-name="ai-el-table-header">
          <el-table-column width=50
                           type="index"
                           align="center"
                           fixed="left"></el-table-column>
          <el-table-column :min-width="150"
                           align='center'
                           label="字段名">
            <template slot-scope="scope">
              {{scope.row.columnName}}
            </template>
          </el-table-column>
          <el-table-column :min-width="150"
                           align='center'
                           label="字段顺序">
            <template slot-scope="scope">
              {{scope.row.columnOrder}}
            </template>
          </el-table-column>
          <el-table-column :min-width="150"
                           align='center'
                           label="数据采集模板类型">
            <template slot-scope="scope">
              {{scope.row.columnParseType}}
            </template>
          </el-table-column>
        </el-table>
      </div>
      <div slot="footer"
           class="dialog-footer-addOrUpdate">
        <el-button @click="inputParamFormVisible = false, cancel()">关闭</el-button>
        <!-- <el-button type="primary"
                   @click="saveOperate()">保存</el-button> -->
      </div>
    </el-dialog>
    <column-step ref="columnStep"></column-step>
    <excel-file ref="excelFile"></excel-file>
    <modification ref="modification"
                  :collectTemplateId='collectTemplateId'
                  :tableId='tableId'></modification>
  </div>
</template>
<script>
import columnStep from './components/templateModification'
import modification from './components/modification'
import excelFile from './components/excelFile'
import { baseRequest } from '@/api/base'
import { baseSearch } from '@/api/base'
// import { saveUpdate } from '@/utils/validate'
// var url = '/taskNotice/getDeployList' // 获取该任务类的输入参数
export default {
  directives: {},
  name: 'templateConfigurationIndex',
  components: {
    columnStep,
    modification,
    excelFile
  },
  mounted() {

  },
  created() {
    this.searchOption()
  },
  data() {
    return {
      tableId: null,
      collectTemplateId: null,
      tableToggle: true,
      searchToggle: true,
      userIdAll: null,
      pageNo: 1,
      total: null,
      pageSize: null,
      listLoading: false, // 加载圆圈是否显示
      formVisible: false,
      configData: [],
      dialogTitle: { // 弹框标题
        1: '查看',
        2: '修改'
      },
      searchData: { // 查询条件字段
        userName: '',
        noticeType: '',
        noticeMode: ''
      },
      updateFormData: { // 添加修改表单数据

      },
      inputParamFormVisible: false, // 输入参数添加修改弹窗
      operateStatus: 1,
      // addRules: {
      //   userId: [{
      //     required: true, message: '请选择用户(前两项必填)'
      //   }],
      //   noticeType: [{
      //     required: true, message: '请选择通知类型'
      //   }],
      //   noticeMode: [{
      //     required: true, message: '请选择通知方式'
      //   }]
      // },
      options: {
        'collectTemplateType': {
          '1': 'csv导入',
          '2': 'excel导入'
        }
      }
    }
  },
  props: {
    paramData: Object
  },
  methods: {
    getDefaultHeight() { // 获取表格高度
      return this.$store.state.app.containHeight - 230
    },
    openDialog() {
      this.formVisible = true
      this.searchOption()
    },
    searchOption(page) {
      this.listLoading = true
      if (!page) {
        this.pageNo = 1
      }
      this.searchData.pageNo = this.pageNo
      this.searchData.pageSize = this.pageSize
      baseSearch('/basic/t04ExcelImportTemplate/selects', this.searchData).then(response => {
        this.configData = response.data.item
        this.total = response.data.total
        this.pageSize = response.data.pageSize
        this.listLoading = false
      })
    },
    handleSizeChange(val) { // 分页
      this.pageSize = val
      this.searchOption()
    },
    handleCurrentChange(val) { // 分页
      this.pageNo = val
      this.searchOption(true)
    },
    addParam(row) {
      this.operateStatus = 1
      // for (var i in this.updateFormData) {
      //   this.updateFormData[i] = ''
      // }
      // delete this.updateFormData.paramId
      const param = {
        collectTemplateId: row.collectTemplateId
      }
      baseRequest('/basic/t04ExcelImportTemplate/select', param).then(request => {
        this.updateFormData = request.data.item
      })
      this.inputParamFormVisible = true
    },
    // saveOperate() {
    //   const param = this.updateFormData
    //   param.deployNodeId = this.paramData.deployNodeId
    //   param.deployFlowId = this.paramData.deployFlowId
    //   saveUpdate('/taskNotice/add', this.updateFormData, this.addRules, this.$refs.addForm).then(() => {
    //     this.searchOption()
    //     this.$Message.success('操作成功')
    //     this.inputParamFormVisible = false
    //     this.updateFormData.userId = ''
    //     this.userIdAll = ''
    //   })
    // },
    handleDelete(row) {
      this.$confirm('此操作将永久删除该资源, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        const param = {
          collectTemplateId: row.collectTemplateId
        }
        baseRequest('/basic/t04ExcelImportTemplate/delete', param).then(request => {
          this.searchOption()
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
        })
      })
    },
    // 获取userId 用户信息
    requestInformation() {
      const userParam = this.updateFormData
      userParam.deployNodeId = this.paramData.deployNodeId
      userParam.deployFlowId = this.paramData.deployFlowId
      if (userParam.noticeType !== '' && userParam.noticeMode !== '') {
        console.log(userParam)

        baseRequest('/basic/T00User/getAllUsers', userParam).then(response => {
          this.userIdAll = response.data.item
        })
      }
    },
    cancel() {
      this.updateFormData.userId = ''
      this.userIdAll = ''
    },
    // 添加
    openColumnStep() {
      this.$refs.columnStep.colmunListVisable = true
      this.$refs.columnStep.clearForm()
      // }
    },
    openUpdateFile() {
      this.$refs.excelFile.inputParamFormVisible = true
      this.$refs.excelFile.clearForm()
    },
    openColumn(row) {
      this.collectTemplateId = row.collectTemplateId
      this.tableId = row.tableId
      this.$refs.modification.openModification(row)
      // }
    }
  },
  watch: {
  }
}
</script>
<style lang="scss" scoped>
.notifacation-ma .form-search .input-both-3 {
  width: 32.3333%;
}
.notifacation-ma .base-row {
  margin: 5px 0;
}
</style>
<style lang="scss" >
.dialog-default {
  .dialog-contant-default-addOrUpdate {
    padding-top: 10px;
  }
}
</style>