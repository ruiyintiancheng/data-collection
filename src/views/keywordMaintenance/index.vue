/*
 * @Author: 1k 
 * @Date: 2019-09-10 14:49:14 
 * @Last Modified by: 1k
 * @Last Modified time: 2019-09-19 13:38:42
 * @Description:  关键字维护
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

          <el-form-item style="margin-left:60px">
            <span class="input-label">关键字名称</span>
            <el-input size="medium "
                      class="form-input"
                      v-model="searchData.keyName"
                      clearable
                      style="width:255px"></el-input>
          </el-form-item>
          <el-form-item style="margin-left:60px">
            <span class="input-label">数据源类型</span>
            <el-select class="form-input"
                       v-model="searchData.dsType"
                       size="medium "
                       clearable
                       style="width:255px"
                       placeholder="">
              <el-option label="Oracle"
                         value="1"></el-option>
              <el-option label="DB2"
                         value="2"></el-option>
              <el-option label="MySql"
                         value="3"></el-option>
            </el-select>
          </el-form-item>

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
                  fit
                  header-cell-class-name="ai-el-table-header">
          <el-table-column width=50
                           type="index"
                           align="center"
                           fixed="left"
                           label="序号"></el-table-column>
          <el-table-column :min-width="150"
                           align='center'
                           label="关键字名称">
            <template slot-scope="scope">
              {{scope.row.keyName}}
            </template>
          </el-table-column>
          <el-table-column :min-width="200"
                           align='center'
                           label="关键字描述">
            <template slot-scope="scope">
              {{scope.row.keyDesc}}
            </template>
          </el-table-column>
          <el-table-column :min-width="150"
                           align='center'
                           label="数据源类型">
            <template slot-scope="scope">
              {{options.dsType[scope.row.dsType] }}
            </template>
          </el-table-column>
          <el-table-column :width="150"
                           label="操作"
                           fixed="right"
                           align="center">
            <template slot-scope="scope">
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
    <!-- 添加  修改 -->
    <el-dialog :title="dialogTitle[operateStatus]"
               :visible.sync="formVisible"
               width="40%"
               custom-class="dialog-default">
      <div class="dialog-contant-default-addOrUpdate">
        <el-form class="baseUpdate-form"
                 ref="form"
                 :rules="addRules"
                 :model="updateFormData"
                 label-width="100px">
          <el-form-item label="关键字名称"
                        prop="keyName">
            <el-input class="form-input"
                      v-model="updateFormData.keyName"
                      clearable></el-input>
          </el-form-item>
          <el-form-item label="关键字描述">
            <el-input class="form-input"
                      v-model="updateFormData.keyDesc"
                      clearable></el-input>
          </el-form-item>
          <el-form-item label="数据源类型"
                        prop="dsType">
            <el-select v-model="updateFormData.dsType"
                       size="small"
                       clearable
                       placeholder="">
              <el-option label="Oracle"
                         value="1"></el-option>
              <el-option label="DB2"
                         value="2"></el-option>
              <el-option label="MySql"
                         value="3"></el-option>
            </el-select>
          </el-form-item>
        </el-form>
      </div>
      <div slot="footer"
           class="dialog-footer-addOrUpdate">
        <el-button @click="formVisible = false">取消</el-button>
        <el-button type="primary"
                   @click="saveOperate()">保存</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
// import excelFile from './components/excelFile'

import { baseSearch } from '@/api/base'
import { baseRequest } from '@/api/base'
import { saveUpdate } from '@/utils/validate'
export default {
  directives: {},
  name: 'templateFileRecordingIndex',
  components: {
  },
  mounted() {
    this.searchOption()
  },
  created() {

  },
  data() {
    return {
      updateFormData: {
        keyName: null,
        keyDesc: null,
        dsType: null
      },
      addRules: {// 校验
        dsType: [{
          required: true, message: '请选择数据源类别'
        }],
        keyName: [{
          required: true, message: '请填写关键字'
        }]
      },
      formVisible: false, // 弹框开关
      operateStatus: null, // 操作选项
      dialogTitle: { // 弹框标题
        1: '添加',
        2: '修改'
      },
      tableToggle: true,
      searchToggle: true,
      pageNo: 1,
      total: null,
      pageSize: null,
      listLoading: false, // 加载圆圈是否显示
      configData: [],
      searchData: { // 查询条件字段
        keyName: null,
        dsType: null
      },
      options: {
        'dsType': {
          '1': 'Oracle',
          '2': 'DB2',
          '3': 'MySql'
        }
      }
    }
  },
  methods: {
    // 保存操作
    saveOperate() {
      if (this.operateStatus === 1) {
        saveUpdate('/basickeywordInfos/add', this.updateFormData, this.addRules, this.$refs.form).then(() => {
          this.searchOption()
          this.$Message.success('操作成功')
          this.formVisible = false
        })
      } else {
        saveUpdate('/basickeywordInfos/update', this.updateFormData, this.addRules, this.$refs.form).then(() => {
          this.searchOption()
          this.$Message.success('操作成功')
          this.formVisible = false
        })
      }
    },
    handleDelete(row) { // 删除
      this.$confirm('此操作将永久删除该资源, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        const param = {
          keyId: row.keyId
        }
        baseRequest('/basic/keywordInfos/delete', param).then(request => {
          this.searchOption()
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
        })
      })
    },
    openColumn(row) { // 打开修改
      this.operateStatus = 2
      this.getUpdateForm(row)
      this.updateFormData.keyId = row.keyId
    },
    // 获取修改表单
    getUpdateForm(row) {
      const param = {
        keyId: row.keyId
      }
      baseRequest('/basic/keywordInfos/select', param).then(response => {
        this.updateFormData = response.data.item
        this.formVisible = true
      })
    },
    openAddSqoop() { // 打开添加
      this.updateFormData.dsType = ''
      this.updateFormData.keyDesc = ''
      this.updateFormData.keyName = ''
      this.updateFormData.keyId = ''
      this.$nextTick(_ => {
        this.$refs.form.clearValidate()
      })
      this.operateStatus = 1
      this.formVisible = true
    },
    getDefaultHeight() { // 获取表格高度
      return this.$store.state.app.containHeight - 230
    },
    searchOption(page) {
      this.listLoading = true
      if (!page) {
        this.pageNo = 1
      }
      this.searchData.pageNo = this.pageNo
      this.searchData.pageSize = this.pageSize
      baseSearch('/basic/keywordInfos/selects', this.searchData).then(response => {
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
<style lang="scss" scoped>
.dialog-contant-default-addOrUpdate {
  margin-top: 10px;
  padding: 0px;
  border: 0px;
}
.dialog-contant-default-addOrUpdate {
  .baseUpdate-form {
    margin-left: calc(50% - 150px);
    .form-input {
      width: 200px;
    }
  }
}
</style>