/*
 * @Author: 1k 
 * @Date: 2019-08-09 16:31:07 
 * @Last Modified by: wk
 * @Last Modified time: 2020-05-21 10:18:18
 * @Description:  模板文件记录列表
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

          <el-form-item style="margin-left:38px">
            <span class="input-label">模板名称</span>
            <el-input size="medium "
                      class="form-input"
                      v-model="searchData.collectTemplateName"
                      clearable
                      style="width:255px"></el-input>
          </el-form-item>

          <el-form-item style="margin-left:38px">
            <span class="input-label">文件名称</span>
            <el-input size="medium "
                      class="form-input"
                      v-model="searchData.fileName"
                      clearable
                      style="width:255px"></el-input>
          </el-form-item>

          <el-form-item style="margin-left:38px">
            <span class="input-label">模板类型</span>
            <el-select class="form-input"
                       v-model="searchData.collectTemplateType"
                       size="medium "
                       clearable
                       style="width:255px"
                       placeholder="">
              <el-option label="csv导入"
                         value="1"></el-option>
              <el-option label="excel导入"
                         value="2"></el-option>

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
          <el-button icon="el-icon-upload2"
                     @click="openUpdateFile">导入数据</el-button>
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
          <el-table-column :min-width="120"
                           align='center'
                           label="文件名称">
            <template slot-scope="scope">
              {{scope.row.filename}}
            </template>
          </el-table-column>
          <el-table-column :min-width="100"
                           align='center'
                           label="文件类型">
            <template slot-scope="scope">
              {{scope.row.filetype}}
            </template>
          </el-table-column>
          <el-table-column :min-width="120"
                           align='center'
                           label="模板名称">
            <template slot-scope="scope">
              {{scope.row.collecttemplatename}}
            </template>
          </el-table-column>
          <el-table-column :min-width="100"
                           align='center'
                           label="模板类型">
            <template slot-scope="scope">
              {{options.collectTemplateType[scope.row.collecttemplatetype]}}
            </template>
          </el-table-column>
          <!-- <el-table-column :min-width="150"
                           align='center'
                           label="采集模板类型">
            <template slot-scope="scope">
              {{scope.row.collecttemplatetype}}
            </template>
          </el-table-column> -->
          <el-table-column :min-width="150"
                           align='center'
                           label="导入表名">
            <template slot-scope="scope">
              {{scope.row.tablename}}
            </template>
          </el-table-column>
          <el-table-column :min-width="90"
                           align='center'
                           label="导入记录数">
            <template slot-scope="scope">
              {{scope.row.datacount}}
            </template>
          </el-table-column>
          <el-table-column :min-width="80"
                           align='center'
                           label="导入人姓名">
            <template slot-scope="scope">
              {{scope.row.username}}
            </template>
          </el-table-column>
          <el-table-column :min-width="150"
                           align='center'
                           label="导入时间">
            <template slot-scope="scope">
              {{scope.row.addtime}}
            </template>
          </el-table-column>

          <!-- <el-table-column :min-width="150"
                           align='center'
                           label="数据采集模板类型">
            <template slot-scope="scope">
              {{options.collectTemplateType[scope.row.collectTemplateType]}}
            </template>
          </el-table-column> -->
          <!-- <el-table-column :min-width="120"
                           label="操作"
                           align="center">
            <template slot-scope="scope">
              <el-button type="primary"
                         plain
                         size="mini"
                         @click="addParam(scope.row)">查看</el-button>
              <el-button plain
                         size="mini"
                         @click="openColumn(scope.row)">修改</el-button>
              <el-button type="danger"
                         plain
                         size="mini"
                         @click="handleDelete(scope.row)">删除</el-button>
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
    <excel-file ref="excelFile"></excel-file>

  </div>
</template>
<script>
import excelFile from './components/excelFile'

import { baseSearch } from '@/api/base'

export default {
  directives: {},
  name: 'templateFileRecordingIndex',
  components: { excelFile },
  mounted() {

  },
  created() {
    this.searchOption()
  },
  data() {
    return {

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
        'collectTemplateType': {
          '1': 'csv导入',
          '2': 'excel导入'
        }
      }
    }
  },
  methods: {
    openUpdateFile() { // 打开模板
      this.$refs.excelFile.inputParamFormVisible = true
      this.$refs.excelFile.clearForm()
    },
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
      baseSearch('/basic/t04CollectFile/selectCollectFile', this.searchData).then(response => {
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
