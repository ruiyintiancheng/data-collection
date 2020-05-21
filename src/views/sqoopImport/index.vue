/*
 * @Author: 1k 
 * @Date: 2019-08-23 14:40:21 
 * @Last Modified by: lk
 * @Last Modified time: 2020-05-09 15:35:01
 * @Description: sqoop 配置 
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
                 size="medium "
                 :inline="true"
                 :model="searchData"
                 class="demo-table-expand">

          <el-form-item style="margin-left:60px">
            <span class="input-label">配置名称</span>
            <el-input size="medium "
                      class="form-input"
                      v-model="searchData.sqoopName"
                      clearable
                      style="width:255px"></el-input>
          </el-form-item>
          <el-form-item style="margin-left:60px">
            <span class="input-label">配置状态</span>
            <el-select class="form-input"
                       v-model="searchData.sqoopStatus"
                       size="medium "
                       clearable
                       style="width:255px"
                       placeholder="">
              <el-option label="等待导入"
                         value="0"></el-option>
              <el-option label="准备导入"
                         value="1"></el-option>
              <el-option label="导入中"
                         value="2"></el-option>
              <el-option label="导入完成"
                         value="3"></el-option>
              <el-option label="导入失败"
                         value="4"></el-option>

            </el-select>
          </el-form-item>
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
          <el-button icon="el-icon-plus"
                     @click="openAddSqoop">添加</el-button>
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
                  header-cell-class-name="ai-el-table-header">
          <el-table-column width=50
                           type="index"
                           align="center"
                           label="序号"></el-table-column>
          <el-table-column :min-width="100"
                           align='center'
                           label="配置名称">
            <template slot-scope="scope">
              {{scope.row.sqoopName}}
            </template>
          </el-table-column>
          <el-table-column :min-width="100"
                           align='center'
                           label="数据源名称">
            <template slot-scope="scope">
              {{scope.row.dsName}}
            </template>
          </el-table-column>
          <el-table-column :min-width="100"
                           align='center'
                           label="用户名">
            <template slot-scope="scope">
              {{scope.row.schemaName}}
            </template>
          </el-table-column>
          <el-table-column :min-width="100"
                           align='center'
                           label="源表名">
            <template slot-scope="scope">
              <el-tooltip :disabled="textJug(scope.row.tableName)"
                          :content="scope.row.tableName"
                          placement="top"
                          visible-arrow
                          effect="light">
                <div class="basic-table-column">
                  {{scope.row.tableName}}
                </div>
              </el-tooltip>
            </template>
          </el-table-column>
          <el-table-column :min-width="100"
                           align='center'
                           label="目标表名">
            <template slot-scope="scope">
              <el-tooltip :disabled="textJug(scope.row.targetTableName)"
                          :content="scope.row.targetTableName"
                          placement="top"
                          visible-arrow
                          effect="light">
                <div class="basic-table-column">
                  {{scope.row.targetTableName}}
                </div>
              </el-tooltip>
            </template>
          </el-table-column>
          <!-- <el-table-column :min-width="100"
                           align='center'
                           label="配置类型">
            <template slot-scope="scope">
              {{options.collectTemplateType[scope.row.collecttemplatetype]}}
            </template>
          </el-table-column> -->
          <el-table-column :min-width="100"
                           align='center'
                           label="配置类型">
            <template slot-scope="scope">
              {{options.sqoopStatus[scope.row.sqoopStatus] }}
            </template>
          </el-table-column>
          <el-table-column width="350"
                           label="操作"
                           fixed="right"
                           align="center">
            <template slot-scope="scope">
              <el-button plain
                         type="success"
                         size="mini"
                         v-if="scope.row.sqoopStatus==='0'"
                         @click="reimport(scope.row)">导入</el-button>
              <el-button plain
                         type="warning"
                         size="mini"
                         v-if="scope.row.sqoopStatus==='3'||scope.row.sqoopStatus==='4'"
                         style="margin-top:4px;margin-bottom:4px;"
                         @click="reimport(scope.row)">重新导入</el-button>
              <el-button plain
                         type="info"
                         size="mini"
                         v-if="scope.row.sqoopStatus==='3'||scope.row.sqoopStatus==='4'"
                         style="margin-top:4px;margin-bottom:4px;"
                         @click="addParam(scope.row)">查看</el-button>
              <el-button plain
                         type="primary"
                         size="mini"
                         v-if="scope.row.sqoopStatus==='0'||scope.row.sqoopStatus==='3'||scope.row.sqoopStatus==='4'"
                         style="margin-top:4px;margin-bottom:4px;"
                         @click="openColumn(scope.row)">修改</el-button>
              <el-button type="danger"
                         plain
                         size="mini"
                         v-if="scope.row.sqoopStatus==='0'||scope.row.sqoopStatus==='3'||scope.row.sqoopStatus==='4'"
                         style="margin-top:4px;margin-bottom:4px;"
                         @click="handleDelete(scope.row)">删除</el-button>
            </template>
          </el-table-column>
          <!-- <el-table-column :min-width="150"
                           align='center'
                           label="数据采集模板类型">
            <template slot-scope="scope">
              {{options.collectTemplateType[scope.row.collectTemplateType]}}
            </template>
          </el-table-column> -->
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
    <!-- <excel-file ref="excelFile"></excel-file> -->
    <!-- 添加修改 -->
    <add-sqoop v-if="addSqoopStatus"
               ref="addSqoop"></add-sqoop>
    <!-- 重新导入 -->
    <el-dialog :close-on-click-modal="false"
               v-el-drag-dialog
               title="重新导入"
               append-to-body
               :visible.sync="inputParamFormVisible"
               width="30%"
               custom-class="dialog-reimport">
      <div class="dialog-contant-default-addOrUpdate">
        <el-form ref="addForm"
                 :rules="addRules"
                 :model="updateFormData"
                 label-width="100px"
                 style="margin:45px 0 0 50px;">
          <el-form-item label="版本日期"
                        prop="verDt">
            <el-date-picker v-model="updateFormData.verDt"
                            type="date"
                            size="small"
                            value-format="yyyy-MM-dd"
                            style="width:200px"
                            placeholder="">
            </el-date-picker>
          </el-form-item>
          <el-form-item label="数据加载模式"
                        prop="sqoopMode">
            <el-select v-model="updateFormData.sqoopMode"
                       size="small"
                       clearable
                       placeholder="">
              <el-option label="追加"
                         value="0"></el-option>
              <el-option label="覆盖"
                         value="1"></el-option>
            </el-select>
          </el-form-item>
        </el-form>
      </div>
      <div slot="footer"
           class="dialog-footer-addOrUpdate">
        <el-button @click="inputParamFormVisible = false">取消</el-button>
        <el-button type="primary"
                   @click="saveOperate()">保存</el-button>
      </div>
    </el-dialog>

    <!-- 查看列表 -->
    <el-dialog :close-on-click-modal="false"
               v-el-drag-dialog
               title="流程配置信息"
               append-to-body
               :visible.sync="toConfigure"
               width="70%"
               custom-class="dialog-default">
      <div class="dialog-contant-default-addOrUpdate">
        <el-table ref="seeTable"
                  :data="toConfigureData"
                  style="width: 100%;"
                  v-loading="listLoading"
                  element-loading-text="给我一点时间"
                  :height="defaultHeight()"
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
          <el-table-column :min-width="150"
                           align='center'
                           label="流程名称">
            <template slot-scope="scope">
              {{scope.row.flowName}}
            </template>
          </el-table-column>
          <el-table-column :min-width="150"
                           align='center'
                           label="流程介绍">
            <template slot-scope="scope">
              {{scope.row.flowDesc}}
            </template>
          </el-table-column>
          <el-table-column :min-width="150"
                           align='center'
                           label="计算类型">
            <template slot-scope="scope">
              {{options.calStatus[scope.row.calStatus] }}
            </template>
          </el-table-column>
          <el-table-column :min-width="210"
                           label="操作"
                           align="center">
            <template slot-scope="scope">
              <el-button plain
                         type="info"
                         size="mini"
                         style="margin-top:4px;margin-bottom:4px;"
                         @click="viewChart(scope.row)">查看</el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <div slot="footer"
           class="dialog-footer-addOrUpdate">
        <el-button @click="toConfigure = false">关闭</el-button>
        <!-- <el-button type="primary"
                   @click="saveOperate()">保存</el-button> -->
      </div>
    </el-dialog>
    <flow-figure ref="flowFigure"></flow-figure>
  </div>
</template>
<script>
// import excelFile from './components/excelFile'

import { baseSearch } from '@/api/base'
import { baseRequest } from '@/api/base'
import { saveUpdate } from '@/utils/validate'
import addSqoop from './components/index'
import flowFigure from 'process-g6-ry'
export default {
  directives: {},
  name: 'templateFileRecordingIndex',
  components: {
    addSqoop,
    flowFigure
  },
  mounted() {

  },
  created() {
    this.searchOption()
  },
  data() {
    return {
      toConfigureData: null, // 查看数据
      toConfigure: false, // 查看开关
      addRules: {// 校验
        sqoopMode: [{
          required: true, message: '必填!!'
        }],
        verDt: [{
          required: true, message: '必填!!'
        }]
      },
      inputParamFormVisible: false, // 重新加载
      updateFormData: {},
      addSqoopStatus: true,
      tableToggle: true,
      searchToggle: true,
      pageNo: 1,
      total: null,
      pageSize: null,
      listLoading: false, // 加载圆圈是否显示
      formVisible: false,
      configData: [],
      searchData: { // 查询条件字段
        userName: '',
        noticeType: '',
        noticeMode: ''
      },
      options: {
        'sqoopStatus': {
          '0': '等待导入',
          '1': '准备导入',
          '2': '导入中',
          '3': '导入完成',
          '4': '导入失败'
        },
        'calStatus': {
          '0': '节点错误',
          '1': '未计算',
          '2': '计算中',
          '3': '计算完成',
          '4': '已终止'
        }
      }
    }
  },
  methods: {
    viewChart(row) {
      const params = {
        add_time: row.addTime || '',
        cal_status: row.calStatus || '',
        data_time: row.dataTime || '',
        field_convert_map: {
          cal_status: row.field_convert_map ? row.field_convert_map.calStatus : ''
        },
        flow_desc: row.flowDesc || '',
        flow_name: row.flowName || '',
        task_flow_id: row.taskFlowId
      }
      this.$refs.flowFigure.openDialog(params)
    },
    textJug(text) {
      if (text) {
        if (text.length > 11) {
          return false
        } else {
          return true
        }
      } else {
        return true
      }
    },
    addParam(row) { // 查看
      baseSearch('/basic/sqoops/selectFlow', { sqoopId: row.sqoopId }).then(response => {
        this.toConfigureData = response.data.item
        this.toConfigure = true
      })
    },
    saveOperate() { // 重新导入保存
      this.updateFormData.sqoopStatus = '1'
      saveUpdate('/basic/sqoops/add', this.updateFormData, this.addRules, this.$refs.addForm).then(() => {
        this.searchOption()
        this.$Message.success('操作成功')
        this.inputParamFormVisible = false
      })
    },
    reimport(row) { // 重新导入
      this.inputParamFormVisible = true
      this.updateFormData = {}
      this.updateFormData.sqoopId = row.sqoopId
      this.$nextTick(_ => {
        this.$refs.addForm.clearValidate()
      })
    },
    // importMethod(row) { // 导入点击
    //   baseRequest('/sqoops/add', { sqoopId: row.sqoopId, sqoopStatus: '1' }).then(response => {
    //     if (response.code === 1) {
    //       this.$message({
    //         message: '操作成功',
    //         type: 'success'
    //       })
    //     } else {
    //       this.$Message.error('error')
    //     }
    //     this.searchOption()
    //   })
    // },
    handleDelete(row) { // 删除
      this.$confirm('此操作将永久删除该资源, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        const param = {
          sqoopId: row.sqoopId
        }
        baseRequest('/basic/sqoops/delete', param).then(request => {
          this.searchOption()
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
        })
      })
    },
    openColumn(row) { // 打开修改
      this.addSqoopStatus = true
      this.$nextTick(_ => {
        this.$refs.addSqoop.colmunListVisable = true
        this.$refs.addSqoop.modifiedEcho(row)
      })
    },
    openAddSqoop() { // 打开添加
      this.addSqoopStatus = true
      this.$nextTick(_ => {
        this.$refs.addSqoop.colmunListVisable = true
        this.$refs.addSqoop.clearForm()
      })
    },
    getDefaultHeight() { // 获取表格高度
      return this.$store.state.app.containHeight - 230
    },
    defaultHeight() { // 获取表格高度
      return this.$store.state.app.containHeight - 150
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
      baseSearch('/basic/sqoops/selects', this.searchData).then(response => {
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
    }
  }
}
</script>
<style lang="scss" >
.dialog-reimport {
  height: 40% !important;
  .el-form-item__label {
    width: 120px !important;
  }
  .el-form-item__error {
    margin-left: 20px;
  }
}
</style>