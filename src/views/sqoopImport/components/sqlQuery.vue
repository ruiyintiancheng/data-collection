/*
 * @Author: 1k 
 * @Date: 2019-08-19 09:50:14 
 * @Last Modified by: 1k
 * @Last Modified time: 2019-08-21 17:54:26
 * @Description:  sql语句查询
 */
<template>
  <div>
    <div class="base-row base-row-self">
      <div class="row-botton clearfix">
        <div class="row-title">
          <span>sql输入框</span>
        </div>
        <div class="row-option">
          <!-- <el-button-group>
                       <el-tooltip class="item" effect="dark" content="查询" placement="top">
                          <el-button  icon="el-icon-search" @click="searchOptionSql(demo)"></el-button>
                       </el-tooltip>
                       <el-tooltip class="item" effect="dark" content="格式化" placement="top">
                          <el-button  icon="el-icon-scissors" @click="formatterSql(demo.name)"></el-button>
                       </el-tooltip>
                       <el-tooltip class="item" effect="dark" content="保存" placement="top">
                          <el-button  icon="el-icon-check" @click="saveOptionSql(demo.name)"></el-button>
                       </el-tooltip>
                       <el-tooltip class="item" effect="dark" content="配置" placement="top">
                          <el-button  icon="el-icon-setting" @click="openColumnStep"></el-button>
                       </el-tooltip>
                    </el-button-group> -->
          <el-button icon="el-icon-search"
                     @click="searchOptionSql">查询</el-button>
          <a @click="sqlUp"
             v-if="searchToggle">
            <svg-icon icon-class="up" />&nbsp;收起</a>
          <a @click="sqlDown"
             v-else>
            <svg-icon icon-class="down" />&nbsp;展开</a>
        </div>
      </div>
      <div class="sqlArea"
           v-show="searchToggle">
        <div class="sqlInputDiv"
             style="200px">
          <sql-input :contentHeight="contentHeightDefault"
                     ref="sqlquery"></sql-input>
        </div>
      </div>
    </div>
    <!-- <div class="base-row tableList">
      <div class="row-botton clearfix">
        <div class="row-title">
          <svg-icon icon-class="ul" />
          <span>数据列表</span>
        </div>
        <div class="row-option">
          <a @click="tableUp()"
             v-if="tableToggle">
            <svg-icon icon-class="up" />&nbsp;收起</a>
          <a @click="tableDown()"
             v-else>
            <svg-icon icon-class="down" />&nbsp;展开</a>
        </div>
      </div>
      <div v-show="tableToggle">
        <el-table :data="tableData"
                  class="tb-edit"
                  :border="true"
                  :fit="true"
                  style="width: 100%;">
          <el-table-column v-for="item in tableHead"
                           :label="item.label"
                           :property="item.property"
                           :key="item.aaa"
                           width="180">
          </el-table-column>
        </el-table>
        <el-pagination background
                       @size-change="handleSizeChange($event)"
                       @current-change="handleCurrentChange($event)"
                       :current-page="pageNo"
                       :page-sizes="[10,20,30,50]"
                       :page-size="pageSize"
                       layout="total, sizes, prev, pager, next, jumper"
                       :total="total">
        </el-pagination>
      </div>
    </div> -->
  </div>
</template>
<script>
import sqlInput from './sqlInput'
import { baseRequest } from '@/api/base'

export default {
  components: {
    sqlInput
  },
  data() {
    return {
      smartIndent: true, // 默认缩进
      styleActiveLine: true, // 当前行高亮
      lineNumbers: true, // 显示行数
      lineWrapping: true, // 自动换行
      matchBrackets: true, // 括号匹配
      autofocus: true, // 自动获取焦点
      extraKeys: { 'Ctrl-Space': 'autocomplete' },
      // theme: 'blackboard', // 编辑框的主题
      indentUnit: 2, // 缩进,默认为2
      abSize: 4, // tab字符的宽度，默认为4
      contentHeightDefault: '150px', // sql输入框高度
      searchToggle: true,
      tableToggle: false,
      pageNo: 1,
      pageSize: null,
      total: 350,
      tableHead: [
        { property: 'DICT_ID', label: 'DICT_ID' },
        { property: 'PARENT_DICT_ID', label: 'PARENT_DICT_ID' },
        { property: 'DICT_TYPE', label: 'DICT_TYPE' },
        { property: 'DICT_CODE', label: 'DICT_CODE' },
        { property: 'MSG_ID', label: 'MSG_ID' },
        { property: 'DISP_ORDER', label: 'DISP_ORDER' },
        { property: 'ADD_TIME', label: 'ADD_TIME' }
      ],
      tableData: [
        { ADD_TIME: null, DICT_CODE: 'basicDictType', DICT_ID: 1, DICT_TYPE: 'basicDictType', DISP_ORDER: 0, MSG_ID: 33, PARENT_DICT_ID: 0 }
      ]
    }
  },
  created() {
    this.containHeight = this.$store.state.app.containHeight - 16
    this.contentHeightDefault = (this.$store.state.app.containHeight - 260) + 'px'
    this.getTrees()
    var self = this
    document.querySelector('body').onclick = function() {
      self.visible = false
    }
  },
  methods: {
    searchOptionSql() { // sql编辑框sql的执行
      var text = this.$refs.sqlquery.getSql()
      var param = {}
      param.sqlContent = text
      param.pageNo = this.pageNo || 1
      param.pageSize = this.pageSize || null
      baseRequest('/basic/sqlQuerys/selectBySql', param).then(response => {
        // const result = response.data.item[0]
        // ref.sqlMessage = result.resultMessage
        // // 查询结果存储到查询的历史数据的对象中 resultMessage
        // ref.tableHead = result.tableHead
        // ref.tableData = result.tableData
        // ref.total = response.data.total
        // ref.pageSize = response.data.pageSize
        // if (ref.searchToggle) {
        //   ref.tableDataH = this.$store.state.app.containHeight - 380
        // } else {
        //   ref.tableDataH = this.$store.state.app.containHeight - 229
        // }
        this.tableDown()
      })
    },
    sqlUp() {
      this.searchToggle = false
      // ref.tableDataH = this.$store.state.app.containHeight - 229
    },
    sqlDown() {
      this.searchToggle = true
      // ref.tableDataH = this.$store.state.app.containHeight - 380
    },
    tableUp() { // 数据列表收起
      this.tableToggle = false
      this.$refs.sqlquery.setSqlnputCss('', this.contentHeightDefault)
    },
    tableDown(ref) { // 数据列表展开
      this.tableToggle = true
      //  this.contentHeight = '150px'
      this.$refs.sqlquery.setSqlnputCss('', '150px')
    },
    handleSizeChange(val) { // 分页
      this.pageSize = val
      this.searchOptionSql()
    },
    handleCurrentChange(val) { // 分页
      this.pageNo = val
      this.searchOptionSql()
    }
  }
}
</script>