<template>
  <div>
    <el-form-item label="提示文字">
      <el-input size="small" v-model="value.placeholder" placeholder="请设置提示语"/>
    </el-form-item>
    <el-form-item label="允许编辑">
      <el-switch v-model="value.enableEdit"></el-switch>
    </el-form-item>
  </div>
</template>

<script>
export default {
  name: "Location",
  props:{
    value:{
      type: Object,
      default: ()=>{
        return {}
      }
    }
  },
  components: {},
  data() {
    return {}
  },
  methods: {}
}
</script>

<style scoped>

</style>
