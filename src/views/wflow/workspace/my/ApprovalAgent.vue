<template>
  <div>
    <el-alert title="👨‍💼 设置您的审批代理人，让其在指定时间段内为您处理所有审批任务" type="success" :closable="false"/>
    <el-form :model="agent" style="margin-top: 20px" label-width="70px">
      <el-form-item label="代理时限" prop="timeRange">
        <el-date-picker v-model="agent.timeRange" type="datetimerange" range-separator="至"
                        start-placeholder="开始日期" end-placeholder="结束日期" value-format="yyyy-MM-dd HH:mm" format="yyyy-MM-dd HH:mm"/>
      </el-form-item>
      <el-form-item label="代理人" prop="user">
        <div style="display:flex;">
          <el-button style="margin-right: 20px;" icon="el-icon-user" @click="$refs.orgPicker.show()">设置代理人</el-button>
          <avatar :size="35" :name="agent.user.name" :src="agent.user.avatar" v-if="agent.user"></avatar>
        </div>
      </el-form-item>
      <el-form-item label="取消代理" v-if="agent.effective">
        <el-button type="warning" size="small" icon="el-icon-refresh-left" @click="cancelAgent" plain>取消代理人</el-button>
      </el-form-item>
      <el-form-item label="">
        <el-button type="primary" @click="setUserAgent">设置</el-button>
        <el-button type="danger" @click="$tab.closePage()">关闭</el-button>
      </el-form-item>
    </el-form>
    <org-picker title="请选择要切换的用户" type="user" ref="orgPicker" :selected="agent.user ? [agent.user] : []" @ok="selected"/>

  </div>
</template>

<script>
import { getUserAgent, setUserAgent, cancelUserAgent } from '@/api/wflow/org'
import OrgPicker from '@/components/common/OrgPicker.vue'

export default {
  name: 'ApprovalAgent',
  components: { OrgPicker },
  data() {
    return {
      agent: {
        timeRange: null,
        user: null,
        effective: false,
      },
    }
  },
  computed: {
    loginUser() {
      return this.$store.state.user.loginUser
    },
  },
  mounted() {
    this.getUserAgent()
  },
  methods: {
    selected(users) {
      this.agent.user = users[0]
    },
    getUserAgent() {
      getUserAgent().then((rsp) => {
        if (rsp.data) {
          this.agent = rsp.data
        } else {
          this.agent.timeRange = null
          this.agent.user = null
          this.agent.effective = false
        }
      }).catch((err) => {
        this.$err(err, '获取用户审批代理人失败')
      })
    },
    setUserAgent(call) {
      if (this.agent.timeRange && this.agent.user) {
        if (this.agent.user.id === this.loginUser.id) {
          this.$message.warning('不允许设置本人为代理人')
          return
        }
        this.$confirm(
          '在代理时间内您的所有审批任务都将转交给审批代理人，确定要提交设置吗?',
          '提示',
          {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning',
          }
        ).then(() => {
          setUserAgent(this.agent).then((rsp) => {
            this.$ok(rsp, '设置审批代理人成功')
            this.getUserAgent()
          }).cache((err) => {
            this.$err(err, '设置用户审批代理人失败')
          })
        })
      } else {
        this.$message.warning('请完善设置信息')
      }
    },
    cancelAgent() {
      this.$confirm('取消审批代理人设置后所有的审批任务都将回归到您来处理，确定要取消?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
        }
      ).then(() => {
        cancelUserAgent().then((rsp) => {
          this.$ok(rsp, '取消审批代理人设置成功')
          this.getUserAgent()
        }).cache((err) => {
          this.$err(err, '取消审批代理人设置失败')
        })
      })
    },
  },
}
</script>

<style scoped></style>
