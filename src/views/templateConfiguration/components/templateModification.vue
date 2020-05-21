/*
 * @Author: 1k 
 * @Date: 2019-08-06 10:16:38 
 * @Last Modified by: wk
 * @Last Modified time: 2020-05-21 10:16:58
 * @Description:  模板添加修改
 */
<template>
  <el-dialog title="字段列表"
             append-to-body
             v-el-drag-dialog
             :close-on-click-modal="false"
             :visible.sync="colmunListVisable"
             width="80%"
             custom-class="dialog-default">
    <div class="dialog-contant-default tempalate-modification"
         v-if="colmunListVisable">
      <el-steps :active="active"
                finish-status="success"
                simple>
        <el-step title="条件筛选"></el-step>
        <el-step title="列排序"></el-step>
      </el-steps>
      <template v-if="active === 0">
        <el-form class="baseUpdate-form"
                 v-if="refreshTable"
                 ref="form"
                 :rules="addRules"
                 :model="updateFormData"
                 label-width="100px"
                 label-position="left"
                 style="margin-top:10px">
          <el-form-item label="模板名称"
                        prop="collectTemplateName">
            <el-input class="form-input"
                      style="width:234px;"
                      v-model="updateFormData.collectTemplateName"
                      clearable></el-input>
          </el-form-item>
          <el-form-item label="数据源选择"
                        prop="optionsValue">
            <el-cascader :options="options"
                         clearable
                         @change="clearTable"
                         v-model="updateFormData.optionsValue"
                         :props="SetKesDept"
                         placeholder=""
                         style="width:234px;">
              <template slot-scope="{ node, data }">
                <span>{{data.name}}</span>
              </template>
            </el-cascader>
          </el-form-item>
          <el-form-item label="表名称"
                        prop="tableName">
            <el-select v-model="updateFormData.tableName"
                       filterable
                       remote
                       clearable
                       reserve-keyword
                       placeholder=""
                       @change="getTableId"
                       :remote-method="remoteMethod"
                       :loading="loading"
                       style="width:234px;">
              <el-option v-for="item in selectOption"
                         :key="item.tableid"
                         :label="item.tablename"
                         :value="item.tablename">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="模板类型"
                        prop="collectTemplateType">
            <el-select class="form-input"
                       clearable
                       style="width:234px;"
                       v-model="updateFormData.collectTemplateType"
                       placeholder="">
              <el-option label="csv导入"
                         value="1">
              </el-option>
              <el-option label="excel导入"
                         value="2">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="sheet索引"
                        v-if="updateFormData.collectTemplateType === '2'"
                        prop="sheetIndex">
            <el-input size="small"
                      class="form-input"
                      style="width:234px;"
                      v-model="updateFormData.sheetIndex"
                      clearable></el-input>
          </el-form-item>

          <el-form-item label="开始行数"
                        v-if="updateFormData.collectTemplateType === '2'"
                        prop="startRow">
            <el-input size="small"
                      class="form-input"
                      style="width:234px;"
                      v-model="updateFormData.startRow"
                      clearable></el-input>
          </el-form-item>
          <el-form-item label="分隔符"
                        v-if="updateFormData.collectTemplateType === '1'"
                        prop="delimiter">
            <el-input size="small"
                      class="form-input"
                      style="width:234px;"
                      v-model="updateFormData.delimiter"
                      clearable></el-input>
          </el-form-item>
          <el-form-item label="文本识别符"
                        v-if="updateFormData.collectTemplateType === '1'"
                        prop="textIdentifier">
            <el-input size="small"
                      class="form-input"
                      style="width:234px;"
                      v-model="updateFormData.textIdentifier"
                      clearable></el-input>
          </el-form-item>

        </el-form>
      </template>
      <!-- 步骤二 -->

      <div v-else>
        <el-transfer v-model="value"
                     :data="data"
                     :titles="['未选字段', '已选字段']"
                     :props="{
                        key: 'columnid',
                        label: 'columnname'
                      }"
                     :render-content="test"></el-transfer>

      </div>

      <!-- <column-list ref="columnList"
                   v-if="active === 0"
                   :dataSourceCondition="dataSourceCondition"></column-list>
      <column-order ref="columnOrder"
                    v-else
                    :orderData="orderData"
                    :tableData="tableData"></column-order> -->
    </div>
    <div slot="footer"
         class="dialog-footer">
      <el-button @click="colmunListVisable = false">取消</el-button>
      <el-button v-if="active !== 0"
                 @click="compeleteReturn">上一步</el-button>
      <el-button @click="compeleteCurrent"
                 type="primary">{{active===1?'完成':'下一步'}}</el-button>
    </div>
  </el-dialog>
