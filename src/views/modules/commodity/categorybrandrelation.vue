<template>
  <div class="mod-config">
    <el-dialog title="品牌与分类的关联设置" :close-on-click-modal="false"
               :visible.sync="visible" @close="$parent.categoryBrandRelationVisible = false">
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
      <el-table
        :data="dataList"
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
          prop="id"
          header-align="center"
          align="center"
          label="ID">
        </el-table-column>
        <el-table-column
          prop="brandId"
          header-align="center"
          align="center"
          label="品牌id">
        </el-table-column>
        <el-table-column
          prop="catelogId"
          header-align="center"
          align="center"
          label="分类id">
        </el-table-column>
        <el-table-column
          prop="brandName"
          header-align="center"
          align="center"
          label="品牌名称">
        </el-table-column>
        <el-table-column
          prop="catelogName"
          header-align="center"
          align="center"
          label="分类名称">
        </el-table-column>
        <el-table-column
          fixed="right"
          header-align="center"
          align="center"
          width="150"
          label="操作">
          <template slot-scope="scope">
            <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
            <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
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
    </el-dialog>
    <!-- 弹窗 -->
    <el-dialog title="添加关联" :close-on-click-modal="false" width="30%" :visible.sync="addRelationVisible">
      <div align="center">
        <CategorySelect ref="categorySelect" :categoryPath.sync="categoryPath"></CategorySelect>
        <br><br><el-button type="primary" @click="addRelation">保存</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
  import AddOrUpdate from './categorybrandrelation-add-or-update'
  import CategorySelect from './common/category-select'
  export default {
    data () {
      return {
        brandId: 0,
        brandName: '',
        categoryPath: [],
        visible: true,
        addRelationVisible: false,
        dataForm: {
          key: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false
      }
    },
    components: {
      AddOrUpdate, CategorySelect
    },
    activated () {
      // this.getDataList()
    },
    methods: {
      addRelation () {
        console.log(this.$refs.categorySelect.$refs.category.$el.innerText)
        let categoryNames = this.$refs.categorySelect.$refs.category.$el.innerText.split(' => ')
        console.log(categoryNames)
        this.$http({
          url: this.$http.adornUrl('commodity/categorybrandrelation/save'),
          method: 'post',
          data: this.$http.adornData({
            brandId: this.brandId,
            brandName: this.brandName,
            catelogId: this.categoryPath[2],
            catelogName: categoryNames[2]
          })
        }).then(({data}) => {
          this.$message.success('关联数据添加完成！')
          this.getDataList(this.brandId)
          this.addRelationVisible = false
        })
      },
      // 获取数据列表
      getDataList (brandId) {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('commodity/categorybrandrelation/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'key': this.dataForm.key,
            brandId: brandId
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
        this.addRelationVisible = true
      },
      // 删除
      deleteHandle (id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('commodity/categorybrandrelation/delete'),
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
