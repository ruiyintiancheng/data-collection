/*
 * @Author: 1k 
 * @Date: 2019-08-16 11:29:27 
 * @Last Modified by: 1k
 * @Last Modified time: 2019-09-24 17:49:44
 * @Description:  Sqoop导入配置弹框
 */
<template>
  <el-dialog title="配置"
             v-el-drag-dialog
             :close-on-click-modal="false"
             :visible.sync="colmunListVisable"
             width="75%"
             top="5vh"
             custom-class="dialog-default dialog-default-more">
    <div class="dialog-contant-default sqoop-input"
         v-if="colmunListVisable">

      <el-steps :active="active"
                finish-status="success"
                simple>
        <el-step title="条件筛选"></el-step>
        <el-step title="源数据选择"></el-step>
        <el-step title="目标数据选择"></el-step>
      </el-steps>

      <template v-if="active === 0">
        <el-form class="baseUpdate-form"
                 ref="fistForm"
                 :rules="addRules"
                 :model="updateFormData"
                 label-width="100px"
                 label-position="right"
                 style="margin-top:50px;margin-left:320px">
          <el-form-item label="配置名称"
                        prop='sqoopName'>
            <el-input class="form-input"
                      style="width:150px;"
                      v-model.trim="updateFormData.sqoopName"
                      clearable></el-input>
          </el-form-item>
          <el-form-item label="导入/导出">
            <el-radio-group v-model="updateFormData.sqoopType"
                            @change="importAndExport">
              <el-radio label="import">sqoop导入</el-radio>
              <el-radio label="export">sqoop导出</el-radio>
            </el-radio-group>
          </el-form-item>
          <el-form-item label="是否加入队列"
                        prop="">
            <el-checkbox-group @change="typeChange"
                               v-model="updateFormData.type">
              <el-checkbox label=" "
                           name="type">
                <el-form-item label=""
                              v-if="updateFormData.type[0]"
                              prop="sqoopQueue">
                  <el-input size="small"
                            width='150'
                            class="form-input input-icon"
                            v-model.trim="updateFormData.sqoopQueue"
                            clearable></el-input>
                </el-form-item>
              </el-checkbox>
            </el-checkbox-group>
          </el-form-item>
          <el-form-item label="任务数"
                        prop="">
            <el-input-number v-model="updateFormData.taskNum"
                             controls-position="right"
                             :min="1"
                             :max="5"
                             size="small"></el-input-number>
          </el-form-item>
        </el-form>
      </template>
      <template v-if="updateFormData.sqoopType==='export'?active===2:active===1">
        <el-form class="baseUpdate-form"
                 ref="secondForm"
                 :rules="addRules"
                 :model="secondFormData"
                 label-width="100px"
                 label-position="right"
                 style="margin-top:10px">

          <el-form-item label="数据源选择"
                        prop="optionsValue">
            <el-cascader :options="options"
                         clearable
                         @change="clearTable"
                         v-model="secondFormData.optionsValue"
                         :props="SetKesDept"
                         placeholder=""
                         style="width:155px">
              <template slot-scope="{ node, data }">
                <span>{{data.name}}</span>
              </template>
            </el-cascader>
          </el-form-item>
          <el-form-item label="查询条件"
                        v-if="updateFormData.sqoopType==='export'?false:updateFormData.optionsVal">
            <el-radio-group v-model="secondFormData.criteria"
                            @change="radioChange">
              <el-radio label="指定表"></el-radio>
              <el-radio label="SQL查询语句"></el-radio>
            </el-radio-group>
          </el-form-item>
          <el-form-item label="查询语句"
                        v-if="updateFormData.sqoopType==='export'?false:(updateFormData.optionsVal?secondFormData.criteria==='SQL查询语句':false)">
            <sql-input ref="sqlInput"
                       style="border:1px solid #ccc"
                       :parameters='parameters'
                       :tableData='tableDataInput'
                       @changeTableData="changeTableData"
                       v-if="secondFormData.criteria==='SQL查询语句'"></sql-input>
          </el-form-item>
          <el-form-item label="选择表"
                        v-if="updateFormData.sqoopType==='export'?(updateFormData.optionsVal?true:false):(updateFormData.optionsVal?secondFormData.criteria==='指定表':false)">
            <table-query ref="tableQuery"
                         @changeCurrentTable="changeCurrentTable"
                         :parameters='parameters'
                         :tableData='tableDataQuery'
                         v-show="updateFormData.sqoopType==='export'?true:secondFormData.criteria==='指定表'"></table-query>

          </el-form-item>
          <el-form-item label="where条件"
                        prop=""
                        v-if="updateFormData.sqoopType==='export'?false:(updateFormData.optionsVal?secondFormData.criteria==='指定表':false)">
            <el-input type="textarea"
                      placeholder="示例:DICT_TYPE='1' AND DICT_VALUE=1"
                      v-model="secondFormData.condition"></el-input>
          </el-form-item>

        </el-form>
      </template>
      <template v-if="updateFormData.sqoopType==='export'?active===1:active===2">
        <el-form class="baseUpdate-form"
                 ref="thirdForm"
                 :rules="addRules"
                 :model="thirdFormData"
                 label-width="100px"
                 label-position="right"
                 style="margin-top:10px;margin-left:200px">

          <el-form-item label="数据源选择"
                        prop="targetOptionsValue">
            <el-cascader :options="targetOptions"
                         clearable
                         @change="targetClearTable"
                         v-model="thirdFormData.targetOptionsValue"
                         :props="targetSetKesDept"
                         placeholder=""
                         style="width:155px">
              <template slot-scope="{ node, data }">
                <span>{{data.name}}</span>
              </template>
            </el-cascader>
          </el-form-item>
          <el-form-item label="选择表">
            <el-table ref="targeterTable"
                      :data="tableDat.filter(data => !search || data.tablename.toLowerCase().includes(search.toLowerCase()))"
                      height="300"
                      highlight-current-row
                      @current-change="handleCurrentChange"
                      style="width: 350px;border:1px solid #ccc">
              <el-table-column label="表名">
                <template slot="header"
                          slot-scope="scope">
                  <el-input v-model="search"
                            size="mini"
                            :clearable="!scope"
                            placeholder="输入关键字搜索" />
                </template>
                <template slot-scope="scope">
                  {{scope.row.tablename}}
                </template>
              </el-table-column>
            </el-table>
          </el-form-item>
          <el-form-item label="表分区"
                        v-if="zoningSwitch"
                        prop="tablePartition">
            <el-select v-model="thirdFormData.tablePartition"
                       style="width:155px"
                       placeholder="">
              <el-option v-for="item in targetPartition"
                         :key="item.partition"
                         :label="item.partition"
                         :value="item.partition"></el-option>
            </el-select>
          </el-form-item>
        </el-form>
      </template>
    </div>
    <div slot="footer"
         class="dialog-footer">
      <el-button @click="colmunListVisable = false">取消</el-button>
      <el-button v-if="active !== 0"
                 @click="compeleteReturn">上一步</el-button>
      <el-button @click="compeleteCurrent"
                 type="primary">{{active===2?'完成':'下一步'}}</el-button>
      <!-- type="primary">保存</el-button> -->
    </div>
  </el-dialog>
