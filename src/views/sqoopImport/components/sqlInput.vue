<template>
  <div class="sql-input-fa clearfix">
    <div class="sql-input-le">
      <textarea id="sqoopImport"
                name="code"
                style="width:100%;height:100%;display:none"></textarea>
    </div>
    <div class="botton-body">
      <el-button @click="getSql"
                 type="text"
                 icon="el-icon-d-arrow-right"
                 style="margin-right: 0;"></el-button>
    </div>
    <el-table ref="multipleTable"
              :data="tableData"
              style="width:47%;border-left:1px solid #ccc"
              height='250'>
      <el-table-column type="index"
                       width="55"
                       label="序号">
      </el-table-column>
      <el-table-column label="字段列表">
        <template slot-scope="scope">{{ scope.row.column }}</template>
      </el-table-column>
      <!-- <el-table-column prop="name"
                       label="姓名"
                       width="120">
      </el-table-column>
      <el-table-column prop="address"
                       label="地址"
                       show-overflow-tooltip>
      </el-table-column> -->
    </el-table>

  </div>
</template>
<script>
import { baseRequest } from '@/api/base'
import CodeMirror from 'codemirror'
import 'codemirror/addon/lint/lint.css'
import 'codemirror/lib/codemirror.css'
import 'codemirror/theme/rubyblue.css'
require('script-loader!jsonlint')
import 'codemirror/mode/javascript/javascript'
import 'codemirror/addon/lint/lint'
import 'codemirror/addon/lint/json-lint'

import 'codemirror/addon/edit/matchbrackets'
import 'codemirror/mode/sql/sql'
import 'codemirror/addon/hint/show-hint.css'
import 'codemirror/addon/hint/show-hint'
import 'codemirror/addon/hint/sql-hint'
import 'codemirror/addon/selection/active-line'
import 'codemirror/theme/blackboard.css'
import sqlFormatter from 'sql-formatter'
export default {
  components: {
    baseRequest,
    CodeMirror,
    sqlFormatter
  },
  props: {
    tableData: Array,
    contentHeight: String,
    id: String,
    code: String,
    dataSourceCondition: Object,
    parameters: Object
  },
  data() {
    return {
      monitor: null, // 点击查询保存的值
      selText: '',
      codeWidth: null, // sql编辑区域的宽
      codeHeigth: null, // sql编辑区域的高
      smartIndent: true,
      styleActiveLine: true,
      lineNumbers: true, // 显示行号
      lineWrapping: true, // 自动换行
      matchBrackets: true, // 括号匹配
      autofocus: true, // 自动获取焦点
      indentUnit: 2, // 缩进,默认为2
      abSize: 4, // tab字符的宽度，默认为4
      returnData: null, // 执行sql返回的数据
      editor: null,
      tables: null, // 数据源表字段信息
      columns: null,
      columnInfo: true,
      filterText: '',
      treeData: [],
      multipleSelection: []
    }
  },
  methods: {
    toggleSelection(rows) {
      if (rows) {
        rows.forEach(row => {
          this.$refs.multipleTable.toggleRowSelection(row)
        })
      } else {
        this.$refs.multipleTable.clearSelection()
      }
    },
    handleSelectionChange(val) {
      this.multipleSelection = val
    },
    getColumns() { // 获取列名
      // const tableColumhns = []
      // for (const item of this.multipleSelection) {
      //   tableColumhns.push(item.name)
      // }
      // return tableColumhns
      return this.multipleSelection
    },
    // 查询生成列
    searchTables() {
      const filterTextUp = this.filterText.toUpperCase()
      const filterTextLo = this.filterText.toLowerCase()
      if (this.tables.hasOwnProperty(filterTextUp)) {
        this.treeData = this.tables[filterTextUp]
      } else if (this.tables.hasOwnProperty(filterTextLo)) {
        this.treeData = this.tables[filterTextLo]
      } else {
        this.treeData = []
        this.$message.warning('没有查到该信息')
      }
    },
    getData() {
      // 出书画编辑区域
      const param = { dsCode: this.parameters.Identification }
      baseRequest('/basic/sqlExecute/getTablesAndColumns', param).then(response => {
        // this.tables = response.data.item
        const data = response.data.item
        this.tables = data
        // })
        // const that = this
        var mime = 'text/x-sql'
        this.editor = CodeMirror.fromTextArea(
          document.getElementById('sqoopImport'),
          {
            mode: mime, // 模型类型
            indentWithTabs: true,
            smartIndent: true, // 默认缩进
            styleActiveLine: true, // 当前行高亮
            lineNumbers: true, // 显示行号
            lineWrapping: true, // 自动换行
            matchBrackets: true, // 括号匹配
            autofocus: true, // 自动获取焦点
            extraKeys: { 'Ctrl-Space': 'autocomplete' },
            // theme: 'blackboard', // 编辑框的主题
            indentUnit: this.indentUnit, // 缩进,默认为2
            abSize: this.abSize, // tab字符的宽度，默认为4
            gutters: ['CodeMirror-lint-markers'],
            hintOptions: {
              tables: data
              // tables: []
            }
          }
        )
        this.editor.on('keyup', function(cm, event) {
          // 所有的字母和'$','{','.'在键按下之后都将触发自动完成
          if (
            !cm.state.completionActive &&
            ((event.keyCode >= 65 && event.keyCode <= 90) ||
              event.keyCode === 52 ||
              event.keyCode === 219 ||
              event.keyCode === 190)
          ) {
            CodeMirror.commands.autocomplete(cm, null, {
              completeSingle: false
            })
          }
        })
        // this.editor.on('cursorActivity', this.hiddenColumn)
        // this.editor.on('dblclick', this.showColumn) // 双击事件
        this.setSql()
        this.editor.setSize(this.codeWidth, this.codeHeigth)
        // if (this.dataSourceCondition.sqlId || this.dataSourceCondition.sqlId === 0) {
        //   baseRequest('/sqlConfig/select', { sqlId: this.dataSourceCondition.sqlId }).then(response => {
        //     this.setSql(response.data.item.sqlContent)
        //   })
        // }
      })
    },
    setSqlnputCss(width, height) {
      // 设置输入sql区域的大小
      // this.codeWidth = width
      this.codeHeigth = height
      this.editor.setSize(this.codeWidth, this.codeHeigth)
    },
    getConfig(faConfig, defaultConfig) {
      // 将子组件和父组件的配置进行整合
      if (typeof faConfig === 'undefined') {
        return defaultConfig
      } else {
        return faConfig
      }
    },
    formatter() {
      // 输入sql进行格式化
      this.editor.setValue(sqlFormatter.format(this.editor.getValue()))
    },
    getSqlValue() {
      // 获取sql语句
      return this.editor.getValue()
    },
    getSql() {
      // 获取sql语句
      // return this.editor.getValue()
      this.monitor = this.editor.getValue()
      // this.gouruyi = this.editor.getValue()
      if (!this.monitor) {
        this.$message.error('sql不能为空')
        return
      }
      baseRequest('/basic/sqoops/selectBySql', { dataSourceCode: this.parameters.Identification, sql: this.monitor }).then(response => {
        // this.tableData = response.data.item.tableHead
        this.$emit('changeTableData', response.data.item.tableHead)
        if (response.data.item.resultMessage) {
          this.$emit('changeTableData', [])
          // this.tableData = null
          this.$message.error(response.data.item.resultMessage)
          return false
        }
      })

      // this.$refs.multipleTable.toggleAllSelection()
    },
    setSql() {
      // 设置sql语句
      if (this.parameters.sql !== null) {
        this.editor.setValue(this.parameters.sql)
      } else {
        this.editor.setValue('')
      }
    },
    // reset(sqlValue) {
    //   // 设置sql编辑区域的值
    //   this.editor.setValue('')
    // },
    searchOptionSql() {
      this.selText = this.editor.getSelection()
    }
  },

  mounted() {
    // 初始化编辑框的属性,调用的组件未传值的使用默认值
    // this.smartIndent = this.$options.methods.getConfig(
    //   this.$parent.smartIndent,
    //   this.smartIndent
    // )
    // this.styleActiveLine = this.$options.methods.getConfig(
    //   this.$parent.styleActiveLine,
    //   this.styleActiveLine
    // )
    // this.lineNumbers = this.$options.methods.getConfig(
    //   this.$parent.lineNumbers,
    //   this.lineNumbers
    // )
    // this.lineWrapping = this.$options.methods.getConfig(
    //   this.$parent.lineWrapping,
    //   this.lineWrapping
    // )
    // this.matchBrackets = this.$options.methods.getConfig(
    //   this.$parent.matchBrackets,
    //   this.matchBrackets
    // )
    // this.autofocus = this.$options.methods.getConfig(
    //   this.$parent.autofocus,
    //   this.autofocus
    // )
    // this.indentUnit = this.$options.methods.getConfig(
    //   this.$parent.indentUnit,
    //   this.indentUnit
    // )
    // this.abSize = this.$options.methods.getConfig(
    //   this.$parent.abSize,
    //   this.abSize
    // )
    this.codeWidth = this.$parent.codeWidth || '100%'
    this.codeHeigth = this.$parent.codeHeigth || '150px'
    // this.codeHeigth = this.$parent.codeHeigth || this.contentHeight
    // 创建编辑框
    // const param = { dataSourceCode: this.parameters.Identification, dataSourceName: this.parameters.dataSourceName }
    // baseRequest('/sqoops/getTablesAndColumns', param).then(response => {
    //   // this.tables = response.data.item
    //   const data = response.data.item
    //   this.tables = data
    this.getData()
    // })
    // this.$options.methods.formatter()
  }
}
</script>