</template>
<script>
// import columnList from './columnList'
// import columnOrder from './columnOrder'
import { baseRequest } from '@/api/base'
// import { saveUpdate } from '@/utils/validate'
export default {
  name: 'templateConfigurationComponentsTemplateModification',
  components: {
    // columnList,
    // columnOrder
  },
  // props: {
  //   dataSourceCondition: Object
  // },
  data() {
    return {
      refreshTable: true,
      loading: false,
      selectOption: [],
      SetKesDept: {
        value: 'id',
        label: 'name',
        type: 'type',
        children: 'children'
      }, // 自定义  级联选择器value label
      options: [],
      data: [],
      value: [],
      item: {
        dsId: null,
        schemaId: null,
        tableId: null,
        tableName: null,
        columnId: null,
        columnName: null,
        relationship: '1',
        type: '1'
      },
      addRules: {// 校验
        collectTemplateName: [{
          required: true, message: '必填!!'
        }],
        tableName: [{
          required: true, message: '必填!!'
        }],
        collectTemplateType: [{
          required: true, message: '必填!!'
        }],
        sheetIndex: [{
          required: true, message: '必填!!'
        }],
        startRow: [{
          required: true, message: '必填!!'
        }],
        delimiter: [{
          required: true, message: '必填!!'
        }],
        textIdentifier: [{
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
        optionsValue: [],
        tableName: ''
      }
    }
  },
  computed: {

  },
  watch: {
    colmunListVisable() {
      this.active = 0
    }
  },
  mounted() {
    baseRequest('/basic/dataResources/getTrees').then(response => {
      var options = response.data.item[0].children
      delete options.name
      delete options.id
      delete options.type
      this.options = options
    })
  },
  methods: {
    clearTable() {
      console.log(123123)
      this.updateFormData.tableName = ''
      this.selectOption.length = 0
    },
    getTableId(val) {
      // this.updateFormData.tableName = val
      for (const item of this.selectOption) {
        if (item.tablename === val) {
          this.updateFormData.tableId = item.tableid
        }
      }
      this.refreshTable = false
      this.$nextTick(_ => {
        this.refreshTable = true
      })
      // this.updateFormData.tableName = this.updateFormData.tableItem.tableName
      // this.updateFormData.tableId = this.updateFormData.tableItem.tableId
    },
    remoteMethod(query) { // 查表名
      if (query !== '' && this.updateFormData.optionsValue[1]) {
        this.loading = true
        baseRequest('/basic/tables/tableOptions', { tableName: query, schemaId: this.updateFormData.optionsValue[1], pageSize: 5, pageNo: 1 }).then(response => {
          this.selectOption = response.data.item
          // console.log(this.updateFormData.optionsValue)
          this.loading = false
        })
      }
    },
    // 表补全
    tableCompelete(queryString, cb) {
      if (!this.updateFormData.tableName) {
        this.updateFormData.tableId = null
        cb([])
        return
      }
      baseRequest('/basic/tables/tableOptions', { tableName: queryString }).then(response => {
        cb(response.data.item)
      })
    },
    // 选中赋值
    handleAutoSelect(obj) {
      for (const key in obj) {
        if (this.item.hasOwnProperty(key)) {
          if (obj[key] !== null && typeof (obj[key]) !== 'undefined') {
            if (key === 'tableName') {
              this.updateFormData[key] = obj.tableName
              this.updateFormData.tableId = obj.tableId
            } else {
              this.updateFormData[key] = obj[key]
            }
          } else {
            this.updateFormData[key] = null
          }
        }
      }
    },
    // 上一步
    compeleteReturn() {
      this.active = 0
    },
    // 下一步/保存
    compeleteCurrent() {
      if (this.active === 0) {
        console.log(1111111)
        // this.tableData = this.$refs.columnList.tableData
        // this.orderData = this.$refs.columnList.orderData
        this.$refs.form.validate(valid => {
          if (valid) {
            baseRequest('/basic/columns/getColumnsByTableId', { tableId: this.updateFormData.tableId }).then(response => {
              this.data = response.data.item
              this.active++
            })
          } else {
            console.log('error submit!!')
            return false
          }
        })
      } else {
        const arr = []
        this.data.forEach((key, value) => {
          // console.log(key, value)
          var obj = {}
          // obj.value = key
          for (let i = 0; i < this.value.length; i++) {
            var columnOrder = document.querySelector('#ppp-options-' + this.value[i]).value
            var columnParseType = document.querySelector('#ppp-select-' + this.value[i]).value
            if (this.value[i] === key.columnid) {
              obj.columnId = key.columnid
              obj.columnName = key.columnname
              obj.columnOrder = columnOrder
              obj.columnParseType = columnParseType
              arr.push(obj)
            }
          }
        })
        for (const num of arr) {
          if (!(/(^[+]{0,1}(\d+)$)/.test(num.columnOrder))) {
            this.$message.error('字段列数为正整数')
            return
          } else if (!num.columnParseType) {
            this.$message.error('请选择解析类型')
            return
          }
        }
        console.log('1')
        const param = {}
        param.tableId = this.updateFormData.tableId
        param.tableName = this.updateFormData.tableName
        param.collectTemplateName = this.updateFormData.collectTemplateName
        param.collectTemplateType = this.updateFormData.collectTemplateType
        param.templateColumns = arr
        if (this.updateFormData.collectTemplateType === '1') {
          param.delimiter = this.updateFormData.delimiter
          param.textIdentifier = this.updateFormData.textIdentifier
        } else {
          param.sheetIndex = this.updateFormData.sheetIndex
          param.startRow = this.updateFormData.startRow
        }

        baseRequest('/basic/t04ExcelImportTemplate/add', param).then(() => {
          this.$Message.success('操作成功')
          this.$parent.searchOption()
          this.updateFormData = {}
          this.value = []
          this.colmunListVisable = false
        })
        // const params = {
        //   sqlId: this.dataSourceCondition.sqlId,
        //   dataSourceCode: this.dataSourceCondition.dataSourceCode,
        //   tableData: this.tableData,
        //   orderData: this.$refs.columnOrder.data
        // }
        // baseRequest('/sqlColumn/add', params).then(_ => {
        //   this.$message.success('操作成功')
        //   this.colmunListVisable = false
        // })
      }
    },
    test(h, option) {
      if (this.value.find(item => option.columnid === item)) {
        // return h('div', [
        //   h('span', option.columnName),
        //   h('span', '    字段对应列数 : '),
        //   h('input', {
        //     attrs: {
        //       id: 'ppp-options-' + option.columnId
        //     }
        //   }),
        //   h('span', '    字段解析类型 : '),
        //   h('select', { children: [
        //     h('option ', {
        //       attrs: {
        //         value: '1'
        //       },
        //       domProps: {
        //         innerHTML: '字符串'
        //       }
        //     })
        //   ]
        //   })
        // ]
        // )
        return <div>
          <span>{option.columnname}</span>
          <span style='margin:0 10px;'>字段对应列数 :</span>
          <input id={'ppp-options-' + option.columnid} />
          <span style='margin:0 10px;'>字段解析类型  :</span>
          <select name='cars' id={'ppp-select-' + option.columnid}>
            <option value='1'>字符串</option>
            <option value='2'>日期</option>
          </select>
        </div>
      } else {
        return h('div', [
          h('span', option.columnname)
        ]
        )
        // return <div>
        //   <span>{option.columnname}</span>
        // </div>
      }
    },
    ccc() {
      const arr = []
      this.data.forEach((key, value) => {
        // console.log(key, value)

        const obj = {}
        obj.value = key
        const columnOrder = document.querySelector('#ppp-options-' + key.columnid).value
        const columnParseType = document.querySelector('#ppp-select-' + key.columnid).value
        if (!(/(^[+]{0,1}(\d+)$)/.test(columnOrder))) {
          this.$message.error('字段列数为正整数')
          return false
        }
        for (let i = 0; i < this.value.length; i++) {
          if (key.columnid === this.value[i]) {
            obj.columnId = key.columnid
            obj.columnName = key.columnname
          }
        }
        obj.columnOrder = columnOrder
        obj.columnParseType = columnParseType
        arr.push(obj)
      })
      //  (const iterator in this.data) {
      //   const obj = {}
      //   obj.value = iterator
      //   // const option = document.querySelector(iterator[columnId]).value
      //   // obj.option = option
      //   arr.push(obj)
      // }
      console.log(arr, this.value)
    },
    clearForm() {
      this.updateFormData = {}
      this.value = []
      this.$nextTick(_ => {
        this.$refs.form.clearValidate()
      })
    }
  }
}

</script>
<style lang="scss">
.tempalate-modification {
  .el-transfer-panel:last-child {
    width: 750px;
  }
  .el-transfer-panel__item {
    display: block;
  }
}
</style>
