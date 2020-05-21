/*
 * @Author: 1k 
 * @Date: 2019-08-19 15:47:44 
 * @Last Modified by: 1k
 * @Last Modified time: 2019-09-24 11:45:51
 * @Description:  查询表
 */
<template>
  <div>
    <div>
      <div v-show="searchToggle"
           class="form-search clearfix">
        <div class="tableoption">
          <el-table ref="singleTable"
                    :data="tableData.filter(data => !search || data.tablename.toLowerCase().includes(search.toLowerCase()))"
                    height="200"
                    highlight-current-row
                    @current-change="handleCurrentChange"
                    style="width: 100%">
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
        </div>
        <div class="tableoption"
             v-if="parameters.tableExport==='import'">
          <el-table ref="multipleTable"
                    :data="tableDate"
                    tooltip-effect="dark"
                    style="width:100%"
                    height="200"
                    @selection-change="handleSelectionChange">
            <el-table-column type="selection"
                             width="55">
            </el-table-column>
            <el-table-column prop="columnname"
                             label="字段名">
            </el-table-column>
          </el-table>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import { baseRequest } from '@/api/base'

export default {
  props: {
    parameters: Object,
    tableData: Array
  },
  data() {
    return {
      alreadyHasCollumn: false,
      tableQueryId: null,
      search: '',
      multipleSelection: [], // 选中的列
      tableDate: [], // 列数据
      currentRow: null, // 表选中的数据
      // tableData: [], // 表数据
      searchToggle: true,
      updateFormData: {},
      tableToggle: false,
      pageNo: 1,
      pageSize: null,
      total: 350,
      tableHead: [

      ]
      // tableData: [
      //   { ADD_TIME: null, DICT_CODE: 'basicDictType', DICT_ID: 1, DICT_TYPE: 'basicDictType', DISP_ORDER: 0, MSG_ID: 33, PARENT_DICT_ID: 0 }
      // ]
    }
  },
  mounted() {
    // this.getTableData()
  },
  methods: {
    selectedStatus(row) { // 修改默认选中
      this.$refs.singleTable.setCurrentRow(row)
      this.tableQueryId = row.tableid
    },
    selectedStatusById(id) { // 修改默认选中
      this.$nextTick(_ => {
        this.$refs.singleTable.setCurrentRow(this.tableData.find(item => item.tableid === id))
        this.tableQueryId = id
      })
    },
    getTableData() {
      baseRequest('/basic/tables/tableOptions', { schemaId: this.parameters.useId }).then(response => {
        this.tableData = response.data.item
      })
    },
    handleSelectionChange(val) { // 选中列
      this.multipleSelection = val
    },
    handleCurrentChange(val) { // 单选选中
      this.currentRow = val
      this.$emit('changeCurrentTable', val)
      baseRequest('/basic/columns/getColumnsByTableId', { tableId: val.tableid }).then(response => {
        this.tableDate = response.data.item
        if (this.tableQueryId === val.tableid) {
          const tableColmns = []
          for (const syg of this.tableDate) {
            for (const rnlb of this.parameters.tableQueryColnmus) {
              if (syg.columnname === rnlb.column) {
                tableColmns.push(syg)
              }
            }
          }
          this.$nextTick(_ => {
            for (const tableAll of this.tableDate) {
              for (const tableOne of tableColmns) {
                if (tableAll.columnname === tableOne.columnname) {
                  this.$refs.multipleTable.toggleRowSelection(tableAll, true)
                }
              }
            }
            this.alreadyHasCollumn = true
          })
        } else if (this.tableQueryId !== val.tableid) {
          this.$nextTick(_ => {
            this.$refs.multipleTable.toggleAllSelection()
          })
        }
      })
    },
    getTableValue() {
      if (this.currentRow) {
        return this.currentRow.tablename
      }
    },
    getColumns() { // 获取列名
      return this.multipleSelection
    }
    // searchOption() {
    //   console.log(2)
    // }
  }
}
</script>>
<style scoped>
.form-search {
  padding: 0 !important;
}
.tableoption {
  /* display: inline-block; */
  width: 50%;
  float: left;
  margin-top: 2px;
  border-top: 1px solid #ccc;
  border-bottom: 1px solid #ccc;
}
.tableoption:first-of-type {
  border-right: 1px solid #ccc;
  border-left: 1px solid #ccc;
}
.tableoption:last-of-type {
  border-right: 1px solid #ccc;
}
.clearfix:after {
  content: "";
  clear: both;
  display: block;
  height: 0;
}
</style>