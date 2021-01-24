<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-table :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
      <el-table-column
        prop="brandId"
        header-align="center"
        align="center"
        label="品牌id">
      </el-table-column>
      <el-table-column
        prop="name"
        header-align="center"
        align="center"
        label="品牌名">
      </el-table-column>
      <el-table-column prop="logo" header-align="center" align="center" label="品牌logo">
        <template slot-scope="scope">
          <img style="width: 80px; height: 80px" :src="scope.row.logo"  alt=""/>
        </template>
      </el-table-column>
      <el-table-column
        prop="descript"
        header-align="center"
        align="center"
        label="介绍">
      </el-table-column>
      <el-table-column prop="showStatus" header-align="center" align="center" label="显示状态">
        <template slot-scope="scope">
          <el-switch v-model="scope.row.showStatus" @change="updateShowStatus(scope.row)"
                     active-color="#13ce66" inactive-color="#ff4949" :inactive-value="0" :active-value="1"></el-switch>
        </template>
      </el-table-column>
      <el-table-column
        prop="firstLetter"
        header-align="center"
        align="center"
        label="检索首字母">
      </el-table-column>
      <el-table-column
        prop="sort"
        header-align="center"
        align="center"
        label="排序">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="brandCateRelation(scope.row.brandId,scope.row.name)">分类关联</el-button>
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.brandId)">修改</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.brandId)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
    <CategoryBrandRelation v-if="categoryBrandRelationVisible"
        ref="categoryBrandRelation" @refreshDataList="getDataList"></CategoryBrandRelation>
  </div>
</template>

<script>
  import AddOrUpdate from './brand-add-or-update'
  import CategoryBrandRelation from './categorybrandrelation.vue'
  export default {
    components: {
      AddOrUpdate, CategoryBrandRelation
    },
    data () {
      return {
        dataForm: {
          key: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        categoryBrandRelationVisible: false
      }
    },
    activated () {
      // this.getDataList()
    },
    methods: {
      updateShowStatus (data) {
        console.log('当前的状态数据：', data.showStatus)
        this.$confirm(`是否修改【${data.name}】的显示状态操作?`, '温馨提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('commodity/brand/update'),
            method: 'post',
            data: this.$http.adornData({
              brandId: data.brandId,
              showStatus: data.showStatus
            }, false)
          }).then(({data}) => {
            console.log(data)
            if (data && data.code === 0) {
              this.$message.success('操作成功')
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      },
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('commodity/brand/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'key': this.dataForm.key
          })
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
      brandCateRelation (brandId, brandName) {
        console.log(brandId, brandName)
        this.categoryBrandRelationVisible = true
        this.$nextTick(() => {
          this.$refs.categoryBrandRelation.brandId = brandId
          this.$refs.categoryBrandRelation.brandName = brandName
        })
      },
      // 每页数
      sizeChangeHandle (val) {
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
      // 当前页
      currentChangeHandle (val) {
        this.pageIndex = val
        this.getDataList()
      },
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },
      // 新增 / 修改
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          console.log(id)
          this.$refs.addOrUpdate.init(id)
        })
      },
      // 删除
      deleteHandle (id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.brandId
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/commodity/brand/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      }
    }
  }
</script>
