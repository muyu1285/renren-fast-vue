<!--  -->
<template>
  <el-tree :data="menus" :props="defaultProps" accordion node-key="catId" @node-click="nodeClick">
      <span class="custom-tree-node" slot-scope="{ node, data }">
          <span>{{ node.label }}</span>
        </span>
  </el-tree>
</template>

<script>
// import com from ''
export default {
  components: {},
  data () {
    return {
      menus: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  // 计算属性
  computed: {
  },
  // 监听器
  watch: {
  },
  methods: {
    nodeClick (data, node, component) {
      if (node.level === 3) {
        this.$emit('treeNodeClick', data, node, component)
      }
    }
  },
  // 创建完成 （可以访问当前this）
  created () {
    this.$http({
      url: this.$http.adornUrl('commodity/category/list/tree'),
      method: 'get'
    }).then(({data}) => {
      this.menus = data.data
    })
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
