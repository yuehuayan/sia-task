<template>
  <div class="mask-add-task-manage" id="mask">
      <div class="mask-content" id="mask-content">
        <div class="mask-main-title">
          <span>添加 Job</span>
          <i class="close-icon" @click="showHiddenAddJob"></i>
        </div>
        <div class="info">
          <el-form :model="taskAddViewModel" :rules="jobAddViewModelRules" ref="jobManageViewForm" label-width="120px" class="jobManageViewForm" auto-complete="off">
            <el-form-item label="Job_Group" prop="jobGroupName">
              <el-select v-model="taskAddViewModel.jobGroupName" placeholder="Job_Group" @visible-change="handChangeWidth">
                <el-option v-for="(item,index) in jobGroupList" :key="index" :label="item" :value="item"></el-option>
              </el-select>
              <el-tooltip class="item" effect="dark" content="选择Job归属的项目组，可选列表中一般包含多个项目组" placement="right">
                <span class="el-icon-warning info-icon"></span>
              </el-tooltip>
            </el-form-item>
            <el-form-item label="Job_Key" prop="jobKey">
              <span class="pre-job-key-text">{{taskAddViewModel.jobGroupName}}</span><span class="center-line">－</span>
              <el-input type="text" auto-complete="off" placeholder="" v-model="taskAddViewModel.jobKey"  class="pre-job-key"></el-input>
              <el-tooltip class="item" effect="dark" content="Job唯一标识,以Job_Group为前缀" placement="right">
                <span class="el-icon-warning info-icon"></span>
              </el-tooltip>
            </el-form-item>
            <el-form-item label="Job类型" prop="jobType">
              <el-select v-model="taskAddViewModel.jobType" placeholder="请选择Job类型">
                <el-option v-for="(item,index) in jobTypeList" :key="index" :label="item" :value="item"></el-option>
              </el-select>
              <el-tooltip placement="right">
                <div slot="content">选择Job类型： </br>TRIGGER_TYPE_CRON （固定时刻，后续输入CRON表达式）；</br>TRIGGER_TYPE_FIXRATE（固定频率，后续输入启动时间，执行次数和执行间隔）</div>
                <span class="el-icon-warning info-icon"></span>
              </el-tooltip>
            </el-form-item>



            <el-form-item v-if="taskAddViewModel.jobType === 'TRIGGER_TYPE_CRON'" label="Job类型值" prop="jobTypeValue">
              <el-input @change="changeInputValue" type="text" auto-complete="off" placeholder="请输入Job类型值" v-model="taskAddViewModel.jobTypeValue">
              </el-input>
              <!--<el-popover ref="popoverCron" popper-class="cron-popover-box" width="630" v-model="isShowCronBox" placement="right">
                <cron @change="changeCron" @close="isShowCronBox=false"></cron>
              </el-popover>
              <el-button size="mini" class="cron-box" v-popover:popoverCron >cron</el-button>-->
              <el-tooltip class="item" effect="dark" content="输入合法的CRON表达式" placement="right">
                <span class="el-icon-warning info-icon"></span>
              </el-tooltip>
            </el-form-item>



            <div v-if="taskAddViewModel.jobType === 'TRIGGER_TYPE_FIXRATE'">
              <el-form-item label="启动时间" prop="starTime">
              <el-date-picker
                  v-model="taskAddViewModel.starTime"
                  type="datetime"
                  placeholder="请选择启动时间">
                </el-date-picker>
                <el-tooltip class="item" effect="dark" content="设置启动时间，如果启动时间滞后于当前时间，则立即执行" placement="right">
                  <span class="el-icon-warning info-icon"></span>
                </el-tooltip>
              </el-form-item>
              <el-form-item label="执行次数" prop="executeNumber">
                <el-input-number v-model="taskAddViewModel.executeNumber" controls-position="right" :min="0" label="请输入执行次数"></el-input-number>
                <el-tooltip class="item" effect="dark" content="设置执行次数，0表示执行次数无限（即无穷大∞）" placement="right">
                  <span class="el-icon-warning info-icon"></span>
                </el-tooltip>
              </el-form-item>
              <el-form-item label="执行间隔(秒)" prop="executeSpace">
                <el-input-number v-model="taskAddViewModel.executeSpace" controls-position="right" :min="0" label="请输入执行间隔"></el-input-number>
                <el-tooltip class="item" effect="dark" content="设置执行间隔，即上一次开始执行的时间和下一次开始执行的时间的差值" placement="right">
                  <span class="el-icon-warning info-icon"></span>
                </el-tooltip>
              </el-form-item>
            </div>
            <el-form-item label="预警邮箱(前缀)" prop="emailWarning">
              <el-input type="text" auto-complete="off" placeholder="请输入预警邮箱" v-model="taskAddViewModel.emailWarning"></el-input>
              <el-tooltip class="item" effect="dark" content="输入预警邮箱前缀，多个请用邮箱全程并以英文逗号隔开" placement="right">
                <span class="el-icon-warning info-icon"></span>
              </el-tooltip>
            </el-form-item>
            <el-form-item label="描述" prop="jobDescribe" class="desc-input">
              <el-input type="textarea" auto-complete="off" placeholder="请输入描述内容" v-model="taskAddViewModel.jobDescribe"></el-input>
              <el-tooltip class="item" effect="dark" content="描述Job的功能" placement="right">
                <span class="el-icon-warning info-icon"></span>
              </el-tooltip>
            </el-form-item>
          </el-form>
          <div class="alert-button">
            <el-button class="blue-button" @click="showHiddenAddJob">取消</el-button>
            <el-button class="blue-button" @click="showHiddenSave">添加</el-button>
          </div>
        </div>
      </div>
  </div>