<style lang="scss" scoped>
.sql-input-fa {
  width: calc(100%);
  .CodeMirror {
    height: 250px !important;
  }
  .botton-body {
    width: 3%;
    border: 1px solid #ccc;
    border-radius: 5px;
    float: left;
    margin-top: 15%;
  }
  .sql-input-le {
    line-height: 20px;
    font-size: 12px;
    float: left;
    width: calc(50%) !important;
  }
  .sql-input-ri {
    float: left;
    width: 300px;
    border-left: 1px solid #ccc;
    .table-currnet-title {
      font-size: 14px;
      height: 30px;
      line-height: 30px;
      padding-left: 10px;
      background: #f3f3f3;
      color: #9d9399;
    }
    .el-tree-node__content {
      font-size: 12px;
    }
  }
  .sqlInputDiv .CodeMirror {
    padding: 0;
    min-height: 150px;
  }
  .sqlInputDiv .CodeMirror .CodeMirror-scroll {
    min-height: 150px;
  }
  .columnInfo {
    background-color: white;
    border: 1px solid gray;
    font-size: 13px;
    padding: 5px;
    position: absolute;
    right: 0;
    z-index: 10;
    max-height: 150px;
    overflow-y: auto;
  }
  .columnInfo > ul {
    list-style-type: none;
    padding: 0;
    margin: 0;
  }
}
</style>
<style lang="scss">
.sql-input-le {
  .CodeMirror {
    height: 250px !important;
  }
}
.CodeMirror-hints {
  z-index: 9999;
}
</style>