</template>
<script>
// import columnList from './columnList'
// import columnOrder from './columnOrder'
import sqlQuery from './sqlQuery'
import tableQuery from './tableQuery'
import sqlInput from './sqlInput'
import { baseRequest } from '@/api/base'
// import { saveUpdate } from '@/utils/validate'
export default {
  name: 'templateConfigurationComponentsModification',
  components: {
    // columnList,
    // columnOrder
    sqlQuery,
    tableQuery,
    sqlInput
  },
  props: {
    collectTemplateId: Number,
    tableId: Number
  },
  data() {
    return {
      record: null,
      zoningSwitch: false, // 分区开关
      targetPartition: null, // 分区的数据
      targeterSchema: null,
      tab: {},
      tableSelection: null, // 目标表选中值
      sqoopId: null, // 行的id
      tableDataInput: null,
      sqlInputValue: null,
      currentTable: null,
      tableDataQuery: [],
      param: {},
      currentRow: null, // 选中目标表
      search: null,
      tableDat: [], // 目标表
      parameters: {
        Identification: null, // 传的数据源code
        dataSourceName: null, // 传的数据源用户
        useId: null,
        tableQueryColnmus: [],
        tableQuerySelection: {},
        sql: null,
        tableExport: null
      },
      option: {
        resource: {
          1: 'sqoop导入',
          2: 'sqoop导出'
        }
      },
      loading: false,
      selectOption: [],
      SetKesDept: {
        value: 'name',
        label: 'name',
        code: 'code',
        children: 'children'
      }, // 自定义  级联选择器value label
      targetSetKesDept: {
        value: 'name',
        label: 'name',
        code: 'code',
        children: 'children'
      },
      options: [],
      targetOptions: [],
      tableToggle: true,
      searchToggle: true,
      data: [],
      value: [],
      addRules: {// 校验
        sqoopName: [{
          required: true, message: '必填!!'
        }],
        tableName: [{
          required: true, message: '必填!!'
        }],
        tablePartition: [{
          required: true, message: '必填!!'
        }],
        targetOptionsValue: [{
          required: true, message: '必填!!'
        }],
        sqoopQueue: [{
          required: true, message: '必填!!'
        }],
        optionsValue: [{
          required: true, message: '必填!!'
        }]
      },
      // 当前步骤
      active: 0,
      // 弹窗开关
      colmunListVisable: false,
      // 解析列
      tableData: [],
      // 排序列
      orderData: [],
      updateFormData: {
        sqoopName: null,
        sqoopType: 'import',
        type: [],
        taskNum: 1,
        tableName: '',
        optionsVal: false,
        sqoopQueue: null
      },
      secondFormData: {
        optionsValue: [],
        criteria: '指定表',
        condition: null
      },
      thirdFormData: {
        targetOptionsValue: [],
        tablePartition: null
      },
      activeCriteria: null, // 记录查询条件
      user: []
    }
  },
  created() {

  },
  watch: {
    colmunListVisable(val) {
      if (val === false) {
        this.$parent.addSqoopStatus = false
        console.log(this.$parent.addSqoopStatus)
      }
    },
    active(value) {
      if (value === 0) {
        this.$nextTick(_ => {
          this.$refs.fistForm.clearValidate()
        })
      }
      if (value === 1) {
        if (this.updateFormData.sqoopType === 'import') {
          this.$nextTick(_ => {
            this.$refs.secondForm.clearValidate()
            if (this.currentTable) {
              this.$refs.tableQuery.selectedStatusById(this.currentTable.tableid)
            }
            if (this.tab.query !== '' && !this.tab.table) {
              this.secondFormData.criteria = 'SQL查询语句'
            } else {
              this.secondFormData.criteria = '指定表'
              if (this.secondFormData.optionsValue.length === 0) {
                this.updateFormData.optionsVal = false
              } else {
                for (const dataSource of this.options) {
                  if (dataSource.name === this.secondFormData.optionsValue[0]) {
                    this.parameters.Identification = dataSource.code
                    this.parameters.dataSourceName = this.secondFormData.optionsValue[1]
                    for (const dataChild of dataSource.children) {
                      if (dataChild.name === this.secondFormData.optionsValue[1]) {
                        // console.log(dataChild.name)
                        // console.log(val[1])
                        this.parameters.useId = dataChild.schemaId
                      }
                    }
                  }
                }
                if (this.tab.table !== '') {
                  baseRequest('/basic/tables/tableOptions', { schemaId: this.parameters.useId }).then(response => {
                    this.tableDataQuery = response.data.item
                    if (this.tableDataQuery !== []) {
                      const laborious = this.tableDataQuery
                      if (this.currentTable) {
                        this.$refs.tableQuery.selectedStatusById(this.currentTable.tableid)
                      } else {
                        for (const labor of laborious) {
                          if (labor.tablename === this.tab.table) {
                            this.parameters.tableQuerySelection = labor
                            this.$refs.tableQuery.selectedStatus(labor)
                          }
                        }
                      }
                    }
                  })
                }
                this.updateFormData.optionsVal = true
              }
            }
          })
        } else if (this.updateFormData.sqoopType === 'export') {
          this.$nextTick(_ => {
            if (this.tableSelection === null) {
              this.$refs.targeterTable.setCurrentRow(this.currentRow)
            } else {
              this.$refs.targeterTable.setCurrentRow(this.tableSelection)
            }
            this.thirdFormData.tablePartition = this.record
          })
        }
      }
      if (this.active === 2) {
        if (this.updateFormData.sqoopType === 'import') {
          this.$nextTick(_ => {
            this.$refs.targeterTable.setCurrentRow(this.tableSelection)
            this.thirdFormData.tablePartition = this.record
          })
        } else if (this.updateFormData.sqoopType === 'export') {
          this.$nextTick(_ => {
            if (this.currentTable) {
              this.$refs.tableQuery.selectedStatusById(this.currentTable.tableid)
            }
            if (this.secondFormData.optionsValue.length === 0) {
              this.updateFormData.optionsVal = false
            } else {
              for (const dataSource of this.options) {
                if (dataSource.name === this.secondFormData.optionsValue[0]) {
                  this.parameters.Identification = dataSource.code
                  this.parameters.dataSourceName = this.secondFormData.optionsValue[1]
                  for (const dataChild of dataSource.children) {
                    if (dataChild.name === this.secondFormData.optionsValue[1]) {
                      // console.log(dataChild.name)
                      // console.log(val[1])
                      this.parameters.useId = dataChild.schemaId
                    }
                  }
                }
              }
              if (this.tab.table !== '') {
                baseRequest('/basic/tables/tableOptions', { schemaId: this.parameters.useId }).then(response => {
                  this.tableDataQuery = response.data.item
                  if (this.tableDataQuery !== []) {
                    const laborious = this.tableDataQuery
                    if (this.currentTable) {
                      this.$refs.tableQuery.selectedStatusById(this.currentTable.tableid)
                    } else {
                      for (const labor of laborious) {
                        if (labor.tablename === this.tab.table) {
                          this.parameters.tableQuerySelection = labor
                          this.$refs.tableQuery.selectedStatus(labor)
                        }
                      }
                    }
                  }
                })
              }

              this.updateFormData.optionsVal = true
            }
          })
        }
      }
    }

  },
  mounted() {
    baseRequest('/basic/sqoops/getMessage').then(response => {
      this.options = response.data.item
    })
    baseRequest('/basic/sqoops/getMessages').then(response => {
      this.targetOptions = response.data.item
    })
    //   baseRequest('/sqoops/getMessage').then(response => { //请求目标数据源
    //   this.options = response.data.item
    // })
  },
  methods: {
    importAndExport(val) { // 导入导出切换 清空步骤内容
      this.secondFormData.optionsValue.length = 0
      this.updateFormData.optionsVal = false
      this.tableDat = []
      this.currentTable = null
      this.thirdFormData.targetOptionsValue.length = 0
    },
    // 重新赋值
    changeTableData(tableData) {
      this.tableDataInput = tableData
    },
    radioChange(val) {
      if (val === '指定表') {
        if (this.currentTable) {
          this.$refs.tableQuery.selectedStatusById(this.currentTable.tableid)
        } else {
          for (const labor of this.tableDataQuery) {
            if (labor.tablename === this.tab.table) {
              this.parameters.tableQuerySelection = labor
              this.$refs.tableQuery.selectedStatus(labor)
            }
          }
        }
      }
    },
    changeCurrentTable(val) {
      this.currentTable = val
    },
    handleCurrentChange(val) { // 单选选中
      this.currentRow = val
      this.thirdFormData.tablePartition = null

      baseRequest('/basic/sqoops/getHivePartitions', { targetTableName: val.tablename, targetDataSourceCode: 'hiveDataSource', targetSchema: this.targeterSchema }).then(response => {
        this.targetPartition = response.data.item
        if (response.data.item.length === 0) {
          this.zoningSwitch = false
          this.thirdFormData.tablePartition = null
        } else {
          this.zoningSwitch = true
        }
      })
    },
    typeChange() {
      if (this.updateFormData.type.length === 0) {
        this.updateFormData.sqoopQueue = ''
      }
    },
    clearTable(val) {
      // console.log(val)
      // console.log(this.updateFormData.optionsValue)
      if (this.secondFormData.optionsValue.length === 0) {
        this.updateFormData.optionsVal = false
      } else {
        for (const dataSource of this.options) {
          if (dataSource.name === val[0]) {
            this.parameters.Identification = dataSource.code
            this.parameters.dataSourceName = val[1]
            for (const dataChild of dataSource.children) {
              if (dataChild.name === val[1]) {
                // console.log(dataChild.name)
                // console.log(val[1])
                this.parameters.useId = dataChild.schemaId
              }
            }
          }
        }
        baseRequest('/basic/tables/tableOptions', { schemaId: this.parameters.useId }).then(response => {
          this.tableDataQuery = response.data.item
        })
        this.updateFormData.optionsVal = true
      }
    },
    targetClearTable(val) {
      this.targeterSchema = val[1]
      if (val.length !== 0) {
        for (const targetSource of this.targetOptions) {
          if (targetSource.name === val[0]) {
            for (const targetChild of targetSource.children) {
              if (targetChild.name === val[1]) {
                baseRequest('/basic/tables/tableOptions', { schemaId: targetChild.schemaId }).then(response => {
                  this.tableDat = response.data.item
                })
              }
            }
          }
        }
      } else {
        this.tableDat = []
      }
    },
    openModification(row) {
      baseRequest('/basic/t04ExcelImportTemplate/select', { collectTemplateId: row.collectTemplateId }).then(response => {
        this.updateFormData = response.data.item
        this.colmunListVisable = true
        // console.log('hahah')
      })
    },
    // 上一步
    compeleteReturn() {
      if (this.active === 1 && this.updateFormData.sqoopType === 'export') {
        this.record = this.thirdFormData.tablePartition
      }
      if (this.active === 1 && this.secondFormData.optionsValue.length === 0) {
        this.active--
        return
      }
      if (this.active === 1 && this.updateFormData.sqoopType === 'import') {
        if (this.secondFormData.criteria === '指定表') {
          if (this.$refs.tableQuery.getColumns()) {
            const shbgon = this.$refs.tableQuery.getColumns()
            for (const ljbd of shbgon) {
              ljbd.column = ljbd.columnname
              ljbd.flag = true
              delete ljbd.columnname
              delete ljbd.columnchnname
              delete ljbd.columnid
            }
            this.parameters.tableQueryColnmus = shbgon
          }
        }
      }

      if (this.currentRow) {
        this.tableSelection = this.currentRow
      }

      if (this.active === 1 && this.secondFormData.criteria === 'SQL查询语句') {
        this.parameters.sql = this.$refs.sqlInput.getSqlValue()
      }
      if (this.active === 2 && this.updateFormData.sqoopType === 'import') {
        this.tableSelection = this.currentRow
        this.record = this.thirdFormData.tablePartition
      }

      this.active--
    },
    // 下一步/保存
    compeleteCurrent() {
      if (this.active === 0) {
        this.$refs.fistForm.validate(valid => {
          if (valid) {
            this.param.sqoopType = this.updateFormData.sqoopType
            this.parameters.tableExport = this.updateFormData.sqoopType
            this.param.sqoopQueue = this.updateFormData.sqoopQueue
            // param.dataSourceCode = this.updateFormData.optionsValue[0]

            this.param.taskNum = this.updateFormData.taskNum
            this.active = 1
          }
        })
      } else if (this.active === 1) {
        if (this.updateFormData.sqoopType === 'export') { // 导出第二步
          this.$refs.thirdForm.validate(valid => {
            if (valid) {
              if (this.thirdFormData.targetOptionsValue.length === 0) {
                return
              } else {
                this.param.targetDataSourceCode = 'hiveDataSource'
                this.param.targetSchema = this.thirdFormData.targetOptionsValue[1]
              }
              if (this.currentRow) {
                this.tableSelection = this.currentRow
                this.param.targetTable = this.currentRow.tablename
              } else {
                this.$Message.error('点击选表')
                return
              }
              if (this.thirdFormData.tablePartition) {
                this.param.partition = this.thirdFormData.tablePartition
                this.record = this.thirdFormData.tablePartition
              }
              this.active = 2
            }
          })
        } else if (this.updateFormData.sqoopType === 'import') { // 导入第二步
          this.$refs.secondForm.validate(valid => {
            if (valid) {
              for (const souce of this.options) {
                if (souce.name === this.secondFormData.optionsValue[0]) {
                  this.param.dataSourceCode = souce.code
                }
              }
              this.param.schemaName = this.secondFormData.optionsValue[1]
              if (this.secondFormData.criteria === 'SQL查询语句') {
                if (this.param.table) {
                  delete this.param.table
                }
                let sqlValue = this.$refs.sqlInput.getSqlValue()
                this.parameters.sql = this.$refs.sqlInput.getSqlValue()
                sqlValue = sqlValue.replace(/^\s+|\s+$/g, '')
                const tabNull = this.tab ? this.tab.query : {}
                if (tabNull && tabNull === sqlValue) {
                  this.param.query = sqlValue
                  const allColumn = this.$refs.sqlInput.tableData
                  this.param.columns = allColumn
                  this.active = 2
                } else {
                  if (sqlValue !== '' && sqlValue !== null) {
                    const clickMonitor = this.$refs.sqlInput.monitor
                    if (sqlValue === clickMonitor) {
                      const allColumns = this.$refs.sqlInput.tableData
                      if (allColumns) {
                        this.param.query = sqlValue
                        this.param.columns = allColumns
                        this.active = 2
                      } else {
                        this.$Message.error('检查sql语句是否正确')
                        return
                      }
                    } else {
                      this.$Message.error('请点击查询字段按钮')
                      return
                    }
                  } else {
                    this.$Message.error('查询语句不能为空')
                    return
                  }
                }
              } else if (this.secondFormData.criteria === '指定表') {
                if (this.param.query) {
                  delete this.param.query
                }
                const tableValue = this.$refs.tableQuery.getTableValue()
                if (tableValue) {
                  this.param.table = this.currentTable.tablename
                  this.param.condition = this.secondFormData.condition
                  // param.columns = this.$refs.tableQuery.getColumns()
                  // param.condition = this.updateFormData.condition
                  // console.log(param.table)
                  // console.log(param.columns)
                  // const tableAll = this.$refs.tableQuery.tableDate
                  // for (const col of tableAll) {
                  //   col.flag = false
                  // }
                  const tablecolumns = this.$refs.tableQuery.getColumns()
                  this.parameters.tableQueryColnmus = tablecolumns
                  if (tablecolumns.length === 0) {
                    this.$Message.error('字段不能为空')
                    return
                  }
                  // for (const forTable of tableAll) {
                  //   for (const forColunm of tablecolumns) {
                  //     if (forTable.column === forColunm.column) {
                  //       forTable.flag = true
                  //     }
                  //   }
                  // }
                  // param.columns = tableAll
                  for (const ljbd of tablecolumns) {
                    ljbd.column = ljbd.columnname
                    ljbd.flag = true
                    delete ljbd.columnname
                    delete ljbd.columnchnname
                    delete ljbd.columnid
                  }

                  this.param.columns = tablecolumns
                  this.active = 2
                } else {
                  this.$Message.error('点击选表')
                  return
                }
              }
            }
          })
        }
      } else if (this.active === 2) {
        if (this.updateFormData.sqoopType === 'export') { // 导出第三步
          this.$refs.secondForm.validate(valid => {
            if (valid) {
              for (const souce of this.options) {
                if (souce.name === this.secondFormData.optionsValue[0]) {
                  this.param.dataSourceCode = souce.code
                }
              }
              this.param.schemaName = this.secondFormData.optionsValue[1]
              const tableValue = this.$refs.tableQuery.getTableValue()
              if (tableValue) {
                this.param.table = this.currentTable.tablename
              } else {
                this.$Message.error('点击选表')
                return
              }
              const jsonScript = JSON.stringify(this.param)
              const id = this.currentTable ? this.currentTable.tableid : null
              const name = this.currentTable ? this.currentTable.tablename : null
              baseRequest('/basic/sqoops/add', { sqoopId: this.sqoopId, sqoopName: this.updateFormData.sqoopName, sqoopScript: jsonScript, tableName: name, tableId: id, targetTableName: this.currentRow.tablename }).then(() => {
                this.$Message.success('操作成功')
                this.$parent.searchOption()
                this.value = []
                this.colmunListVisable = false
              })
            }
          })
        } else if (this.updateFormData.sqoopType === 'import') { // 导入第三步
          this.$refs.thirdForm.validate(valid => {
            if (valid) {
              if (this.thirdFormData.targetOptionsValue.length === 0) {
                return
              } else {
                this.param.targetDataSourceCode = 'hiveDataSource'
                this.param.targetSchema = this.thirdFormData.targetOptionsValue[1]
              }
              if (this.currentRow) {
                this.param.targetTable = this.currentRow.tablename
              } else {
                this.$Message.error('点击选表')
                return
              }
              if (this.thirdFormData.tablePartition) {
                this.param.partition = this.thirdFormData.tablePartition
              }
              const jsonScript = JSON.stringify(this.param)
              const id = this.currentTable ? this.currentTable.tableid : null
              const name = this.currentTable ? this.currentTable.tablename : null
              const targetName = this.currentRow ? this.currentRow.tablename : null
              baseRequest('/basic/sqoops/add', { sqoopId: this.sqoopId, sqoopName: this.updateFormData.sqoopName, sqoopScript: jsonScript, tableName: name, tableId: id, targetTableName: targetName }).then(() => {
                this.$Message.success('操作成功')
                this.$parent.searchOption()
                this.value = []
                this.colmunListVisable = false
              })
            }
          })
        }
      }
    },
    modifiedEcho(row) { // 修改
      this.sqoopId = row.sqoopId
      baseRequest('/basic/sqoops/getMessage').then(response => {
        this.options = response.data.item
        baseRequest('/basic/sqoops/getMessages').then(response => {
          this.targetOptions = response.data.item
          baseRequest('/basic/sqoops/select', { sqoopId: row.sqoopId }).then(response => {
            this.active = 0
            const modified = response.data.item
            const jsonObj = JSON.parse(modified.sqoopScript)
            this.tab = jsonObj
            this.updateFormData.sqoopName = modified.sqoopName
            this.updateFormData.sqoopType = jsonObj.sqoopType
            if (jsonObj.sqoopQueue) {
              this.updateFormData.type = [' ']
              this.updateFormData.sqoopQueue = jsonObj.sqoopQueue
            }
            this.updateFormData.taskNum = jsonObj.taskNum
            for (let i = 0; i < this.options.length; i++) {
              if (this.options[i].code === jsonObj.dataSourceCode) {
                for (let j = 0; j < this.options[i].children.length; j++) {
                  if (this.options[i].children[j].name === jsonObj.schemaName) {
                    this.secondFormData.optionsValue = [this.options[i].name, this.options[i].children[j].name]
                    this.parameters.useId = this.options[i].children[j].schemaId
                  }
                }
              }
            }
            if (this.secondFormData.optionsValue !== []) {
              this.updateFormData.optionsVal = true
            }

            this.parameters.sql = jsonObj.query
            if (!jsonObj.query) {
              this.secondFormData.criteria = '指定表'
              this.parameters.tableQueryColnmus = jsonObj.columns
            } else {
              this.secondFormData.criteria = 'SQL查询语句'
              this.tableDataInput = jsonObj.columns
              baseRequest('/basic/tables/tableOptions', { schemaId: this.parameters.useId }).then(response => {
                this.tableDataQuery = response.data.item
              })
            }
            this.parameters.Identification = jsonObj.dataSourceCode
            this.parameters.dataSourceName = jsonObj.schema
            this.secondFormData.condition = jsonObj.condition
            this.targeterSchema = jsonObj.targetSchema
            for (let c = 0; c < this.targetOptions.length; c++) { // 目标数据源赋值
              if (this.targetOptions[c].code === jsonObj.targetDataSourceCode) {
                for (let k = 0; k < this.targetOptions[c].children.length; k++) {
                  if (this.targetOptions[c].children[k].name === jsonObj.targetSchema) {
                    this.thirdFormData.targetOptionsValue = [this.targetOptions[c].name, this.targetOptions[c].children[k].name]
                    baseRequest('/basic/tables/tableOptions', { schemaId: this.targetOptions[c].children[k].schemaId }).then(response => {
                      this.tableDat = response.data.item
                      if (jsonObj.targetTable) {
                        for (const tableAl of this.tableDat) {
                          if (tableAl.tablename === jsonObj.targetTable) {
                            // this.$refs.targeterTable.setCurrentRow(tableAl)
                            this.tableSelection = tableAl
                          }
                        }
                      }
                    })
                  }
                }
              }
            }

            if (jsonObj.partition.length === 0) {
              this.zoningSwitch = false
            } else {
              baseRequest('/basic/sqoops/getHivePartitions', { targetTableName: modified.targetTableName, targetDataSourceCode: 'hiveDataSource', targetSchema: jsonObj.targetSchema }).then(response => {
                this.targetPartition = response.data.item
                if (response.data.item.length === 0) {
                  this.zoningSwitch = false
                  this.thirdFormData.tablePartition = null
                } else {
                  this.thirdFormData.tablePartition = jsonObj.partition
                  this.record = jsonObj.partition
                  this.zoningSwitch = true
                }
              })
            }
            // this.$refs.sqlInput.setSql()
          })
        })
      })
      // this.updateFormData.optionsValue = []
    },
    clearForm() {
      this.active = 0
      this.updateFormData.sqoopName = ''
      this.updateFormData.sqoopQueue = ''
      this.secondFormData.condition = ''
      this.secondFormData.criteria = '指定表'
      this.updateFormData.sqoopType = 'import'
      this.updateFormData.type = []
      this.tab = null
      this.secondFormData.optionsValue = []
      this.thirdFormData.targetOptionsValue = []
      this.parameters.tableQueryColnmus = []
      this.tableDat = []
      this.parameters.sql = null
      this.thirdFormData.tablePartition = null
      this.updateFormData.optionsVal = false
      this.updateFormData
      this.$nextTick(_ => {
        this.$refs.form.clearValidate()
      })
    }
  }
}

</script>
<style lang="scss" >
.sqoop-input {
  .baseUpdate-form {
    padding: 0 38px;
    margin-left: 0;
    .input-icon::after {
      content: "*";
      color: #f56c6c;
      margin-left: 4px;
    }
  }
}
.tempalate-modification {
  .el-transfer-panel:last-child {
    width: 750px;
  }
  .el-transfer-panel__item {
    display: block;
  }
}
</style>