</template>

<script>
import moment from 'moment'
import {cron} from 'vue-cron'
export default {
  name: 'AddJobTmpl',
  props: ['addParamsSearch'],
  components: {cron},
  data () {
    var checkBlur = (rule, value, callback) => {
      switch (rule.field) {
        case 'emailWarning':
          // if (!new RegExp(/^(([a-zA-Z0-9_.-])+@((********)+.)+(cn)+(,){0,1}){0,}$/).test(value)) {
          //   return callback(new Error('邮箱格式为xxx@********.cn'))
          // }
          break
        case 'jobKey':
          if (value.replace(/(^\s*)|(\s*$)/g, '') !== '' && !new RegExp(/^[^\/]{2,40}$/).test(value)) {
            return callback(new Error('输入内容不能含有"/"且长度为2到40个字符!'))
          }
          break
        case 'jobDescribe':
          if (value.replace(/(^\s*)|(\s*$)/g, '') !== '' && !new RegExp(/^[^]{2,1000}$/).test(value)) {
            return callback(new Error('输入内容必须是2到1000个字符!'))
          }
          break
        case 'jobTypeValue':
          if (value === '') {
            return callback(new Error('请输入Job类型值'))
          }
          break
        case 'starTime':
          if (value === '') {
            return callback(new Error('请选择启动时间'))
          }
          break
        case 'executeNumber':
          if (value === '') {
            return callback(new Error('请输入执行次数'))
          }
          if (value % 1 !== 0) {
            return callback(new Error('执行次数必须为整数'))
          }
          break
        case 'executeSpace':
          if (value === '') {
            return callback(new Error('请输入执行间隔'))
          }
          if (value % 1 !== 0) {
            return callback(new Error('执行间隔必须为整数'))
          }
          break
      }
      callback()
    }
    return {
      isShowCronBox: false, // cron表达式是否显示
      jobGroupList: [],
      jobTypeList: ['TRIGGER_TYPE_CRON', 'TRIGGER_TYPE_FIXRATE'],
      taskAddViewModel: {
        jobGroupName: '',
        jobKey: '',
        jobType: '',
        jobTypeValue: '',
        starTime: '',
        executeNumber: '',
        executeSpace: '',
        emailWarning: '',
        jobDescribe: ''
      },
      jobAddViewModelRules: {
        jobGroupName: [this.$validator.required('请输入Job_Group')],
        jobKey: [this.$validator.required('请输入Job_Key'), { validator: checkBlur, trigger: 'blur' }],
        jobTypeValue: [this.$validator.required('请输入Job类型值'), { validator: checkBlur, trigger: 'blur' }],
        jobType: [this.$validator.required('请输入Job类型')],
        jobDescribe: [this.$validator.required('请输入描述内容'), { validator: checkBlur, trigger: 'blur' }],
        starTime: [this.$validator.required('请选择启动时间'), { validator: checkBlur, trigger: 'blur' }],
        executeNumber: [this.$validator.required('请输入执行次数'), { validator: checkBlur, trigger: 'blur' }],
        executeSpace: [this.$validator.required('请输入执行间隔'), { validator: checkBlur, trigger: 'blur' }],
        emailWarning: [this.$validator.required('请输入预警邮箱'), { validator: checkBlur, trigger: 'blur' }]
      }
    }
  },
  watch: {
    'taskAddViewModel.jobType': function (newVal, oldVal) {
      this.$refs.jobManageViewForm.validate(this.taskAddViewModel.jobTypeValue)
    },
    'taskAddViewModel.jobTypeValue': function (newVal, oldVal) {
      this.$refs.jobManageViewForm.validate(this.taskAddViewModel.jobTypeValue)
    }
  },
  created () {
    this.taskAddViewModel.jobType = this.jobTypeList[0]
    this.getSearchList()
  },
  methods: {
    changeCron (val) {
      this.taskAddViewModel.jobTypeValue = val
      this.$refs['popoverCron'].doClose()
      this.changeInputValue(val)
    },
    changeInputValue: function (val) {
      let self = this
      if (self.taskAddViewModel.jobType !== 'TRIGGER_TYPE_FIXRATE' && val !== '') {
        self.$http.get(self.$api.getApiAddress('/jobapi/cronexpression', 'CESHI_API_HOST'), {
          cron: val
        }).then((res) => {
          if (res.data.code !== 0) {
            self.$message({message: res.data.message, type: 'error'})
            self.taskAddViewModel.jobTypeValue = ''
            self.$refs.jobManageViewForm.validate(self.taskAddViewModel.jobTypeValue)
          }
        }).catch(() => {
          self.$message({message: '输入内容不可用！', type: 'error'})
        })
      }
    },
    handChangeWidth: function () {
      document.getElementsByClassName('el-select-dropdown')[0].style.minWidth = '360px'
    },
    getSearchList: function () {
      let self = this
      self.$http.get(self.$api.getApiAddress('/taskapi/selectAuth', 'CESHI_API_HOST')).then((res) => {
        if (res.data.code === 0) {
          let data = res.data.data
          self.jobGroupList = data
          self.taskAddViewModel.jobGroupName = self.jobGroupList[0]
          self.$refs.jobManageViewForm.validate(self.taskAddViewModel.jobGroupName)
        } else {
          self.$message({message: res.data.message, type: 'error'})
        }
      }).catch(() => {
        self.$message({message: '服务未响应！', type: 'error'})
      })
    },
    showHiddenSaveIpPost: function () {
    },
    showHiddenAddJob: function () {
      this.$emit('showHiddenAddJob', false, this.addParamsSearch)
    },
    showHiddenSave: function () {
      let self = this
      let jobValueArr = moment(new Date(self.taskAddViewModel.starTime)).format('YYYY-MM-DD HH:mm:ss') + ',' + self.taskAddViewModel.executeNumber + ',' + self.taskAddViewModel.executeSpace
      let modalObj = {
        jobKey: self.taskAddViewModel.jobGroupName + '_' + self.taskAddViewModel.jobKey,
        jobGroup: self.taskAddViewModel.jobGroupName,
        jobTrigerType: self.taskAddViewModel.jobType,
        jobTrigerValue: self.taskAddViewModel.jobType === 'TRIGGER_TYPE_CRON' ? self.taskAddViewModel.jobTypeValue : jobValueArr,
        jobDesc: self.taskAddViewModel.jobDescribe,
        jobAlarmEmail: self.taskAddViewModel.emailWarning
      }
      let jobListParams = {
        jobGroupName: self.taskAddViewModel.jobGroupName,
        jobKeyName: ''
      }
      this.$refs.jobManageViewForm.validate(valid => {
        if (valid) {
          self.$http.post(
            self.$api.getApiAddress('/jobapi/insertJob', 'CESHI_API_HOST'), modalObj)
            .then((res) => {
              switch (res.data.code) {
                case 0:
                  self.$message({message: '添加成功', type: 'success'})
                  self.$emit('showHiddenAddJob', false, jobListParams)
                  break
                default:
                  // self.$message({message: 'Job_Key已存在！', type: 'error'})
                  self.$message({message: res.data.message, type: 'error'})
                  self.$emit('showHiddenAddJob', false, jobListParams)
                  break
              }
            })
            .catch(() => {
              self.$message({message: '添加失败', type: 'error'})
              self.$emit('showHiddenAddJob', false, self.addParamsSearch)
            })
        }
      })
    }
  }
}
</script>
<style lang="less">
@import '../styles/common/add-job.tmpl.less';
</style>
