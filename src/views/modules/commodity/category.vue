<!--  -->
<template>
  <div>
    <el-tree :data="menus" :props="defaultProps" accordion :expand-on-click-node="false" node-key="catId">
      <span class="custom-tree-node" slot-scope="{ node, data }">
          <span>{{ node.label }}</span>
          <span>
            <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)"> 新增 </el-button>
            <el-button v-if="node.childNodes.length == 0" type="text" size="mini" @click="() => remove(node, data)"> 删除 </el-button>
          </span>
        </span>
    </el-tree>
    <el-dialog title="温馨提示" :visible.sync="dialogVisible" width="40%">
      <el-form :model="category" >
        <el-form-item label="上级菜单：">
          <el-input readonly :value="categoryParantName"></el-input>
        </el-form-item>
        <el-form-item label="子菜单名称：">
          <el-input v-model="category.name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveCategory">确 定</el-button>
      </span>
    </el-dialog>
  </div>

</template>

<script>
// import com from ''
export default {
  components: {},
  data () {
    return {
      nodeData: {},
      category: {id: '', name: '', parentCid: 1, catLevel: 1, showStatus: 1, sort: 0},
      categoryParantName: '',
      menus: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      },
      dialogVisible: false
    }
  },
  // 计算属性
  computed: {
  },
  // 监听器
  watch: {
  },
  methods: {
    append (data) {
      this.categoryParantName = data.name
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1
      this.dialogVisible = true
      this.nodeData = data
    },
    remove (node, data) {
      console.log(node)
      console.log(data)
    },
    saveCategory () {
      // console.log('要添加的菜单数据：', this.category)
      this.$confirm('是否保存当前子菜单数据？', '温馨提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('commodity/category/save'),
          method: 'post',
          data: this.$http.adornData(this.category, false)
        }).then(({data}) => {
          this.category.id = data.data.catId
          this.$message({
            message: data.msg,
            type: 'success'
          })
          this.dialogVisible = false
          this.nodeData.children.push(data.data)
          this.category = {id: '', name: '', parentCid: 1, catLevel: 1, showStatus: 1, sort: 0}
        })
      })
    }
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
      this.menus = data.data
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
