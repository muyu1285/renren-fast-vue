<!--  -->
<template>
  <el-cascader v-model="path" :options="categorys" expand-trigger="hover" clearable placeholder="请选择分类！"
               ref="category" filterable separator="=>"
               :props="defaultProps" @change="$emit('update:categoryPath',path)" style="width: 100%"></el-cascader>
</template>

<script>
// import com from ''
export default {
  components: {},
  props: {
    categoryPath: {
      type: Array,
      default () {
        return []
      }
    }
  },
  data () {
    return {
      categorys: [],
      defaultProps: {
        value: 'catId',
        children: 'children',
        label: 'name'
      },
      path: ''
    }
  },
  // 计算属性
  computed: {
  },
  // 监听器
  watch: {
  },
  methods: {
  },
  // 创建完成 （可以访问当前this）
  created () {
  },
  // 挂载完成（可以访问DOM）
  mounted () {
    this.$http({
      url: this.$http.adornUrl('commodity/category/list/tree'),
      method: 'get'
    }).then(({data}) => {
      this.categorys = data.data
    })
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
