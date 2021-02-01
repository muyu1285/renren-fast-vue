<!--  -->
<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="searchSkuInfo()" ref="skuFormReset">
      <el-form-item label="分类" prop="categoryPath">
        <CategorySelect :categoryPath.sync="categoryPath"></CategorySelect>
      </el-form-item>
      <el-form-item label="品牌" prop="brandId">
        <BrandSelect :brandId.sync="dataForm.brandId" ref="brandSelect"></BrandSelect>
      </el-form-item>
      <el-form-item label="价格" prop="minPrice">
        <el-input-number style="width:160px" v-model="dataForm.minPrice" :min="0"></el-input-number>&nbsp;&nbsp;&nbsp;-
      </el-form-item>
      <el-form-item prop="maxPrice">
        <el-input-number style="width:160px" v-model="dataForm.maxPrice" :min="0"></el-input-number>
      </el-form-item>
      <el-form-item label="关键字" prop="keyword">
        <el-input placeholder="请输入查询关键字！" style="width:160px" v-model="dataForm.keyword" clearable></el-input>
      </el-form-item>
      <br />
    </el-form>
    <div align="center">
      <el-button icon="el-icon-search" round type="primary" @click="searchSkuInfo">查询</el-button>
    </div>
    <br />
    <el-table :data="dataList" border v-loading="dataListLoading"
        style="width: 100%;" @expand-change="getSkuDetails" >
      <el-table-column type="expand">
        <template slot-scope="scope">
          商品标题：{{scope.row.skuTitle}} <br />
          商品副标题：{{scope.row.skuSubtitle}} <br />
          商品描述：{{scope.row.skuDesc}} <br />
          分类ID：{{scope.row.catalogId}} <br />
          SpuID：{{scope.row.spuId}} <br />
          品牌ID：{{scope.row.brandId}} <br />
        </template>
      </el-table-column>
      <el-table-column prop="skuId" header-align="center" align="center" label="skuId" width="60"></el-table-column>
      <el-table-column prop="skuName" header-align="center" label="名称"></el-table-column>
      <el-table-column prop="skuDefaultImg" header-align="center" align="center" width="130" label="默认图片">
        <template slot-scope="scope">
          <img :src="scope.row.skuDefaultImg" style="width:80px;height:80px;"  alt=""/>
        </template>
      </el-table-column>
      <el-table-column prop="price" header-align="center" align="center" width="120" label="价格"></el-table-column>
      <el-table-column prop="saleCount" header-align="center" align="center" width="100" label="销量"></el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" >预览</el-button>
          <el-button type="text" size="small" >评论</el-button>
          <el-dropdown @command="handleCommand(scope.row,$event)" size="small" type="text">
            <span class="el-dropdown-link" >
              更多<i class="el-icon-arrow-down el-icon--right"></i>
            </span>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item command="uploadImages">上传图片</el-dropdown-item>
              <el-dropdown-item command="seckillSettings">参与秒杀</el-dropdown-item>
              <el-dropdown-item command="reductionSettings">满减设置</el-dropdown-item>
              <el-dropdown-item command="discountSettings">折扣设置</el-dropdown-item>
              <el-dropdown-item command="memberPriceSettings">会员价格</el-dropdown-item>
              <el-dropdown-item command="stockSettings">库存管理</el-dropdown-item>
              <el-dropdown-item command="couponSettings">优惠劵</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination @size-change="sizeChangeHandle" @current-change="currentChangeHandle" :current-page="dataForm.page"
        :page-sizes="[10, 20, 50, 100]" :page-size="dataForm.limit" :total="totalPage" layout="total, sizes, prev, pager, next, jumper"></el-pagination>
  </div>
</template>

<script>
  import CategorySelect from '@/views/modules/commodity/common/category-select'
  import BrandSelect from '@/views/modules/commodity/common/brand-select.vue'
  export default {
    components: {CategorySelect, BrandSelect},
    data () {
      return {
        dataForm: {
          keyword: '',
          brandId: '',
          catelogId: '',
          limit: 10,
          page: 1,
          minPrice: '',
          maxPrice: ''
        },
        dataList: [],
        totalPage: 0,
        dataListLoading: false,
        addOrUpdateVisible: false,
        categoryPath: []
      }
    },
    // 计算属性
    computed: {
    },
    // 监听器
    watch: {
      categoryPath: {
        handler (newVal, oldVal) {
          this.dataForm.catelogId = newVal[2]
          this.$refs.brandSelect.getCatBrands(newVal[2])
        },
        deep: true
      }
    },
    methods: {
      getSkuDetails (row, expand) {
        // sku详情查询
      },
      // 处理更多指令
      handleCommand (row, command) {
        if (command === 'stockSettings') {
          // this.$router.push({path: "", query: {}})
        }
      },
      // 获取数据列表
      searchSkuInfo () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('commodity/skuinfo/list'),
          method: 'get',
          params: this.$http.adornParams(this.dataForm)
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.dataList = data.page.list
            this.totalPage = data.page.totalCount
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
      // 每页数
      sizeChangeHandle (val) {
        this.dataForm.limit = val
        this.dataForm.page = 1
        this.searchSkuInfo()
      },
      // 当前页
      currentChangeHandle (val) {
        this.dataForm.page = val
        this.searchSkuInfo()
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
    activated () {
    }
  }
</script>

<style scoped>
.el-dropdown-link {
  cursor: pointer;
  font-size: 12px;
  padding-left: 9px;
  color: #00b7ee;
}
</style>
