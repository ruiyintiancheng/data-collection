/*
 * @Author: 1k 
 * @Date: 2019-08-09 10:10:43 
 * @Last Modified by: wk
 * @Last Modified time: 2020-05-21 10:18:04
 * @Description:  excel文件上传
 */
<template>
  <div>
    <el-dialog :close-on-click-modal="false"
               v-el-drag-dialog
               title="导入数据"
               append-to-body
               :visible.sync="inputParamFormVisible"
               width="50%"
               custom-class="dialog-default">
      <div class="dialog-contant-default-addOrUpdate">
        <el-form class="baseUpdate-form"
                 ref="addForm"
                 :rules="addRules"
                 :model="updateFormData"
                 label-width="100px"
                 label-position="right">
          <el-form-item label="模板类型"
                        prop="collectTemplateType">
            <el-select class="form-input"
                       clearable
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
          <el-form-item label="首行状态"
                        prop="headState"
                        v-if="updateFormData.collectTemplateType === '1'">
            <el-select class="form-input"
                       clearable
                       v-model="updateFormData.headState"
                       placeholder="">
              <el-option label="首行为列名"
                         value="0">
              </el-option>
              <el-option label="首行非列名"
                         value="1">
              </el-option>
            </el-select>
          </el-form-item>

          <el-form-item label="模板名称"
                        prop="collectTemplateName">
            <el-select v-model="updateFormData.collectTemplateName"
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
                         :key="item.collectTemplateId"
                         :label="item.collectTemplateName"
                         :value="item.collectTemplateName">
              </el-option>
            </el-select>
          </el-form-item>

          <el-upload class="upload-demo"
                     v-if="loadingShow"
                     ref="uploaded"
                     :action='url'
                     :headers="myHeaders"
                     :before-upload="beforeAvatarUpload"
                     :on-success="uploadSuccess"
                     :on-exceed="onExceed"
                     :on-progress='upLoad'
                     multiple
                     :limit="1"
                     style="margin-left:45px"
                     :file-list="fileList">
            <span style="font-size: 14px;font-weight: 700;">上传文件 </span>
            <el-button size="small"
                       type="primary"
                       style="margin-left:45px">点击上传</el-button>
            <!-- <div slot="tip"
                 class="el-upload__tip"
                 style="color:red">只能上传xml文件</div> -->
          </el-upload>
          <div style="padding-left:25px;"
               v-else> <i class="el-icon-loading"></i> 文件上传中</div>
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
  name: 'templateFileRecordingComponentsExcelFile',

  data() {
    return {
      loadingShow: true,
      loading: false,
      selectOption: [],
      myHeaders: { token: getToken() },

      fileList: [],
      upNumber: true,
      url: '',
      updata: { // 上传参数
        collectTemplateType: '2'

      },
      updateFormData: {},
      inputParamFormVisible: false, // 输入参数添加修改弹窗
      addRules: {
        collectTemplateName: [{
          required: true, message: '请选择模板'
        }],
        collectTemplateType: [{
          required: true, message: '请选择模板类型'
        }],
        headState: [{
          required: true, message: '请选择首行状态'
        }]
      }
    }
  },
  computed: {
    chooseValue: function() {
      return this.updateFormData.collectTemplateType
    }
  },
  watch: {
    chooseValue(val) {
      if (val === '1') {
        this.url = process.env.BASE_API + '/import/uploadCsv'
      } else if (val === '2') {
        this.url = process.env.BASE_API + '/import/uploadExcel'
      }
    }
  },
  methods: {
    getTableId(val) {
      // this.updateFormData.tableName = val
      console.log(val)
      for (const item of this.selectOption) {
        if (item.collectTemplateName === val) {
          this.updateFormData.collectTemplateId = item.collectTemplateId
        }
      }

      // this.updateFormData.tableName = this.updateFormData.tableItem.tableName
      // this.updateFormData.tableId = this.updateFormData.tableItem.tableId
    },
    remoteMethod(query) { // 查表名
      if (query !== '') {
        this.loading = true
        baseRequest('/basic/t04ExcelImportTemplate/selectTemplate', { collectTemplateName: query }).then(response => {
          this.selectOption = response.data.item
          // console.log(this.updateFormData.optionsValue)

          this.loading = false
        }).catch(() => {
          this.$message.error('请求模板名失败,不为空')
        })
      }
    },
    // 超过限制
    onExceed() {
      this.$message.info('只能上传一个文件,请先删除已存在文件')
    },
    upLoad() { // 文件上传时
      this.loadingShow = false
      // alert(1)
    },
    // 上传成功
    uploadSuccess(response, file, fileList) {
      const data = response.code
      if (data === 1) {
        this.loadingShow = true
        this.updata.fileId = response.data.item.fileId
        if (this.updateFormData.collectTemplateType === '1') {
          baseRequest('/basic/import/importCsv', this.updata).then(response => {
            this.$parent.searchOption()
            this.inputParamFormVisible = false
            this.$message({
              message: '导入数据成功',
              type: 'success'
            })
          })
        } else {
          baseRequest('/basic/import/importExcel', this.updata).then(response => {
            this.$parent.searchOption()
            this.inputParamFormVisible = false
            this.$message({
              message: '导入数据成功',
              type: 'success'
            })
          })
        }
      } else {
        this.$message.error(response.msg)
        this.loadingShow = true
        this.fileList = []
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
      this.$refs.addForm.validate(valid => {
        if (valid) {
          // this.updata.tableName = this.updateFormData.tableName
          this.updata.collectTemplateId = this.updateFormData.collectTemplateId
          if (this.updateFormData.headState) {
            this.updata.headState = this.updateFormData.headState
          }

          // const arr = file.name.split('.')
          // const typeRight = arr[arr.length - 1] === 'xml' || arr[arr.length - 1] === 'XML'
          // if (!typeRight) {
          //   this.$message.error('上传文件只能是 xml 格式!')
          //   this.upNumber = false
          // }
          // return typeRight
        } else {
          this.$Message.error('上传失败,请填写完整信息!')
          this.upNumber = false
        }
      })
      return this.upNumber
    },
    // 表补全
    tableCompelete(queryString, cb) {
      if (!this.updateFormData.collectTemplateName) {
        this.updateFormData.collectTemplateId = null
        cb([])
        return
      }
      baseRequest('/basic/t04ExcelImportTemplate/selectTemplate', { collectTemplateName: queryString }).then(response => {
        cb(response.data.item)
      })
    },
    // 选中赋值
    handleAutoSelect(obj) {
      for (const key in obj) {
        if (this.updateFormData.hasOwnProperty(key)) {
          if (obj[key] !== null && typeof (obj[key]) !== 'undefined') {
            if (key === 'collectTemplateName') {
              this.updateFormData[key] = obj.collectTemplateName
              this.updateFormData.collectTemplateId = obj.collectTemplateId
            } else {
              this.updateFormData[key] = obj[key]
            }
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
      this.selectOption = []
      this.loadingShow = true
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