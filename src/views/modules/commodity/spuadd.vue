<template>
  <div>
    <el-row>
      <el-col :span="24">
        <el-steps :active="stepNumber" finish-status="success" process-status="finish">
          <el-step title="基本信息"></el-step>
          <el-step title="规格参数"></el-step>
          <el-step title="销售属性"></el-step>
          <el-step title="SKU信息"></el-step>
          <el-step title="保存完成"></el-step>
        </el-steps>
      </el-col>
      <el-col :span="24" v-show="stepNumber === 0">
        <stepOne @nextStep="nextStep" ref="stepOne"></stepOne>
      </el-col>
      <el-col :span="24" v-show="stepNumber === 1">
        <stepTwo @nextStep="nextStep" ref="stepTwo"></stepTwo>
      </el-col>
      <el-col :span="24" v-show="stepNumber === 2">
        <stepThree @nextStep="nextStep" ref="stepThree"></stepThree>
      </el-col>
      <el-col :span="24" v-show="stepNumber === 3">
        <stepFour @nextStep="nextStep" ref="stepFour"></stepFour>
      </el-col>
    </el-row>
  </div>
</template>


<script>
import stepOne from './steps/spuadd-step-one'
import stepTwo from './steps/spuadd-step-two'
import stepThree from './steps/spuadd-step-three'
import stepFour from './steps/spuadd-step-four'
export default {
  components: {stepOne, stepTwo, stepThree, stepFour},
  data () {
    return {
      stepNumber: 0
    }
  },
  methods: {
    nextStep (number) {
      let setpObj
      if (number === 1) {
        setpObj = this.$refs.stepTwo
      }
      if (number === 2) {
        setpObj = this.$refs.stepThree
      }
      if (number === 3) {
        this.$refs.stepFour.initData(this)
      }
      if (setpObj) {
        setpObj.getAttrData(this.$refs.stepOne.categoryPath[2])
      }
      this.stepNumber = number
    }
  },

  // 计算属性
  computed: {
  },
  // 监听器
  watch: {
  },
  // 创建完成 （可以访问当前this）
  created () {
  },
  // 挂载完成（可以访问DOM）
  mounted () {
  },
  // 创建之前
  beforeCreate () {},
  // 挂载之前
  beforeMount () {},
  // 更新之前
  beforeUpdate () {},
  // 更新之后
  updated () {},
  // 销毁之前
  beforeDestroy () {},
  // 销毁完成
  destroyed () {},
  // 页面有keep-alive缓存，触发函数
  activated () {}
}
</script>

<style scoped>
</style>
