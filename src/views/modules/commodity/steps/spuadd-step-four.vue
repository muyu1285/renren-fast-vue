<!-- 相类数据表pms_product_attr_value pms_sku_info -->
<template>
  <el-card class="box-card" style="width:90%;margin:20px auto">
    <div slot="header" class="clearfix">
      <el-alert :title="'商品名称：' + viewData.spuName" type="warning" center :closable="false" /><br>
      <el-table :data="viewData.tableData" style="width: 100%" height="400">
        <el-table-column type="expand" >
          <template slot-scope="scope">
            <el-form :model="scope.row" label-width="110px">
              <el-form-item label="商品名称">
                <el-input readonly v-model="viewData.spuName" />
              </el-form-item>
              <el-form-item label="标题">
                <el-input v-model="scope.row.skuTitle" />
              </el-form-item>
              <el-form-item label="副标题">
                <el-input v-model="scope.row.skuSubtitle" />
              </el-form-item>
              <el-row>
                <el-col :span="24">
                  <el-form-item label="设置折扣">
                    <label>满</label>
                    <el-input-number style="width:160px" :min="0" v-model="scope.row.fullCount" />
                    <label>件</label>
                    <label style="margin-left:15px;">打</label>
                    <el-input-number style="width:160px" :precision="2" :max="1"
                          v-model="scope.row.discount" :min="0" :step="0.01" />
                    <label>折</label>
                    <el-checkbox :true-label="1" :false-label="0" v-model="scope.row.countStatus">可叠加优惠</el-checkbox>
                  </el-form-item>
                </el-col>
                <el-col :span="24">
                  <el-form-item label="设置满减">
                    <label>满</label>
                    <el-input-number style="width:160px" :step="100" :min="0" v-model="scope.row.fullPrice" />
                    <label>元</label>
                    <label style="margin-left:15px;">减</label>
                    <el-input-number style="width:160px" :step="10" :min="0" v-model="scope.row.reducePrice" />
                    <label>元</label>
                    <el-checkbox :true-label="1" :false-label="0" v-model="scope.row.priceStatus">可叠加优惠</el-checkbox>
                  </el-form-item>
                </el-col>
                <el-col :span="24">
                  <el-form-item label="设置会员价" v-if="scope.row.memberPrice.length > 0">
                    <el-row>
                      <el-col :span="12" v-for="(member,idx) in scope.row.memberPrice" :key="idx" style="margin-top: 10px">
                        <label>{{member.memberLevelName}}</label>
                        <el-input-number style="width:160px" :step="50" :min="0" v-model="scope.row.memberPrice[idx].memberPrice" />
                      </el-col>
                    </el-row>
                  </el-form-item>
                </el-col>
                <el-col :span="24">
                  <el-form-item label="商品预览图集" >
                    <el-card style="width:170px;float:left;margin-left:15px;margin-top:15px;" :body-style="{ padding: '0px' }"
                        v-for="(item, idx) in scope.row.productPreviews" :key="idx">
                      <img :src="item.imgUrl" style="width:160px;height:120px;object-fit:contain;" />
                      <div style="padding: 14px;">
                        <el-row>
                          <el-col :span="12">
                            <el-checkbox :name="viewData.spuName" v-model="item.selected" true-label="true" false-label="false"></el-checkbox>
                          </el-col>
                          <el-col :span="12">
                            <el-tag>
                              <input type="radio" :name="viewData.spuName" @change="setImgDefault(item, scope.row.productPreviews, $event)" />设为默认
                            </el-tag>
                          </el-col>
                        </el-row>
                      </div>
                    </el-card>
                  </el-form-item>
                </el-col>
              </el-row>
            </el-form>
          </template>
        </el-table-column>
        <el-table-column label="组 合 属 性" header-align="center">
          <el-table-column header-align="center" width="120px"
                           v-for="(item,idx) in viewData.saleAttrColumns" :label="item.attrName" >
            <template slot-scope="scope">
              <span style="margin-left: 10px">{{scope.row.attr[idx].attrValue}}</span>
            </template>
          </el-table-column>
        </el-table-column>
        <el-table-column header-align="center" prop="skuTitle" label="标 题" width="220px">
          <template slot-scope="scope">
            <el-input v-model="scope.row.skuTitle"></el-input>
          </template>
        </el-table-column>
        <el-table-column header-align="center" prop="skuTitle" label="价 格" width="120px">
          <template slot-scope="scope">
            <el-input v-model="scope.row.price"></el-input>
          </template>
        </el-table-column>
        <el-table-column label=""></el-table-column>
      </el-table>
    </div>
    <div align="center">
      <el-button type="primary" @click="$emit('nextStep',2)">上一步</el-button>
      <el-button type="success" @click="addSpuData()">数据保存</el-button>
    </div>
  </el-card>
