<!--  -->
<template>
  <div>
    <el-select placeholder="请选择品牌！" v-model="brandId"  @change="$emit('update:brandId',brandId)"
               filterable clearable style="width: 100%">
      <el-option v-for="item in brands" :key="item.brandId"
                 :label="item.brandName" :value="item.brandId" ></el-option>
    </el-select>
  </div>
</template>


<script>
// import com from ''
export default {
  components: {},
  data () {
    return {
      catId: 0,
      brands: [],
      brandId: ''
    }
  },
  // 计算属性
  computed: {
  },
  // 监听器
  watch: {
  },
  methods: {
    getCatBrands (catId) {
      this.$http({
        url: this.$http.adornUrl('commodity/categorybrandrelation/list'),
        method: 'get',
        params: this.$http.adornParams({
          catId: catId,
          groupValue: 'brand_name'
        })
      }).then(({data}) => {
        this.brands = data.page.list
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
