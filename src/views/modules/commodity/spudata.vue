<!--  -->
<template>
  <div>
    <el-row>
      <el-col :span="24">
        <el-form :inline="true" :model="dataForm">
          <el-form-item label="分类：">
            <CategorySelect :categoryPath.sync="categoryPath"></CategorySelect>
          </el-form-item>
          <el-form-item label="品牌：">
            <BrandSelect :brandId.sync="dataForm.brandId" ref="brandSelect"></BrandSelect>
          </el-form-item>
          <el-form-item label="状态：">
            <el-tooltip class="item" effect="light" content="新增的SPU状态为【下架】" placement="top">
              <el-select style="width:160px" v-model="dataForm.status" clearable>
                <el-option label="上架" :value="1"></el-option>
                <el-option label="下架" :value="0"></el-option>
              </el-select>
            </el-tooltip>
          </el-form-item>
          <el-form-item label="搜索关键字：">
            <el-input style="width:160px" v-model="dataForm.keyword" clearable></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="searchSpuData">查询</el-button>
          </el-form-item>
        </el-form>
      </el-col>
      <el-col :span="24">
        <div class="mod-config">
          <el-table :data="dataList" border v-loading="dataListLoading" style="width: 100%;">
            <el-table-column type="index" header-align="center" align="center" width="50" label="序号"></el-table-column>
            <el-table-column prop="id" header-align="center" align="center" label="id" width="60" ></el-table-column>
            <el-table-column prop="spuName" header-align="center" label="名称" :show-overflow-tooltip="true"></el-table-column>
            <el-table-column prop="spuDescription" header-align="center" label="描述" :show-overflow-tooltip="true"></el-table-column>
            <el-table-column prop="weight" header-align="center" align="center" label="重量" width="100" ></el-table-column>
            <el-table-column prop="publishStatus" header-align="center" align="center" label="上架状态" width="120" >
              <template slot-scope="scope">
                <el-tag v-if="scope.row.publishStatus == '0'">已下架</el-tag>
                <el-tag v-if="scope.row.publishStatus == '1'">已上架</el-tag>
              </template>
            </el-table-column>
            <el-table-column prop="createTime" header-align="center" align="center" label="创建时间"></el-table-column>
            <el-table-column prop="updateTime" header-align="center" align="center" label="修改时间"></el-table-column>
            <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
              <template slot-scope="scope">
                <el-button type="primary" size="mini" round @click="putawayOrSoldout(true, scope.row.id)" v-if="scope.row.publishStatus == '0'">上架</el-button>
                <el-button type="warning" size="mini" round @click="putawayOrSoldout(false, scope.row.id)" v-if="scope.row.publishStatus == '1'">下架</el-button>
              </template>
            </el-table-column>
          </el-table>
          <el-pagination @size-change="sizeChangeHandle" @current-change="currentChangeHandle" :current-page="dataForm.page"
                         :page-sizes="[1, 5, 10, 20]" :page-size="dataForm.limit" :total="totalPage" layout="total, sizes, prev, pager, next, jumper"></el-pagination>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
  import CategorySelect from '@/views/modules/commodity/common/category-select'
  import BrandSelect from '@/views/modules/commodity/common/brand-select.vue'
  export default {
    components: {CategorySelect, BrandSelect},
    data () {
      return {
        categoryPath: [],
        dataForm: {
          status: '',
          keyword: '',
          brandId: '',
          catelogId: '',
          page: 1,
          limit: 5
        },
        dataList: [],
        totalPage: 0,
        dataListLoading: false
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
      searchSpuData () {
        this.$http({
          url: this.$http.adornUrl('commodity/spuinfo/list'),
          method: 'get',
          params: this.$http.adornParams(this.dataForm)
        }).then(({data}) => {
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
      putawayOrSoldout (flag, id) {
        this.$confirm(`是否进行商品【上架】操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          // 上架
          if (flag) {
            this.$http({
              url: this.$http.adornUrl(`commodity/spuinfo/${id}/spup`),
              method: 'post'
            }).then(({ data }) => {
              if (data && data.code === 0) {
                this.$message.success('操作成功')
                this.searchSpuData()
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      },
      // 每页数
      sizeChangeHandle (val) {
        this.dataForm.limit = val
        this.dataForm.page = 1
        this.searchSpuData()
      },
      // 当前页
      currentChangeHandle (val) {
        this.dataForm.page = val
        this.searchSpuData()
      },
      // 新增 / 修改
      addOrUpdateHandle (id) {}
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
