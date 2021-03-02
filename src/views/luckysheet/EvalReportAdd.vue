<template>
  <div class="common-wrapper">
    <page-title title="申报" />
    <div v-loading="loading" class="eval-report page-border">
      <div>
        <div class="primary-title">单号：{{ baseInfoForm.serialCode }}</div>
        <el-row type="flex" class="mt-15">
          <el-col :span="16">
            <el-form ref="baseInfoForm" :model="baseInfoForm" label-width="100px" label-position="left">
              <el-row>
                <el-col :span="12">
                  <el-form-item label="被评估机构：">
                    <span
                      v-if="baseInfoForm.evaluationTargetName"
                    >{{ baseInfoForm.evaluationTargetName }}</span>
                    <span v-else>暂无</span>
                  </el-form-item>
                </el-col>
                <el-col v-if="isAppeal||isAppealPreview" :span="12">
                  <el-form-item label="报告日期：">
                    <span v-if="baseInfoForm.reportedDate">
                      {{
                        baseInfoForm.reportedDate*1 | datetime
                      }}
                    </span>
                    <span v-else>暂无</span>
                  </el-form-item>
                </el-col>
                <el-col :span="12">
                  <el-form-item label="任务名称：">
                    <span v-if="baseInfoForm.taskName">{{ baseInfoForm.taskName }}</span>
                    <span v-else>暂无</span>
                  </el-form-item>
                </el-col>
                <el-col :span="12">
                  <el-form-item label="依据模型：">
                    <span v-if="baseInfoForm.modelName">{{ baseInfoForm.modelName }}</span>
                    <span v-else>暂无</span>
                  </el-form-item>
                </el-col>
              </el-row>
            </el-form>
          </el-col>
          <el-col v-if="baseInfoForm.finalScore" :span="8" class="score">
            <div style="margin-top: 10px;">得分</div>
            <div style="font-size: 42px;">{{ baseInfoForm.finalScore }}</div>
          </el-col>
        </el-row>
      </div>
      <div class="primary-title">详情</div>
      <div>
        <el-form ref="baseInfoForm" :model="baseInfoForm" label-width="100px" label-position="left">
          <el-form-item v-if="isFillInPreview" label="填报说明：" style="margin-top:20px;">
            <span v-if="baseInfoForm.submitDescription">{{ baseInfoForm.submitDescription }}</span>
            <span v-else>暂无</span>
          </el-form-item>
          <el-row v-if="this.isAppealPreview">
            <el-col :span="24">
              <el-form-item prop="reason" label="申诉理由：">
                <span v-if="baseInfoForm.reason">{{ baseInfoForm.reason }}</span>
                <span v-else>暂无</span>
              </el-form-item>
            </el-col>
            <el-col :span="24">
              <el-form-item label="申诉依据：">
                <span v-if="baseInfoForm.basis">{{ baseInfoForm.basis }}</span>
                <span v-else>暂无</span>
              </el-form-item>
            </el-col>
            <el-col :span="24">
              <el-form-item label="申诉要求：">
                <span v-if="baseInfoForm.demand">{{ baseInfoForm.demand }}</span>
                <span v-else>暂无</span>
              </el-form-item>
            </el-col>
          </el-row>
          <!-- 申诉 -->
          <el-table
            v-if="isAppeal||isAppealPreview"
            :data="appealListData"
            border
            style="width: 100%;margin:20px 0;"
            size="mini"
          >
            <el-table-column prop="itemName" label="指标名称" align="center" />
            <el-table-column prop="maxScore" label="指标最高得分" align="center" />
            <el-table-column prop="originalScore" label="原分值" align="center" />
            <el-table-column label="申请分值" align="center">
              <template slot-scope="scope">
                <el-input-number
                  v-model="scope.row.demandScore"
                  :min="0"
                  :max="scope.row.maxScore"
                  :precision="0"
                  :disabled="isAppealPreview"
                  @change="handleDemandScore(scope.$index,scope.row.demandScore)"
                />
              </template>
            </el-table-column>
            <el-table-column
              v-if="isAppealPreview&&appealStatus!=='000'"
              prop="approveScore"
              label="审核后分值"
              align="center"
            />
            <el-table-column label="证明材料" align="center">
              <template slot-scope="scope">
                <el-upload
                  :action="uploadUrl"
                  :headers="$utils.uploadHeader()"
                  :before-upload="onBeforeUpload"
                  :on-success="onFileUploadSuccess"
                  :on-error="onFileUploadError"
                  :on-remove="onFileRemoveSuccess"
                  :on-preview="onFilePreview"
                  :file-list="scope.row.proofAttachments"
                  multiple
                  :disabled="isPreview"
                >
                  <el-button
                    v-if="!isPreview"
                    size="small"
                    type="text"
                    @click="uploadBtnClick(scope.$index)"
                  >上传</el-button>
                </el-upload>
              </template>
            </el-table-column>
          </el-table>
          <!-- 填报 -->
          <el-table
            v-if="isFillIn || isFillInPreview"
            :data="fillInListData"
            :span-method="objectSpanMethod"
            class="tableArea"
            style="width: 100%;margin:20px 0;"
          >
            <el-table-column prop="name" label="一级分类" align="center" width="180" />
            <el-table-column prop="subName" label="二级分类" width="180" />
            <!-- <el-table-column prop="itemNo" label="指标编号" /> -->
            <el-table-column prop="itemName" label="指标标题" />
            <el-table-column label="自评结果">
              <template slot-scope="scope">
                <el-radio-group
                  v-if="scope.row.type=='EQUAL'"
                  v-model="scope.row.selfResult"
                  :disabled="isPreview"
                >
                  <el-radio
                    v-for="(item,index) in scope.row.rules"
                    :key="index"
                    :label="item.name"
                    @change="handleSelfResult(scope.$index,item.name)"
                  >
                    <span v-if="scope.row.optionType=='CODE'">{{ item.name|dictText(Dicts[scope.row.optionCode]) }}</span>
                    <span v-else>{{ item.name }}</span>
                  </el-radio>
                </el-radio-group>
                <el-input-number
                  v-if="scope.row.type=='RANGE'"
                  v-model="scope.row.selfResult"
                  :min="0"
                  :disabled="isPreview"
                  @change="handleSelfResult(scope.$index,scope.row.selfResult)"
                />
                <el-checkbox-group
                  v-if="scope.row.type=='MAX'||scope.row.type=='SUM'"
                  v-model="scope.row.selfResult"
                  :disabled="isPreview"
                  @change="handleSelfResult(scope.$index,scope.row.selfResult)"
                >
                  <el-checkbox
                    v-for="(item,index) in scope.row.rules"
                    :key="index"
                    :label="item.name"
                  />
                </el-checkbox-group>
                <el-row v-if="scope.row.type=='WEIGHT'" justify="space-between" type="flex">
                  <div style="width:100%">
                    <el-col
                      v-for="(val, key) in scope.row.selfResult"
                      :key="key"
                      :span="12"
                      :gutter="20"
                    >
                      <span>{{ key }}:</span>
                      <el-input-number
                        v-model="scope.row.selfResult[key]"
                        style="margin-bottom:5px;"
                        :disabled="isPreview"
                        @change="handleSelfResult(scope.$index,scope.row.selfResult[key],key,1)"
                      />
                    </el-col>
                  </div>
                </el-row>
              </template>
            </el-table-column>
            <el-table-column label="证明材料" align="center">
              <template slot-scope="scope">
                <el-upload
                  :action="uploadUrl"
                  :headers="$utils.uploadHeader()"
                  :before-upload="onBeforeUpload"
                  :on-success="onFileUploadSuccess"
                  :on-error="onFileUploadError"
                  :on-remove="onFileRemoveSuccess"
                  :on-preview="onFilePreview"
                  :file-list="scope.row.proofAttachments"
                  multiple
                  :disabled="isPreview"
                >
                  <el-button
                    v-if="!isPreview"
                    size="small"
                    type="text"
                    @click="uploadBtnClick(scope.$index)"
                  >上传</el-button>
                </el-upload>
              </template>
            </el-table-column>
          </el-table>
          <el-form-item v-if="isFillIn" label="填报说明：">
            <el-input
              v-model="baseInfoForm.submitDescription"
              type="textarea"
              :rows="6"
              :disabled="isPreview"
              maxlength="300"
              show-word-limit
            />
          </el-form-item>
          <el-form-item v-if="isAppeal" label="申诉理由：">
            <el-input
              v-model="baseInfoForm.reason"
              type="textarea"
              :rows="6"
              :disabled="isPreview"
              maxlength="300"
              show-word-limit
            />
          </el-form-item>
          <el-form-item v-if="isAppeal" label="申诉依据：">
            <el-input
              v-model="baseInfoForm.basis"
              type="textarea"
              :rows="6"
              :disabled="isPreview"
              maxlength="300"
              show-word-limit
            />
          </el-form-item>
          <el-form-item v-if="isAppeal" label="申诉要求：">
            <el-input
              v-model="baseInfoForm.demand"
              type="textarea"
              :rows="6"
              :disabled="isPreview"
              maxlength="300"
              show-word-limit
            />
          </el-form-item>
          <el-row v-if="baseInfoForm.approveResult=='001'||baseInfoForm.approveResult=='002'">
            <el-col :span="8">
              <el-form-item prop="approveResult" label="审核结果：">
                <span v-if="baseInfoForm.approveResult=='001'">通过</span>
                <span v-if="baseInfoForm.approveResult=='002'">未通过</span>
                <span v-if="baseInfoForm.approveResult=='000'">未审批</span>
              </el-form-item>
            </el-col>
            <el-col :span="8">
              <el-form-item prop="approver" label="审  核  人：">
                <span v-if="baseInfoForm.approver">{{ baseInfoForm.approver }}</span>
                <span v-else>暂无</span>
              </el-form-item>
            </el-col>
            <el-col :span="8">
              <el-form-item prop="approveTime" label="审核时间：">
                <span v-if="baseInfoForm.approveTime">{{ baseInfoForm.approveTime*1 | datetime }}</span>
                <span v-else>暂无</span>
              </el-form-item>
            </el-col>
            <el-form-item label="审核说明：">
              <span v-if="baseInfoForm.approveDescription">{{ baseInfoForm.approveDescription }}</span>
              <span v-else>暂无</span>
            </el-form-item>
          </el-row>

          <el-form-item style="text-align:center">
            <el-button @click="goback()">返回</el-button>
            <el-button
              v-if="isFillIn"
              type="primary"
              :loading="btnLoading"
              @click.prevent="submitFillInForm()"
            >提交</el-button>
            <el-button
              v-if="isAppeal"
              type="primary"
              :loading="btnLoading"
              @click.prevent="submitAppealForm()"
            >提交</el-button>
          </el-form-item>
        </el-form>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: 'EvalReportAdd',
  data() {
    return {
      btnLoading: false,
      loading: false,
      isPreview: this.$route.query.type == 1,
      isFillInPreview: this.$route.query.type == 1 && this.$route.query.processType == 'fillIn', // 填报详情
      isAppealPreview: this.$route.query.type == 1 && this.$route.query.processType == 'appeal', // 申诉详情
      isAppeal: this.$route.query.type == 2 && this.$route.query.processType == 'appeal',
      isFillIn: this.$route.query.type == 2 && this.$route.query.processType == 'fillIn',
      responseCode: this.$route.query.responseCode,
      reportCode: this.$route.query.reportCode,
      appealStatus: this.$route.query.appealStatus,
      // id: this.$route.query.id,
      serialCode: this.$route.query.serialCode,
      appealCode: this.$route.query.appealCode,
      activeName: 'detail',
      baseInfoForm: {
        serialCode: '', // 单号
        evaluationTargetName: '', // 被评估机构
        reportedDate: '', // 报告时区
        taskName: '', // 任务名称
        modelName: '', // 评估模型
        processCode: '',
        // creator: "",
        // cardNo: "",
        // tel: "",
        reason: '',
        basis: '',
        demand: '',
        approveResult: '',
        approver: '',
        approveTime: '',
        approveDescription: '',
        submitDescription: '',
        itemSet: [],
        tel: ''
      },
      curFileUploader: {
        // prop: 'document',
        // limit: 999,
        maxSize: 2
      },
      spanArr: [],
      posFirst: 0,
      posSecond: 0,
      spanArrFirst: [],
      spanArrSecond: [],
      fillInListData: [],
      selfResultVal: [],
      appealListData: []
    }
  },
  computed: {
    uploadUrl() {
      return this.Configs.sys.uploadUrl
    }
  },
  mounted() {
    if (!this.isAppealPreview) {
      this.getProcessData()
    } else {
      this.getAppealById()
    }
  },
  methods: {
    // 获取申诉审核详情
    getAppealById() {
      this.loading = true
      const params = {
        serialCode: this.serialCode
      }
      this.$store
        .dispatch('tssAppeal/getAppealById', params)
        .then(res => {
          this.baseInfoForm = res
          if (this.isAppealPreview) {
            this.baseInfoForm.finalScore = this.$route.query.finalScore
            this.baseInfoForm.evaluationTargetName = this.$route.query.evaluationTargetName
            this.baseInfoForm.taskName = this.$route.query.taskName
            this.baseInfoForm.modelName = this.$route.query.modelName
            this.baseInfoForm.reportedDate = this.$route.query.reportedDate
            this.responseCode = res.responseCode
          }
          this.appealListData = res.itemSet
        })
        .catch(err => {
          this.$message.error(err.message)
        })
        .finally(() => {
          this.loading = false
        })
    },
    // 获取填报审核详情
    getReportDetail() {
      this.loading = true
      const params = {
        serialCode: this.reportCode
      }
      this.$store
        .dispatch('tssEvalReport/getReportDetail', params)
        .then(res => {
          this.loading = false
          this.baseInfoForm.processCode = res.processCode
          // this.baseInfoForm = res;其他数据在process接口里获得
          // debugger;
          const arr2 = res.itemScoreSet
          for (var item1 of arr2) {
            for (var item2 of this.appealListData) {
              if (item2.itemNo == item1.itemNo) {
                item2.originalScore = item1.score,
                item2.demandScore = 0,
                item2.proofAttachments = []// ==========添加新的上传附件
              }
            }
          }
        })
        .catch(err => {
          this.$message.error(err.message)
        })
        .finally(() => {
          this.loading = false
        })
    },
    getFillInDetail() {
      this.loading = true
      const par = {
        serialCode: this.responseCode
      }
      this.$store
        .dispatch('tssFillIn/getFillInById', par)// 查看填报详情
        .then(res => {
          this.baseInfoForm.submitDescription = res.submitDescription
          const arr2 = res.itemSet
          for (var item1 of arr2) {
            for (var item2 of this.fillInListData) {
              if (item2.itemNo == item1.itemNo) {
                item2.selfResult = item1.selfResult
                item2.approveResult = item1.approveResult
                item2.proofAttachments = item1.proofAttachments
              }
            }
          }
          this.loading = false
        })
        .catch(err => {
          this.$message.error(err.message)
        })
        .finally(() => {
          this.loading = false
        })
    },
    getProcessData() {
      this.loading = true
      const par = {
        serialCode: this.appealCode ? this.appealCode : this.serialCode
      }
      this.$store
        .dispatch('tssEvalReport/getOne', par)
        .then(res => {
          this.baseInfoForm.evaluationTargetName = res.evaluationTargetName
          this.baseInfoForm.taskName = res.taskName
          this.baseInfoForm.serialCode = res.serialCode
          this.baseInfoForm.reportedDate = res.reportedDate
          this.baseInfoForm.modelName = res.evaluationBatch.modelName
          this.responseCode = res.responseCode
          res.model.units.forEach(elem1 => {
            // 有二级指标
            if (elem1.sections.length !== 0) {
              elem1.sections.forEach(elem2 => {
                elem2.modelItems.forEach(elem3 => {
                  let selfResult = null
                  if (elem3.item.type == 'MAX' || elem3.item.type == 'SUM') {
                    selfResult = []
                  } else if (elem3.item.type == 'WEIGHT') {
                    selfResult = {}
                    for (var i = 0; i < elem3.rules.length; i++) {
                      selfResult = Object.assign(selfResult, { [elem3.rules[i].name]: 0 })
                    }
                  } else if (elem3.item.type == 'RANGE' || elem3.item.type == 'EQUAL') {
                    selfResult = ''
                  }
                  if (this.isFillIn || this.isAppeal) {
                    this.fillInListData.push({
                      name: elem1.name,
                      subName: elem2.name,
                      itemNo: elem3.item.itemNo,
                      itemName: elem3.item.name,
                      maxScore: elem3.max,
                      selfResult: selfResult,
                      proofAttachments: [],
                      type: elem3.item.type,
                      rules: elem3.rules,
                      optionCode: elem3.item.optionCode ? elem3.item.optionCode : null,
                      optionType: elem3.item ? elem3.item.optionType : null
                    })
                  } else {
                    this.fillInListData.push({
                      name: elem1.name,
                      subName: elem2.name,
                      itemNo: elem3.item.itemNo,
                      itemName: elem3.item.name,
                      maxScore: elem3.max,
                      selfResult: selfResult,
                      type: elem3.item.type,
                      rules: elem3.rules
                    })
                  }
                })
              })
            } else {
              // 无二级指标
              elem1.modelItems.forEach(elem2 => {
                let selfResult = null
                if (elem2.item.type == 'MAX' || elem2.item.type == 'SUM') {
                  selfResult = []
                } else if (elem2.item.type == 'WEIGHT') {
                  selfResult = {}
                  for (var i = 0; i < elem2.rules.length; i++) {
                    selfResult = Object.assign(selfResult, { [elem2.rules[i].name]: 0 })
                  }
                } else if (elem2.item.type == 'RANGE' || elem2.item.type == 'EQUAL') {
                  selfResult = ''
                }
                if (this.isFillIn || this.isAppeal) {
                  this.fillInListData.push({
                    name: elem1.name,
                    subName: null,
                    itemNo: elem2.item.itemNo,
                    itemName: elem2.item.name,
                    maxScore: elem2.max,
                    selfResult: selfResult,
                    proofAttachments: [],
                    type: elem2.item.type,
                    rules: elem2.rules,
                    optionCode: elem2.item.optionCode ? elem2.item.optionCode : null,
                    optionType: elem2.item.optionType ? elem2.item.optionType : null
                  })
                } else {
                  this.fillInListData.push({
                    name: elem1.name,
                    subName: null,
                    itemNo: elem2.item.itemNo,
                    itemName: elem2.item.name,
                    maxScore: elem2.max,
                    selfResult: selfResult,
                    type: elem2.item.type,
                    rules: elem2.rules
                  })
                }
              })
            }
          })
          this.appealListData = this.fillInListData
          if (this.isAppeal) {
            this.getReportDetail()
          }
          if (this.isFillInPreview) {
            this.getFillInDetail()
          }
          if (this.isFillIn || this.isFillInPreview) {
            this.getSpanArrFirst(this.fillInListData)
            this.getSpanArrSecond(this.fillInListData)
          }
          if (this.isAppeal || this.isAppealPreview) {
            this.getSpanArrFirst(this.appealListData)
            this.getSpanArrSecond(this.appealListData)
          }
        })
        .catch(err => {
          this.$message.error(err.message)
        })
        .finally(() => {
          this.loading = false
        })
    },
    goback() {
      this.$router.go(-1)
    },
    // region 文件上传处理
    onBeforeUpload(file) {
      if (file.size >= this.curFileUploader.maxSize * 1024 * 1024) {
        this.$message.warning('超出单个文件大小限制，最大允许上传' + this.curFileUploader.maxSize + 'M！')
        return false
      }
    },
    onFileUploadSuccess(response, file, fileList) {
      this.toFileList(fileList)
    },
    onFileUploadError(err, file, fileList) {
      this.$message.error(err.message ? err.message : err.msg)
    },
    onFileRemoveSuccess(file, fileList) {
      this.toFileList(fileList)
    },
    onFilePreview(file) {
      window.open(this.Configs.sys.uploadUrl + '/' + file.url, '_blank')
    },
    toFileList(fileList) {
      const tmp = []
      fileList && fileList.forEach(file => {
        if (file.response) {
          tmp.push({
            name: file.name,
            url: file.response.fileKey,
            description: file.response.originalFileName
          })
        } else {
          tmp.push({
            name: file.name,
            url: file.url,
            description: file.description
          })
        }
      })
      // if (isAppeal) {
      this.appealListData[this.currentIndex].proofAttachments = tmp
      // } else {
      this.fillInListData[this.currentIndex].proofAttachments = tmp

      // }
    },
    uploadBtnClick(index) {
      this.currentIndex = index
    },
    // endregion
    checkInteger(index, name, key) {
      const reg = /^[0-9]+$/
      if (!reg.test(name)) {
        this.$message.warning('只能输入整数')
        this.$nextTick(() => {
          this.fillInListData[index].selfResult[key] = 0
        })
      }
    },
    handleSelfResult(index, name, key, type) {
      if (type) {
        this.checkInteger(index, name, key)
        this.fillInListData[index].selfResult[key] = name
      } else {
        this.fillInListData[index].selfResult = name
      }
    },
    handleDemandScore(index, val) {
      this.appealListData[index].demandScore = val
    },
    submitFillInForm() {
      const params = {
        ...this.baseInfoForm,
        itemSet: this.fillInListData.map(item => {
          if (item.type == 'RANGE') {
            return {
              ...item,
              selfResult: item.selfResult + ''
            }
          } else {
            return item
          }
        }),
        processCode: this.baseInfoForm.serialCode
      }
      this.btnLoading = true
      this.$store.dispatch('tssFillIn/addFillIn', params).then(res => {
        if (res) {
          this.btnLoading = false
          this.$message.success('提交成功')
          this.$router.push({
            path: '/talent/evalReport'
          })
        }
      }).catch(error => {
        this.$message.error(error.message)
      }).finally(() => {
        this.btnLoading = false
      })
    },
    submitAppealForm() {
      const params = {
        serialCode: this.baseInfoForm.serialCode,
        processCode: this.baseInfoForm.processCode,
        tel: this.baseInfoForm.tel,
        reason: this.baseInfoForm.reason,
        basis: this.baseInfoForm.basis,
        demand: this.baseInfoForm.demand,
        itemSet: this.appealListData.filter(item => {
          return item.demandScore && item.demandScore !== 0
        }),
        approveResult: this.baseInfoForm.approveResult,
        approver: this.baseInfoForm.approver,
        approveDescription: this.baseInfoForm.approveDescription,
        approveTime: this.baseInfoForm.approveTime
      }
      this.btnLoading = true
      this.$store.dispatch('tssAppeal/addAppeal', params).then(res => {
        if (res) {
          this.$message.success('提交成功')
          this.$router.push({
            path: '/talent/evalReport'
          })
        }
      }).catch(error => {
        this.$message.error(error.message)
      })
        .finally(() => {
          this.btnLoading = false
        })
    },
    // 合并一级指标
    getSpanArrFirst(data) {
      for (var i = 0; i < data.length; i++) {
        if (i === 0) {
          this.spanArrFirst.push(1)
          this.posFirst = 0
        } else {
          // 判断当前元素与上一个元素是否相同
          if (data[i].name === data[i - 1].name) {
            this.spanArrFirst[this.posFirst] += 1
            this.spanArrFirst.push(0)
          } else {
            this.spanArrFirst.push(1)
            this.posFirst = i
          }
        }
      }
    },
    // 合并二级指标
    getSpanArrSecond(data) {
      for (var i = 0; i < data.length; i++) {
        if (i === 0) {
          this.spanArrSecond.push(1)
          this.posSecond = 0
        } else {
          // 判断当前元素与上一个元素是否相同
          if (data[i].subName === data[i - 1].subName) {
            this.spanArrSecond[this.posSecond] += 1
            this.spanArrSecond.push(0)
          } else {
            this.spanArrSecond.push(1)
            this.posSecond = i
          }
        }
      }
    },
    objectSpanMethod({ row, column, rowIndex, columnIndex }) {
      if (columnIndex === 0) {
        const _row = this.spanArrFirst[rowIndex]
        const _col = _row > 0 ? 1 : 0
        return {
          rowspan: _row,
          colspan: _col
        }
      }
      if (columnIndex === 1) {
        const _row = this.spanArrSecond[rowIndex]
        const _col = _row > 0 ? 1 : 0
        return {
          rowspan: _row,
          colspan: _col
        }
      }
    }
  }
}
</script>
<style lang="scss">
.eval-report {
  .el-button--medium.is-circle {
    padding: 6px !important;
    margin-right: 5px;
  }
  .el-tabs .el-tabs__item {
    cursor: text !important;
  }
  .el-input-number {
    width: 151px !important;
  }
  .score {
    div {
      text-align: right;
    }
  }

  .el-tabs__nav-wrap::after {
    height: 0 !important;
  }

  .detail-table .el-button + .el-button {
    margin-left: 0 !important;
  }

  .mt-3 {
    margin-top: 10px;

    .el-col {
      margin-top: 8px;
    }
  }
}
</style>
