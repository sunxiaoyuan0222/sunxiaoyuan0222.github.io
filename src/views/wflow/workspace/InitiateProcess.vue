<template>
  <div v-loading="loading">
    <el-row :gutter="20" >
      <el-col :span="15" style=" border-right: 1px solid #e8e8e8;">
        <div v-if="userDepts.length > 1" style="margin-bottom: 10px">
          本次发起部门：
          <el-radio-group size="mini" v-model="userDeptId">
            <el-radio :label="dept.id" :key="dept.id" v-for="dept in userDepts">{{dept.name}}</el-radio>
          </el-radio-group>
        </div>
        <!--渲染表单-->
        <form-render v-if="!loading" class="process-form" ref="form" mode="PC" :forms="forms" v-model="formData"/>
      </el-col>
      <el-col :span="9">
        <!--渲染执行流程-->
        <process-render v-if="!loading && userDeptId" :dept-id="userDeptId" v-model="processUsers" :forms="forms" :formData="formData" :process="process"/>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import ProcessRender from "../process/ProcessRender";
import FormRender from '@/views/wflow/common/form/FormRender'
import FormDesignRender from '@/views/wflow/admin/layout/form/FormDesignRender'
import {getUserDepts} from '@/api/wflow/org'
import {getInstanceFormData} from '@/api/wflow/processTask'
import {getModelById} from '@/api/wflow/modelGroup'
import {startProcess} from '@/api/wflow/process'


export default {
  name: "InitiateProcess",
  components: {FormDesignRender, FormRender, ProcessRender},
  props: {
    code: {
      type: String,
      required: true
    },
    instanceId:{
      type: String,
      required: false
    }
  },
  data() {
    return {
      loading: false,
      formData: {},
      processUsers:{},
      userDepts: [],
      userDeptId: null,
      form: {
        formId: '',
        formName: "",
        logo: {},
        formItems: [],
        process: {},
        remark: ""
      }
    }
  },
  mounted() {
    this.loadFormInfo(this.code)
    this.getUserDept()
  },
  computed: {
    forms() {
      return this.$store.state.design.formItems;
    },
    process(){
      return this.$store.state.design.process;
    }
  },
  methods: {
    getUserDept(){
      getUserDepts(this.$store.state.user.loginUser.id).then(rsp => {
        this.userDepts = rsp.data
        if (this.userDepts.length > 0){
          this.userDeptId = this.userDepts[0].id
        }
      }).catch(err => {
        this.$err(err, '获取用户部门信息失败')
      })
    },
    loadFormData(){
      if (this.$isNotEmpty(this.instanceId)){
        getInstanceFormData(this.instanceId).then(rsp => {
          for (let key in this.formData) {
            this.formData[key] = rsp.data[key]
          }
        }).catch(e => {
          this.$err(e, "加载原始数据失败")
        })
      }
    },
    loadFormInfo(id) {
      this.loading = true;
      this.formData = {}
      getModelById(id).then(rsp => {
        this.loading = false
        let form = rsp.data;
        form.logo = JSON.parse(form.logo)
        // 采购申请表
        if(form.formId == 'wf648fd4bc62b88643e7f166b1') {
           form.formItems = form.formItems.filter(item => {
           if(item.title != '备注'){
                return item
              }
           })
        }
        //付款申请
        if(form.formId == 'wf647dbcca62b8bb5638e31180') {
           form.formItems = form.formItems.filter(item => {
           if(item.title != '银行账号' && item.title != '开户行'){
                return item
              }
           })
        }
        this.form = form
        //构建表单及校验规则
        console.log(form,'form')
        this.$store.state.design = form
        this.loadFormData()
      }).catch(err => {
        this.loading = false
        this.$err(err, '获取流程模型失败')
      })
    },
    validate(call) {
      this.$refs.form.validate(call);
    },
    submit(){
      let startParams = {
        deptId: this.userDeptId,
        formData: this.formData,
        processUsers: this.processUsers
      }
      //this.$message.success("表单填写OK了，提交功能还在开发哦")
      startProcess(this.form.processDefId, startParams).then(rsp => {
        this.$message.success(rsp.data)
        this.$emit('ok')
      }).catch(err => {
        this.$emit('fail')
        this.$err(err, '发起审批失败')
      })
    }
  }
}
</script>

<style lang="less" scoped>
.process-form {
  /deep/ .el-form-item__label {
    padding: 0 0;
    font-weight: normal;
  }
}
</style>
