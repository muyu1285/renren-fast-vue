<template>
  <el-dialog
    :title="!dataForm.brandId ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="110px">
    <el-form-item label="品牌名" prop="name">
      <el-input v-model.trim="dataForm.name" placeholder="品牌名"></el-input>
    </el-form-item>
    <el-form-item label="品牌logo地址" prop="logo">
      <el-upload class="avatar-uploader" action="http://0720xydmall.oss-cn-shenzhen.aliyuncs.com"
        :data="uploadData" :show-file-list="false" :on-success="handleAvatarSuccess" :before-upload="beforeAvatarUpload">
        <img v-if="dataForm.logo" :src="dataForm.logo" class="avatar" alt="">
        <i v-else class="el-icon-plus avatar-uploader-icon"></i>
      </el-upload>
    </el-form-item>
    <el-form-item label="介绍" prop="descript">
      <el-input v-model.trim="dataForm.descript" placeholder="介绍"></el-input>
    </el-form-item>
    <el-form-item label="显示状态" prop="showStatus">
      <el-switch v-model="dataForm.showStatus" active-color="#13ce66" inactive-color="#ff4949" inactive-value="0" active-value="1"></el-switch>
    </el-form-item>
    <el-form-item label="检索首字母" prop="firstLetter">
      <el-input v-model="dataForm.firstLetter" placeholder="检索首字母"></el-input>
    </el-form-item>
    <el-form-item label="排序" prop="sort">
      <el-input v-model.number="dataForm.sort" type="number" placeholder="排序"></el-input>
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        imageUrl: '',
        uploadData: {},
        dataForm: {
          brandId: 0,
          name: '',
          logo: '',
          descript: '',
          showStatus: '1',
          firstLetter: '',
          sort: '0'
        },
        dataRule: {
          name: [
            { required: true, message: '品牌名不能为空', trigger: 'blur' }
          ],
          logo: [
            { required: true, message: '品牌logo地址不能为空', trigger: 'blur' }
          ],
          showStatus: [
            { validator: (rule, value, callback) => {
              if (!/^[01]$/.test(value)) {
                callback(new Error('设置品牌显示状态，0表示不显示，1表示显示'))
              } else {
                callback()
              }
            },
              trigger: 'blur' }
          ],
          firstLetter: [
            { required: true, message: '检索首字母不能为空', trigger: 'blur' }
          ],
          sort: [
            { required: true, message: '排序不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.brandId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.brandId) {
            this.$http({
              url: this.$http.adornUrl(`commodity/brand/info/${this.dataForm.brandId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.brand.name
                this.dataForm.logo = data.brand.logo
                this.dataForm.descript = data.brand.descript
                this.dataForm.showStatus = data.brand.showStatus + ''
                this.dataForm.firstLetter = data.brand.firstLetter
                this.dataForm.sort = data.brand.sort
              }
            })
          }
        })
      },
      handleAvatarSuccess (res, file) {
        this.$message.success('图片上传完成！')
        this.dataForm.logo = this.uploadData.host + '/' + this.uploadData.key
        this.imageUrl = this.uploadData.host + '/' + this.uploadData.key
      },
      beforeAvatarUpload (file) {
        // console.log(file)
        const isJPG = file.type === 'image/jpeg' || file.type === 'image/png' || file.type === 'image/gif'
        const isLt2M = file.size / 1024 / 1024 < 2

        if (!isJPG || !isLt2M) {
          this.$message.error('上传品牌LOGO图片只能是 JPG PNG GIF 格式!，且大小不能超过2MB！')
          return false    // 返回false表示不再做上传请求
        }
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
              // 这里的this 是请求对象，不是组件对象
              // this.uploadData = ossInfo
              _this.uploadData = ossInfo
              // console.log('删除前的对象：', _this.uploadData)
              delete _this.uploadData.__ob__
              // console.log('删除后的对象：', _this.uploadData)
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
      },
      // 表单提交
      dataFormSubmit () {
        // 获取表单的验证对象结果，如果是true就进行请求保存数据
        this.$refs['dataForm'].validate((valid) => {
          console.log('dataFormSubmit', valid)
          if (valid) {
            this.$confirm(`是否保存当前品牌信息？`, '温馨提示', {
              confirmButtonText: '确定',
              cancelButtonText: '取消',
              type: 'warning'
            }).then(() => {
              this.$http({
                url: this.$http.adornUrl(`commodity/brand/${!this.dataForm.brandId ? 'save' : 'update'}`),
                method: 'post',
                data: this.$http.adornData({
                  'brandId': this.dataForm.brandId || undefined,
                  'name': this.dataForm.name,
                  'logo': this.dataForm.logo,
                  'descript': this.dataForm.descript,
                  'showStatus': this.dataForm.showStatus,
                  'firstLetter': this.dataForm.firstLetter,
                  'sort': this.dataForm.sort
                })
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
                  console.log(data)
                  this.$message.error(data.msg)
                }
              })
            })
          }
        })
      }
    }
  }
</script>

<style>
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409EFF;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>
