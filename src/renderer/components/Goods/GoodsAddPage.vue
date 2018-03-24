<template>
  <div class="content-page">
    <div class="content-nav">
      <el-breadcrumb class="breadcrumb" separator="/">
        <el-breadcrumb-item :to="{ name: 'dashboard' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>{{infoForm.id ? '编辑商品' : '添加商品'}}</el-breadcrumb-item>
      </el-breadcrumb>
      <div class="operation-nav">
        <el-button type="primary" @click="goBackPage" icon="arrow-left">返回列表</el-button>
      </div>
    </div>
    <div class="content-main">
      <div class="form-table-box">
        <el-form ref="infoForm" :rules="infoRules" :model="infoForm" label-width="120px">
          <el-form-item label="所属分类" >
            <el-select v-model="infoForm.category_id" placeholder="请选择产品分类">
              <el-option v-for="item in goodsCategory" :key="item.id" :label="item.name" :value="item.id"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="商品名称" prop="name">
            <el-input v-model="infoForm.name"></el-input>
          </el-form-item>
          <el-form-item label="所属品牌">
            <el-select v-model="infoForm.brand_id" placeholder="请选择所属品牌">
              <el-option v-for="item in brandList" :key="item.id" :label="item.name" :value="item.id"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="商品简介" prop="goods_brief">
            <el-input type="textarea" v-model="infoForm.goods_brief" :rows="3"></el-input>
            <div class="form-tip"></div>
          </el-form-item>
          <el-form-item label="详情图片">
            <el-upload class="image-uploader" label="详情图" name="primary_pic_url"
                       action="https://www.wanders.com.cn/admin/upload/goodsPrimaryPic" :show-file-list="true"
                       :on-success="handleUploadImageSuccess" :headers="uploaderHeader">
              <img v-if="infoForm.primary_pic_url" :src="infoForm.primary_pic_url" class="image-show">
              <i v-else class="el-icon-plus image-uploader-icon"></i>
            </el-upload>
            <div class="form-tip">图片尺寸：750*420</div>
          </el-form-item>
          <el-form-item label="列表图片">
            <el-upload class="image-uploader" label="列表图" name="list_pic_url"
                       action="https://www.wanders.com.cn/admin/upload/goodsListPic" :show-file-list="true"
                       :on-success="handleUploadImageSuccess" :headers="uploaderHeader">
              <img v-if="infoForm.list_pic_url" :src="infoForm.list_pic_url" class="image-show">
              <i v-else class="el-icon-plus image-uploader-icon"></i>
            </el-upload>
            <div class="form-tip">图片尺寸：750*420</div>
          </el-form-item>
          <el-form-item label="推荐类型">
            <el-checkbox label="新品" v-model="infoForm.is_new"></el-checkbox>
            <el-checkbox label="人气" v-model="infoForm.is_hot"></el-checkbox>
          </el-form-item>
          <el-form-item label="上架">
                <el-switch v-model="infoForm.is_on_sale"></el-switch>
          </el-form-item>
          <el-form-item label="零售价">
            <el-input-number placeholder="请输入正整数" :min="1" :max="2000" v-model="infoForm.retail_price"></el-input-number>
            <span>元</span>
          </el-form-item>
          <el-form-item label="库存数">
            <el-input-number placeholder="请输入正整数" :min="1" :max="2000" v-model="infoForm.goods_number"></el-input-number>
            <span>件</span>
          </el-form-item>
          <el-form-item label="商品排序">
            <el-input-number v-model="infoForm.sort_order" :min="1" :max="1000"></el-input-number>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="onSubmitInfo">确定保存</el-button>
            <el-button @click="goBackPage">取消</el-button>
          </el-form-item>
        </el-form>
      </div>
    </div>
  </div>
</template>