</template>
<script>
// import com from ''

export default {
  components: {},
  inject: ['reload'],
  data () {
    return {
      viewData: {
        spuName: '',
        productPreviews: [],
        saleAttrColumns: [],
        tableData: [],
        memberPrice: []
      },
      stepOne: {},
      stepTwo: {},
      stepThree: {},
      saleAttrColumnDatas: [],
      SpuSaveDataVo: {}
    }
  },
  // 计算属性
  computed: {
  },
  // 监听器
  watch: {
  },
  methods: {
    initData (parentsVue) {
      console.log('parentsVue = ', parentsVue)
      let stepOne = parentsVue.$refs.stepOne
      this.stepOne = stepOne
      this.stepTwo = parentsVue.$refs.stepTwo
      let stepThree = parentsVue.$refs.stepThree
      this.stepThree = stepThree
      this.viewData.spuName = stepOne.spuData.spuName   // 商品名称
      this.viewData.productPreviews = stepOne.spuData.productPreviews   // 商品预览图
      // 组装数据
      let spuSaleDatas = stepThree.spuData.spuSaleData
      console.log('spuSaleDatas = ', spuSaleDatas)

      let saleAttrColumnValues = []
      this.viewData.saleAttrColumns = []
      spuSaleDatas.forEach(item => {
        if (item.attrValues.length > 0) {
          saleAttrColumnValues.push(item.attrValues)    // 获取当前销售属性对应的值
          this.viewData.saleAttrColumns.push(item) // 列名 --> 销售属性名
        }
      })
      // 获取迪卡尔乘积
      console.log('saleAttrColumnValues = ', saleAttrColumnValues)
      this.saleAttrColumnDatas = this.getCartesianProduct(saleAttrColumnValues)
      console.log('saleAttrColumnDatas = ', this.saleAttrColumnDatas)

      // 遍历每一个值，封装成列表中所需要的数据
      this.viewData.tableData = []
      this.saleAttrColumnDatas.forEach(arr => {
        let txt = ''    // 拼接属性标题
        let saleAttrValues = []
        if (Array.isArray(arr)) {
          arr.forEach((val, idx) => {
            txt += ' ' + val
            saleAttrValues.push({
              attrName: arr[idx],
              attrValue: val
            })
          })
        } else {
          saleAttrValues.push({
            attrName: arr,
            attrValue: arr
          })
        }
        this.viewData.tableData.push({
          attr: saleAttrValues,
          skuTitle: stepOne.spuData.spuName + txt,
          skuSubtitle: '',
          price: 0.0,
          fullPrice: 0,   // 满减额度
          reducePrice: 0, // 减额
          priceStatus: 0, // 满减是否叠加
          fullCount: 0,   // 满减数量
          discount: 0,    // 折扣
          countStatus: 0,  // 数量折扣是否叠加
          memberPrice: JSON.parse(JSON.stringify(this.viewData.memberPrice)),  // 复制对象
          productPreviews: JSON.parse(JSON.stringify(this.viewData.productPreviews))   // 复制对象
        })
      })
      console.log('viewData = ', this.viewData)
    },
    getCartesianProduct (values) {
      if (values.length < 2) return values[0] || []
      // reduce -->为数组中的所有元素调用指定的回调函数。回调函数的返回值是累计的结果，并在下一次调用回调函数时作为参数提供。
      // call() --> 调用对象的方法，用另一个对象替换当前对象。就是拿values的每一个元素进行遍历操作
      // [] --> 是一个空数组对象   frontArr--> 前一个数组  backArr --> 后一个数组
      return [].reduce.call(values, function (frontArr, backArr) {
        console.log('frontArr = ', frontArr)
        console.log('backArr = ', backArr)
        let result = []
        // 第1个数组遍历
        frontArr.forEach(function (item) {
          // 第二个数组遍历
          backArr.forEach(function (value) {
            // 判断arr是否为数组，如果是就合并到新数组中
            let t = [].concat(Array.isArray(item) ? item : [item])
            t.push(value)
            result.push(t)
          })
        })
        return result
      })
    },
    getMemberData () {
      this.$http({
        url: this.$http.adornUrl('member/memberlevel/list'),
        method: 'get',
        params: this.$http.adornParams({
          queryType: 'all'
        })
      }).then(({data}) => {
        let memberLevels = data.data
        if (memberLevels.length > 0) {
          memberLevels.forEach(m => {
            this.viewData.memberPrice.push({
              'memberLevelId': m.id,
              'memberLevelName': m.name,
              'addOther': 0,
              'memberPrice': 0.0
            })
          })
        }
      })
    },
    setImgDefault (imgObj, productPreviews, el) {
      // 遍历把默认都设置成false
      productPreviews.forEach(item => {
        item.default = 'false'
      })
      // 把当前图片设置成默认
      imgObj.default = 'true'
      imgObj.selected = 'true'
    },
    addSpuData () {   // 保存数据
      // console.log("stepOne", this.stepOne)
      // console.log("stepTwo", this.stepTwo)
      // console.log("stepThree", this.stepThree)
      // 封装SpuSaveDataVo数据
      this.SpuSaveDataVo.brandId = this.stepOne.brandId      // 品牌ID
      this.SpuSaveDataVo.catalogId = this.stepOne.categoryPath[2]    // 分类ID
      this.SpuSaveDataVo.spuName = this.viewData.spuName    // 商品名称
      this.SpuSaveDataVo.weight = this.stepOne.spuData.weight    // 商品重量
      this.SpuSaveDataVo.spuDescription = this.stepOne.spuData.spuDescription    // 商品描述
      this.SpuSaveDataVo.bounds = {   // 积分
        'buyBounds': this.stepOne.spuData.buyBounds,
        'growBounds': this.stepOne.spuData.growBounds
      }
      this.SpuSaveDataVo.goodsImgs = this.stepOne.spuData.goodsImgs    // 详细图片
      this.SpuSaveDataVo.productPreviews = this.stepOne.spuData.productPreviews    // 预览图片
      this.SpuSaveDataVo.baseAttrs = this.getBaseAttrs()    // 基本属性
      // this.SpuSaveDataVo.decript = 0
      this.SpuSaveDataVo.skus = this.getSkuInfoData()    // SKU信息
      console.log('SpuSaveDataVo = ', this.SpuSaveDataVo)

      this.$confirm('是否保存当前商品信息？', '温馨提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('commodity/spuinfo/save'),
          method: 'post',
          data: this.$http.adornData(this.SpuSaveDataVo)
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message.success('操作成功')
            setTimeout(() => {
              // location.reload()
              this.reload()
            }, 1500)
          } else {
            this.$message.error(data.msg)
          }
        })
      })
    },
    getBaseAttrs () {
      let datas = this.stepTwo.spuData.spuBasicData
      let arr = []
      datas.forEach(item => {
        item.forEach(attr => {
          if (attr.attrValue.length > 0) {  // 去除空数据，没有数据不需要保存
            arr.push({
              'attrId': attr.attrId,
              'attrName': attr.attrName,
              'attrValue': attr.attrValue.join(';'),   // 把数组中的元素转换成字符串，使用;分隔
              'quickShow': attr.quickShow
            })
          }
        })
      })
      return arr
    },
    getSkuInfoData () {
      let skus = []
      this.saleAttrColumnDatas.forEach((columnDatas, i) => {
        let attrArray = []
        columnDatas.forEach((item, idx) => {
          attrArray.push({
            attrId: this.viewData.saleAttrColumns[idx].attrId,
            attrName: this.viewData.saleAttrColumns[idx].attrName,
            attrValue: item
          })
        })
        skus.push({
          attrs: attrArray,
          skuTitle: this.viewData.tableData[i].skuTitle,
          skuName: this.viewData.tableData[i].skuTitle,
          skuSubtitle: this.viewData.tableData[i].skuSubtitle,
          price: this.viewData.tableData[i].price,
          fullPrice: this.viewData.tableData[i].fullPrice,
          reducePrice: this.viewData.tableData[i].reducePrice,
          priceStatus: this.viewData.tableData[i].priceStatus,
          fullCount: this.viewData.tableData[i].fullCount,
          discount: this.viewData.tableData[i].discount,
          countStatus: this.viewData.tableData[i].countStatus,
          memberPrices: this.viewData.tableData[i].memberPrice,
          images: this.handlerImages(this.viewData.tableData[i].productPreviews)
        })
      })
      return skus
    },
    handlerImages (imgList) {
      let imgs = []
      imgList.forEach((item, index) => {
        if (item.selected == 'true') {
          imgs.push({
            defaultImg: item.default == 'true' ? 1 : 0,
            imgName: item.imgName,
            imgUrl: item.imgUrl
          })
        }
      })
      return imgs
    }
  },
  // 创建完成 （可以访问当前this）
  created () {
    // 获取会员数据
    this.getMemberData()
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
  .demo-table-expand {
    font-size: 0;
  }
  .demo-table-expand label {
    width: 90px;
    color: #99a9bf;
  }
  .demo-table-expand .el-form-item {
    margin-right: 0;
    margin-bottom: 0;
    width: 50%;
  }
</style>
