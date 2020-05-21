/*
 * @Author: 1k 
 * @Date: 2019-08-06 18:02:50 
 * @Last Modified by: wk
 * @Last Modified time: 2020-05-21 10:16:16
 * @Description: 修改功能
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
                 ref="form"
                 v-if="refreshTable"
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
                         disabled
                         @change="clearTable"
                         style="width:234px;"
                         v-model="updateFormData.optionsValue"
                         :props="SetKesDept"
                         placeholder="">
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
                       disabled
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
  name: 'templateConfigurationComponentsModification',

  components: {
    // columnList,
    // columnOrder
  },
  props: {
    collectTemplateId: Number,
    tableId: Number
  },
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
        tableName: '',
        tableId: null
      }
    }
  },
  watch: {
    colmunListVisable() {
      this.active = 0
    }
  },
  created() {

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
      // console.log(123123)
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
      this.updateFormData.tableName = val
      this.$nextTick(_ => {
        this.updateFormData.tableName = val
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

          this.loading = false
        })
      }
    },
    openModification(row) { // 获取修改信息
      baseRequest('/basic/t04ExcelImportTemplate/select', { collectTemplateId: row.collectTemplateId }).then(response => {
        this.updateFormData = response.data.item
        const optionsValue = []
        optionsValue.push(response.data.item.dsId)
        optionsValue.push(response.data.item.schemaId)
        this.updateFormData.optionsValue = optionsValue
        this.colmunListVisable = true
        // console.log('hahah')
      })
    },
    // 表补全tableCompelete
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
        this.$refs.form.validate(valid => {
          if (valid) {
            baseRequest('/basic/t04ImportTemplateColumn/selectColumns', { tableId: this.tableId, collectTemplateId: this.collectTemplateId }).then(response => {
              this.data = response.data.item.columns
              this.value = response.data.item.selectColumns
              this.active++
              // console.log('hahah')
            })
          } else {
            console.log('error submit!!')
            return false
          }
        })
      } else {
        const arr = []
        this.data.forEach((key, value) => {
          console.log(key, value)

          const obj = {}
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
        console.log('1')
        for (const num of arr) {
          if (!(/(^[+]{0,1}(\d+)$)/.test(num.columnOrder))) {
            this.$message.error('字段列数为正整数')
            return
          } else if (!num.columnParseType) {
            this.$message.error('请选择解析类型')
            return
          }
        }

        const param = {}
        param.collectTemplateId = this.collectTemplateId
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
        console.log(this.$refs)
        baseRequest('/basic/t04ExcelImportTemplate/update', param).then(() => {
          this.$Message.success('操作成功')
          this.$parent.searchOption()
          this.value = []
          this.colmunListVisable = false
        })
      }
    },
    test(h, option) {
      console.log(option)
      console.log(h)
      if (this.value.find(item => option.columnid === item)) {
        return <div>
          <span>{option.columnname}</span>
          <span style='margin:0 10px;'>字段对应列数 :</span>
          <input id={'ppp-options-' + option.columnid} value={option.columnorder} />
          <span style='margin:0 10px;'>字段解析类型  :</span>
          <select name='cars' id={'ppp-select-' + option.columnid} value={option.columnparsetype}>
            <option value='1'>字符串</option>
            <option value='2'>日期</option>
          </select>
        </div>
      } else {
        return h('div', [
          h('span', option.columnname)
        ]
        )
      }
    },
    clearForm() {
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