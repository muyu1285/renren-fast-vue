<!-- 相关数据表pms_sku_sale_attr_value -->
<template>
  <div>
    <el-card class="box-card" style="width:90%;margin:20px auto">
      <div slot="header" class="clearfix">
        <el-alert title="选择销售属性" type="success" center :closable="false"></el-alert><br>
        <el-form ref="saleform" :model="spuData" label-width="100px" size="mini">
          <el-form-item :label="attr.attrName + '：'" v-for="(attr,index) in saleAttrDatas" :key="attr.attrId">
            <el-input v-model="spuData.spuSaleData[index].attrId" v-show="false"></el-input>
            <el-checkbox-group v-model="spuData.spuSaleData[index].attrValues">
              <el-checkbox
                  v-if="attr.valueSelect != ''"
                  v-for="(val,i) in attr.valueSelect.split(',')" :key="val + i"
                  :label="val"></el-checkbox>
              <div style="margin-left:20px;display:inline">
                <el-button size="mini" round icon="el-icon-plus" @click="addSaleValue(index)">自定义</el-button>
              </div>
            </el-checkbox-group>
          </el-form-item>
        </el-form>
      </div>
      <div align="center">
        <el-button type="primary" @click="$emit('nextStep',1)">上一步</el-button>
        <el-button type="success" @click="$emit('nextStep',3)">下一步</el-button>
      </div>
    </el-card>
  </div>
</template>
<script>
// import com from ''

  export default {
    components: {},
    data () {
      return {
        saleAttrDatas: [],
        spuData: {
          spuSaleData: []
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
      getAttrData (catId) {
        if (catId) {
          this.$http({
            url: this.$http.adornUrl('commodity/attr/list'),
            method: 'get',
            params: this.$http.adornParams({
              'catId': catId,
              'queryType': 'all',
              'attrType': 0
            })
          }).then(({data}) => {
            let queryData = data.data
            // console.log(queryData)
            this.saleAttrDatas = queryData
            // 销售属性数组初始化
            this.spuData.spuSaleData = []   // 清空，重新获取数制
            queryData.forEach(item => {
              this.spuData.spuSaleData.push({
                attrId: item.attrId,
                attrValues: [],
                attrName: item.attrName
              })
            })
          })
        }
      },
      addSaleValue (index) {
        // console.log("saleAttrDatas = ", index, this.spuSaleData[index])
        this.$prompt('请输入【' + this.saleAttrDatas[index].attrName + '】销售属性的选项数据：', '温馨提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消'
        }).then(({ value }) => {
          if (value.trim().length === 0) {
            this.$message.error('当前值为空，请重新输入销售属性选项数据！')
          } else {
            if (this.saleAttrDatas[index].valueSelect === '') {
              this.saleAttrDatas[index].valueSelect = value
            } else {
              this.saleAttrDatas[index].valueSelect += ',' + value
            }
          }
        }).catch(() => {})
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
