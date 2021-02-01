<!--  -->
<template>
  <el-row>
    <el-col :span="6"><div class="grid-content bg-purple">
      <CategoryMenu @treeNodeClick="treeNodeClick"></CategoryMenu>
    </div></el-col>
    <el-col :span="18"><div class="grid-content bg-purple-light">
      <AttrgroupItem ref="attrgroupItem"></AttrgroupItem>
    </div></el-col>
  </el-row>
</template>

<script>
import CategoryMenu from './common/category-menu'
import AttrgroupItem from './attrgroup-item'
export default {
  components: {CategoryMenu, AttrgroupItem},
  data () {
    return {}
  },
  // 计算属性
  computed: {
  },
  // 监听器
  watch: {
  },
  methods: {
    treeNodeClick (data, node, component) {
      console.log(data)
      this.$nextTick(() => {
        this.$refs.attrgroupItem.$refs.addOrUpdate.dataForm.catelogId = data.catId
        this.$refs.attrgroupItem.$refs.addOrUpdate.dataForm.catelogName = data.name
        this.$refs.attrgroupItem.getDataList(data.catId)
      })
    }
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
