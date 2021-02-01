<template>
  <el-dialog :title="!dataForm.id ? '新增' : '修改'" :close-on-click-modal="false" :visible.sync="visible">
    <el-form :model="dataForm" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="110px">
    <el-form-item label="属性名" prop="attrName">
      <el-input v-model="dataForm.attrName" placeholder="属性名"></el-input>
    </el-form-item>
    <el-form-item label="属性图标" prop="icon">
      <el-input v-model="dataForm.icon" placeholder="属性图标"></el-input>
    </el-form-item>
    <el-form-item label="可选值" prop="valueSelect">
      <el-select v-model="dataForm.valueSelect" multiple filterable allow-create default-first-option
                 placeholder="请设置默认的选项值！" style="width: 100%;"></el-select>
    </el-form-item>
    <el-form-item label="所属分类" prop="catelogId">
      <CategorySelect ref="categorySelect" :categoryPath.sync="categoryPath"></CategorySelect>
    </el-form-item>
    <el-row>
      <el-col :span="12">
        <el-form-item label="属性类型" prop="attrType">
          <el-select v-model="dataForm.attrType" placeholder="请设置默认的选项值！" @change="showTag">
            <el-option label="规格属性" value="1" value-key="1"></el-option>
            <el-option label="销售属性" value="0" value-key="0"></el-option>
            <el-option label="混合属性" value="2" value-key="2"></el-option>
          </el-select>
        </el-form-item>
      </el-col>
      <el-col :span="12" v-if="!isSale">
        <el-form-item label="属性分组" prop="attrGroupId">
          <el-select v-model="dataForm.attrGroupId" placeholder="请设置当前属性的分组！">
            <el-option v-for="(item,i) in attrGroups" :key="i" :label="item.attrGroupName" :value="item.attrGroupId"></el-option>
          </el-select>
        </el-form-item>
      </el-col>
    </el-row>
      <el-row>
        <el-col :span="8" v-if="!isSale">
          <el-form-item label="参与检索" prop="searchType">
            <el-switch v-model="dataForm.searchType"
                       active-color="#13ce66" inactive-color="#ff4949" :inactive-value="'0'" :active-value="'1'"></el-switch>
          </el-form-item>
        </el-col>
        <el-col :span="8">
          <el-form-item label="启用状态" prop="enable">
            <el-switch v-model="dataForm.enable"
                       active-color="#13ce66" inactive-color="#ff4949" :inactive-value="'0'" :active-value="'1'"></el-switch>
          </el-form-item>
        </el-col>
        <el-col :span="8">
          <el-form-item label="快速展示" prop="showDesc">
            <el-switch v-model="dataForm.showDesc"
                       active-color="#13ce66" inactive-color="#ff4949" :inactive-value="'0'" :active-value="'1'"></el-switch>
          </el-form-item>
        </el-col>
      </el-row>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
import CategorySelect from './common/category-select'
export default {
  components: {CategorySelect},
  data () {
    return {
      isSale: false,
      visible: false,
      categoryPath: [],
      attrGroups: [],
      dataForm: {
        attrGroupId: '',
        attrId: 0,
        attrName: '',
        searchType: '1',
        icon: '',
        valueSelect: '',
        attrType: '',
        enable: '1',
        catelogId: '',
        showDesc: '1'
      }
    }
  },
  methods: {
    showTag () {
      if (this.dataForm.attrType == 0) {
        this.isSale = true
      } else {
        this.isSale = false
      }
    },
    init (id) {
      this.dataForm.attrId = id || 0
      this.visible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        if (this.dataForm.attrId) {
          this.$http({
            url: this.$http.adornUrl(`/commodity/attr/info/${this.dataForm.attrId}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.dataForm.attrName = data.attr.attrName
              this.dataForm.searchType = data.attr.searchType
              this.dataForm.icon = data.attr.icon
              this.dataForm.valueSelect = data.attr.valueSelect
              this.dataForm.attrType = data.attr.attrType
              this.dataForm.enable = data.attr.enable
              this.dataForm.catelogId = data.attr.catelogId
              this.dataForm.showDesc = data.attr.showDesc
            }
          })
        }
      })
    },
      // 表单提交
    dataFormSubmit () {
      this.$confirm(`是否【${!this.dataForm.attrId ? '保存' : '修改'}】当前数据?`, '温馨提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.dataForm.attrId = this.dataForm.attrId || undefined
        this.dataForm.valueSelect = this.dataForm.valueSelect.toString()
        this.dataForm.catelogId = this.categoryPath[2]
        this.$http({
          url: this.$http.adornUrl(`commodity/attr/${!this.dataForm.attrId ? 'save' : 'update'}`),
          method: 'post',
          data: this.$http.adornData(this.dataForm)
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.visible = false
                this.$emit('refreshDataList')
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      })
    }
  },
  watch: {
    categoryPath () {
      if (this.categoryPath.length === 3) {
        this.$http({
          url: this.$http.adornUrl('commodity/attrgroup/list'),
          method: 'GET',
          params: this.$http.adornParams({
            queryType: 'all',
            catId: this.categoryPath[2]
          })
        }).then(({data}) => {
          this.attrGroups = data.data
        })
      }
    }
  }
}
</script>
