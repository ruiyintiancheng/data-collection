/*
 * @Author: 1k 
 * @Date: 2019-08-09 10:10:43 
 * @Last Modified by: wk
 * @Last Modified time: 2020-05-21 10:16:48
 * @Description:  excel文件上传
 */
<template>
  <div>
    <el-dialog :close-on-click-modal="false"
               v-el-drag-dialog
               title="上传模板"
               append-to-body
               :visible.sync="inputParamFormVisible"
               width="50%"
               custom-class="dialog-default">
      <div class="dialog-contant-default-addOrUpdate">
        <el-form class="baseUpdate-form"
                 v-if="refreshTable"
                 ref="addForm"
                 :rules="addRules"
                 :model="updateFormData"
                 label-width="100px"
                 label-position="right">
          <el-form-item label="数据源选择"
                        prop="optionsValue">
            <el-cascader :options="options"
                         clearable
                         @change="clearTable"
                         v-model="updateFormData.optionsValue"
                         :props="SetKesDept"
                         placeholder=""
                         style="width:155px">
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
                       style="width:155px">
              <el-option v-for="item in selectOption"
                         :key="item.tableid"
                         :label="item.tablename"
                         :value="item.tablename">
              </el-option>
            </el-select>
          </el-form-item>

          <el-upload class="upload-demo"
                     ref="uploaded"
                     :action='url'
                     :headers="myHeaders"
                     :before-upload="beforeAvatarUpload"
                     :on-success="uploadSuccess"
                     :on-exceed="onExceed"
                     multiple
                     :limit="1"
                     style="margin-left:45px"
                     :file-list="fileList">
            <span style="font-size: 14px;font-weight: 700;">上传xml文件 </span>
            <el-button size="small"
                       type="primary"
                       style="margin-left:45px">点击上传</el-button>
            <!-- <div slot="tip"
                 class="el-upload__tip"
                 style="color:red">只能上传xml文件</div> -->
          </el-upload>
        </el-form>
      </div>
      <div slot="footer"
           class="dialog-footer-addOrUpdate">
        <el-button @click="inputParamFormVisible = false">关闭</el-button>
        <!-- <el-button type="primary"
                   @click="saveOperate()">保存</el-button> -->
      </div>
    </el-dialog>
  </div>
</template>
<script>
import { baseRequest } from '@/api/base'
import { getToken } from '@/utils/auth'
export default {
  name: 'templateConfigurationComponentsExcelFile',

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
      myHeaders: { token: getToken() },
      fileList: [],
      upNumber: true,
      url: process.env.BASE_API + '/t04ExcelImportTemplate/uploadXML',
      updata: { // 上传参数
        collectTemplateType: '2'

      },
      updateFormData: {
        optionsValue: [],
        tableName: ''
      },
      inputParamFormVisible: false, // 输入参数添加修改弹窗
      addRules: {
        tableName: [{
          required: true, message: '请选择表'
        }],
        optionsValue: [{
          required: true, message: '请选数据源'
        }]
      }
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
      console.log(this.updateFormData)
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
          console.log(this.updateFormData.optionsValue)

          this.loading = false
        })
      }
    },
    // 超过限制
    onExceed() {
      this.$message.info('只能上传一个文件,请先删除已存在文件')
    },
    // 上传成功
    uploadSuccess(response, file, fileList) {
      const data = response.code
      if (data === 1) {
        this.$refs.addForm.validate(valid => {
          if (valid) {
            this.updata.tableName = this.updateFormData.tableName
            this.updata.tableId = this.updateFormData.tableId
            this.updata.fileId = response.data.item.fileId
            baseRequest('/basic/t04ExcelImportTemplate/insertByXML', this.updata).then(response => {
              this.$parent.searchOption()
              this.inputParamFormVisible = false
              this.$message({
                message: '模板上传成功',
                type: 'success'
              })
            })
          } else {
            this.$Message.error('选择模板名称')
            this.fileList = []
            return
          }
        })
      } else {
        this.$message.error(response.msg)
      }
      // if (data !== '' && data !== null) {
      //   this.requestResult = 1
      // } else {
      //   this.requestResult = 2
      // }
      // setTimeout(() => {
      //   this.requestResult = 0
      // }, 2000)
      // this.$refs.uploaded.clearFiles
      // console.log(response.data.item.fileId)
    },
    // 上传验证
    beforeAvatarUpload(file) {
      console.log('1')

      // this.$refs.addForm.validate(valid => {
      //   if (valid) {
      //     this.updata.tableName = this.updateFormData.tableName
      //     this.updata.tableId = this.updateFormData.tableId

      //     // const arr = file.name.split('.')
      //     // const typeRight = arr[arr.length - 1] === 'xml' || arr[arr.length - 1] === 'XML'
      //     // if (!typeRight) {
      //     //   this.$message.error('上传文件只能是 xml 格式!')
      //     //   this.upNumber = false
      //     // }
      //     // return typeRight
      //   } else {
      //     this.$Message.error('上传失败,请选择表!')
      //     this.upNumber = false
      //   }
      // })
      // return this.upNumber
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
        if (this.updateFormData.hasOwnProperty(key)) {
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
    saveOperate() {
      this.inputParamFormVisible = false
      // const param = this.updateFormData
      // param.deployNodeId = this.paramData.deployNodeId
      // param.deployFlowId = this.paramData.deployFlowId
      // saveUpdate('/taskNotice/add', this.updateFormData, this.addRules, this.$refs.addForm).then(() => {
      //   this.$Message.success('操作成功')
      //   this.inputParamFormVisible = false
      //   this.updateFormData.userId = ''
      //   this.userIdAll = ''
      // })
    },
    clearForm() {
      this.updateFormData = {}
      this.fileList = []
      this.$nextTick(_ => {
        this.$refs.addForm.clearValidate()
      })
    }
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
    padding-top: 30px;
  }
}
</style>