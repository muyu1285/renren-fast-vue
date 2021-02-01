<!--  -->
<template>
  <el-card class="box-card" style="width:90%;margin:20px auto">
    <div slot="header" class="clearfix">
      <span v-if="Object.keys(attrGroupDatas).length === 0">无规格参数可用！</span>
      <el-tabs v-else>
        <el-tab-pane :label="attrGroup.attrGroupName" v-for="(attrGroup,gi) in attrGroupDatas" :key="attrGroup.attrGroupId">
          <el-form ref="form" :model="spuData" :label-position="labelPosition" label-width="110px" >
            <el-form-item :label="attr.attrName" v-for="(attr,ai) in attrGroup.attrs" :key="attr.attrId">
              <el-input :v-model="spuData.spuBasicData[gi][ai].attrId" hidden v-show="false" :value="attr.attrId"></el-input>
              <el-select v-model="spuData.spuBasicData[gi][ai].attrValue" filterable allow-create multiple
                         style="width: 50%"     default-first-option placeholder="请选择或输入值">
                <el-option v-if="attr.valueSelect.split(',').length > 0" v-for="(val,vidx) in attr.valueSelect.split(',')" :key="vidx" :label="val" :value="val"></el-option>
              </el-select>
              <el-checkbox v-model="spuData.spuBasicData[gi][ai].quickShow" :true-label="1" :false-label="0">快速展示</el-checkbox>
            </el-form-item>
          </el-form>
        </el-tab-pane>
      </el-tabs>
    </div>

    <div align="center">
      <el-button type="primary" @click="$emit('nextStep',0)">上一步</el-button>
      <el-button type="success" @click="$emit('nextStep',2)">下一步</el-button>
    </div>
  </el-card>
</template>


<script>
// import com from ''
export default {
  components: {},
  data () {
    return {
      labelPosition: 'right',
      spuData: {
        spuBasicData: []
      },
      activeName: 'first',
      attrGroupDatas: []
    }
  },
  // 计算属性
  computed: {},
  // 监听器
  watch: {},
  methods: {
    getAttrData (catId) {
      if (catId) {
        this.$http({
          url: this.$http.adornUrl('commodity/attrgroup/agandattr'),
          method: 'get',
          params: this.$http.adornParams({
            'catId': catId
          })
        }).then(({data}) => {
          let sources = data.data
          console.log('sources', sources)
          if (sources != null) {
            this.attrGroupDatas = sources
            // 创建数组，对spuBasicData进行二维数组初始化
            for (let attrGroups of sources) {
              let arr = []
              for (let attr of attrGroups.attrs) {
                arr.push({attrId: attr.attrId, attrName: attr.attrName, attrValue: '', quickShow: 0})
              }
              if (arr.length > 0) {
                this.spuData.spuBasicData.push(arr)
              }
            }
          }
        })
      }
    }

  },
  // 创建完成 （可以访问当前this）
  created () {
  },
  // 挂载完成（可以访问DOM）
  mounted () {
  },
  // 创建之前
  beforeCreate () {
  },
  // 挂载之前
  beforeMount () {
  },
  // 更新之前
  beforeUpdate () {
  },
  // 更新之后
  updated () {
  },
  // 销毁之前
  beforeDestroy () {
  },
  // 销毁完成
  destroyed () {
  },
  // 页面有keep-alive缓存，触发函数
  activated () {
  }
}
</script>

<style scoped>
</style>
