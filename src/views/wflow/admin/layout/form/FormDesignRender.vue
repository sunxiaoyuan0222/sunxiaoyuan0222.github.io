<template>
  <component ref="form" :index="index" :formData="formData" :readonly="readonly" :is="config.name" :mode="mode" v-model="_value" v-bind="config.props"/>
</template>
<script>

import components from '@/views/wflow/common/form/ComponentExport'

export default {
  name: "FormRender",
  components: components,
  props:{
    mode:{
      type: String,
      default: 'DESIGN'
    },
    readonly:{
      type: Boolean,
      default: false
    },
    value: {
      default: undefined
    },
    config:{
      type: Object,
      default: ()=>{
        return {}
      }
    },
    index:{
      type: Number,
      default: 0
    },
    formData:{
      type: Object,
      default: ()=>{
        return {}
      }
    }
  },
  computed: {
    _value: {
      get() {
        return this.value;
      },
      set(val) {
        this.$emit("input", val);
      }
    }
  },
  data() {
    return {}
  },
  methods: {
    validate(call){
      if (this.$refs.form.validate){
        this.$refs.form.validate(call)
      }else {
        call(true)
      }
    }
  }
}
</script>

<style lang="less" scoped>

</style>
