<!-- 基本信息步骤条分片 相关数据表pms_spu_info-->
<template>
  <el-card class="box-card" style="width:90%;margin:20px auto">
    <div slot="header" class="clearfix">
      <el-form ref="spuBaseForm" :model="spuData" label-width="100px" :rules="dataRule">
        <el-form-item label="商品名称" prop="spuName">
          <el-input v-model="spuData.spuName" clearable></el-input>
        </el-form-item>
        <el-form-item label="商品描述" prop="spuDescription">
          <el-input type="textarea" v-model="spuData.spuDescription" resize="none"></el-input>
        </el-form-item>
        <el-row>
          <el-col :span="12">
            <el-form-item label="选择分类" prop="catalogId">
              <CategorySelect :categoryPath.sync="categoryPath"></CategorySelect>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="选择品牌" prop="brandId">
              <BrandSelect ref="brandSelect" :brandId.sync="brandId"></BrandSelect>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="8">
            <el-form-item label="商品重量(Kg)" prop="weight">
              <el-input-number v-model="spuData.weight" :precision="2" :step="0.5"></el-input-number>
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item label="设置积分" prop="buyBounds">
              <el-input-number v-model="spuData.buyBounds"></el-input-number>
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item label="设置经验" prop="growBounds">
              <el-input-number v-model="spuData.growBounds"></el-input-number>
            </el-form-item>
          </el-col>
        </el-row>
        <el-form-item label="商品图片" prop="goodsImgs">
          <el-upload class="upload-demo" multiple ref="goodsImgsUpload"
                     action="http://0720xydmall.oss-cn-shenzhen.aliyuncs.com"
                     :data="uploadData" :before-upload="beforeAvatarUpload"
                     :on-success="handleUploadSuccess" :auto-upload="false">
            <el-button slot="trigger" size="small" type="primary">选取文件</el-button>
            <el-button style="margin-left: 10px;" size="small"
                       type="success" @click="$refs.goodsImgsUpload.submit()">开始上传
            </el-button>&nbsp;
            <span slot="tip" class="el-upload__tip">可以同时选择多张图片，但只能上传jpg/png文件，且不超过100kb</span>
          </el-upload>
        </el-form-item>
        <el-form-item label="商品预览图" prop="productPreviews">
          <el-upload class="upload-demo" multiple ref="ProductPreviewsUpload"
                     action="http://0720xydmall.oss-cn-shenzhen.aliyuncs.com"
                     :data="uploadData" :before-upload="beforeAvatarUpload"
                     :on-success="handleUploadSuccess" :auto-upload="false">
            <el-button slot="trigger" size="mini" type="primary">选取文件</el-button>
            <el-button style="margin-left: 10px;" size="mini"
                       type="success" @click="$refs.ProductPreviewsUpload.submit()">开始上传
            </el-button> &nbsp;
            <span slot="tip" class="el-upload__tip">可以同时选择多张图片，但只能上传jpg/png文件，且不超过100kb</span>
          </el-upload>
        </el-form-item>
      </el-form>
    </div>
    <div align="center">
      <el-button type="primary" round @click="nextStep(1)" >下一步</el-button>
    </div>
  </el-card>
</template>


<script>
import CategorySelect from '@/views/modules/commodity/common/category-select'
import BrandSelect from '@/views/modules/commodity/common/brand-select.vue'
export default {
  components: {CategorySelect, BrandSelect},
  data () {
    return {
      brandId: 0,
      categoryPath: [],
      uploadData: {},
      spuData: {
        spuName: '华为 HUAWEI Mate 30E Pro 5G',
        spuDescription: '华为 HUAWEI Mate 30E Pro 5G麒麟990E SoC芯片 双4000万徕卡电影影像 8GB+256GB亮黑色全网通手机',
        decript: '',
        weight: 0.18,
        bounds: 0,
        buyBounds: 30,
        growBounds: 30,
        goodsImgs: [],
        productPreviews: []
      },
      dataRule: {
        spuName: [
          { required: true, message: '商品名称不能为空', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    nextStep (number) {
      this.$refs['spuBaseForm'].validate((valid) => {
        if (valid) {
          // 获取商品图片数组
          let files = this.$refs.goodsImgsUpload.uploadFiles
          this.spuData.goodsImgs = []
          this.encapsulateImagedData(this.spuData.goodsImgs, files)
          // 获取商品预览图片数组
          let files2 = this.$refs.ProductPreviewsUpload.uploadFiles
          this.spuData.productPreviews = []
          this.encapsulateImagedData(this.spuData.productPreviews, files2)
          console.log(this.spuData)
          this.$emit('nextStep', number)
        }
      })
    },
    encapsulateImagedData (array, files) {
      for (let file of files) {
        console.log('encapsulateImagedData-->', file)
        array.push({
          imgName: file.name,
          imgUrl: this.uploadData.host + '/' + file.raw.key,
          selected: false,
          default: false
        })
      }
    },
    beforeAvatarUpload (file) {
      const isJPG = file.type === 'image/jpeg' || file.type === 'image/png' || file.type === 'image/gif'
      const isLt2M = file.size / 1024 / 1024 < 2

      if (!isJPG || !isLt2M) {
        this.$message.error('上传品牌LOGO图片只能是 JPG PNG GIF 格式!，且大小不能超过2MB！')
        return false    // 返回false表示不再做上传请求
      }
      return this.getOssData(file)
    },
    handleUploadSuccess (response, file, fileList) {
      this.$message.success('图片上传完成！')
      console.log('file = ', file)
    },
    getOssData (file) {
      // 获取组件对象
      let _this = this
      return new Promise((resolve, reject) => {
        this.$http({
          url: this.$http.adornUrl('oss/policy'),
          method: 'post'
        }).then(({data}) => {
          let ossInfo = data.data
          // 签名是否有效，过期没
          if ((ossInfo.expire - new Date().getTime() / 1000) > 0) {
            ossInfo.key = ossInfo.dir + this.getFileName(file)
            file.key = ossInfo.key
            _this.uploadData = ossInfo
            delete _this.uploadData.__ob__
            resolve(true)
          } else {
            this.$message.error('oss签名已失效！')
          }
        }).catch(() => {
          this.$message.error('oss签名获取失败！')
          reject(false)
        })
      })
    },
    getFileName (file) {
      let str = 'qwertyuioplkjhgfdsazxcvbnmQWERTYUIOPLKJHGFDSAZXCVBNM0123456789'
      let res = ''
      for (var i = 0; i < 6; i++) {
        let random = Math.random()// 取0-1 随机小数 包含0  不包含1
        random = random * (str.length) // 随机
        random = Math.floor(random)// 向下取整  0-verification.length-1
        res += str[random]
      }
      let filename = file.name
      let suffix = filename.substring(filename.lastIndexOf('.'))
      return new Date().getTime() + res + suffix
    }
  },
  // 计算属性
  computed: {
  },
  // 监听器
  watch: {
    categoryPath () {
      this.$refs.brandSelect.getCatBrands(this.categoryPath[2])
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