<script>
  import api from '@/config/api';
  export default {
    data() {
      return {
        uploaderHeader: {
          'X-Nideshop-Token': localStorage.getItem('token') || '',
        },
        goodsCategory: [],
        brandList: [],
        infoForm: {
          id: 0,
          name: "",
          category_id: '',
          brand_id: '',
          goods_brief: '', //简介
          sort_order: 10,  //排序
          retail_price: 0,  //零售价
          counter_price: 0,  //专柜价
          list_pic_url: '', //列表图
          primary_pic_url: '', //主图
          goods_number: 0, //商品数量
          sell_volume: 0, //销售量
          is_on_sale: true, //是否上架
          is_new: false, //是否新品
          is_hot: false //是否热卖人气品
        },
        infoRules: {
          name: [
            { required: true, message: '请输入名称', trigger: 'blur' },
          ],
          category_id: [
            { required: true, message: '请选择产品分类', trigger: 'blur' },
          ],
          brand_id: [
            { required: true, message: '请选择所属品牌', trigger: 'blur' },
          ],
          goods_brief: [
            { required: true, message: '请输入简介', trigger: 'blur' },
          ],
          pic_url: [
            { required: true, message: '请选择商品图片', trigger: 'blur' },
          ],
          inventory: [
            { required: true, message: '库存数不能为空', trigger: 'blur' },
          ],
          retailprice: [
            { required: true, message: '零售价不能为空', trigger: 'blur' },
          ],
        },
      }
    },
    methods: {
      goBackPage() {
        this.$router.go(-1);
      },
      onSubmitInfo() {
        this.$refs['infoForm'].validate((valid) => {
          if (valid) {
            this.axios.post('goods/store', this.infoForm).then((response) => {
              if (response.data.errno === 0) {
                this.$message({
                  type: 'success',
                  message: '保存成功'
                });
                this.$router.go(-1)
              } else {
                this.$message({
                  type: 'error',
                  message: '保存失败'
                })
              }
            })
          } else {
            return false;
          }
        });
      },
      handleUploadImageSuccess(res, file) {
        if (res.errno === 0) {
          switch (res.data.name) {
            //商品图片
            case 'list_pic_url':
              this.infoForm.list_pic_url = res.data.fileUrl;
              break;
            case 'primary_pic_url':
              this.infoForm.primary_pic_url = res.data.fileUrl;
              break;
          }
        }
      },
      getInfo() {
        if (this.infoForm.id <= 0) {
          return false
        }

        //加载商品详情
        let that = this
        this.axios.get('goods/info', {
          params: {
            id: that.infoForm.id
          }
        }).then((response) => {
          let resInfo = response.data.data;
          that.infoForm = resInfo;
          this.infoForm.is_new = resInfo.is_new === 1 ? true : false;
          this.infoForm.is_hot = resInfo.is_hot === 1 ? true : false;
          this.infoForm.is_on_sale = resInfo.is_on_sale === 1 ? true : false;
        })
      },

      getCategoryList() {
        this.axios.get('category', {
          params: {
            page: 1,
            name: ''
          }
        }).then((response) => {
          this.goodsCategory = response.data.data
          // console.log('========goodsCategory=======');
          // console.log(JSON.stringify(this.goodsCategory));
        })
      },

      getBrandList() {
        this.axios.get('brand', {
          params: {
            page: 1,
            name: ''
          }
        }).then((response) => {
          this.brandList = response.data.data.data;
          // console.log('========brandlist=======');
          // console.log(JSON.stringify(this.brandList));
        })
      }

    },
    components: {},
    mounted() {
      this.getCategoryList();
      this.getBrandList();
      this.infoForm.id = this.$route.query.id || 0;
      this.getInfo();
    }
  }

</script>

<style>
  .image-uploader{
    height: 105px;
  }
  .image-uploader .el-upload {
    border: 1px solid #d9d9d9;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }

  .image-uploader .el-upload:hover {
    border-color: #20a0ff;
  }

  .image-uploader .image-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 187px;
    height: 105px;
    line-height: 105px;
    text-align: center;
  }

  .image-uploader .image-show {
    width: 187px;
    height: 105px;
    display: block;
  }

  .image-uploader.new-image-uploader {
    font-size: 28px;
    color: #8c939d;
    width: 165px;
    height: 105px;
    line-height: 105px;
    text-align: center;
  }

  .image-uploader.new-image-uploader .image-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 165px;
    height: 105px;
    line-height: 105px;
    text-align: center;
  }

  .image-uploader.new-image-uploader .image-show {
    width: 165px;
    height: 105px;
    display: block;
  }
</style>
