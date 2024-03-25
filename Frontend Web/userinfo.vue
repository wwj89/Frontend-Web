
<template>
  <div class="wrap">
    <div class="header">
      <div class="logoIndex">
        <div class="imgDiv">
          <img src="../assets/images/logo.png" class="imgLogo"/>
        </div>
      </div>
      <div class="btn-box">
        <div class="home" @click="goIndex()">Home</div>
      </div>
    </div>
    <div class="content">
      <div class="nav">
        <div class="user-name">
          <img :src="userProfile"/>
          <div style="color: #00afff">{{userName}}</div>
        </div>
        <div class="colletion" @click="tab = 1">colletion</div>
        <div class="publish" @click="tab = 2">publish</div>
      </div>
      <div class="commodity-area" v-if="tab == 1">
        <div v-for="(item,index) in collectProductList" :key="item.id">
          <div class="commodity">
            <div class="commodity-box" @click="goDetail(item.id)">
              <img :src="item.urls[0]" alt="" class="commodity-image" />
              <div class="commodity-name">{{ item.name }}</div>
              <div class="commodity-price">{{ item.price }}</div>
            </div>
          </div>
        </div>
        <div class="paging">
          <el-pagination
            layout="prev, pager, next, total, jumper"
            :total="total"
            :page-size="pageSize"
            @current-change="handleCurrentChange"
          ></el-pagination>
        </div>
      </div>
      <div class="publish-area" v-if="tab == 2">
        <div style="width: 100%">
          <el-form
            :label-position="labelPosition"
            label-width="80px"
            :model="goods"
            :rules="rules"
          >
            <el-form-item label="picture">
              <!-- 图片 -->
              <el-upload
                ref="upload"
                action="#"
                list-type="picture-card"
                :on-preview="handlePictureCardPreview"
                :on-success="hanleSuccess"
                :on-remove="handleRemove"
                :on-exceed="handleExceed"
                :on-error="handleUploadError"
                :file-list="fileList"
                :before-upload="hanleBeforeUpload"
                :on-change="hanleChangeUpload"
                :headers="headers"
                :http-request="uploadPicture"
              >
                <i class="el-icon-plus"></i>
              </el-upload>
              <el-dialog :visible.sync="dialogVisible">
                <img width="100%" :src="dialogImageUrl" alt="" />
              </el-dialog>
            </el-form-item >
            <!-- name -->
            <el-form-item label="name" style="width: 30%" prop="name">
              <el-input v-model="goods.name"></el-input>
            </el-form-item>
            <!-- price -->
            <el-form-item label="price" style="width: 30%" prop="price">
              <el-input v-model="goods.price"></el-input>
            </el-form-item>
            <!-- type -->
            <el-form-item label="classify" style="width: 30%" prop="classify">
              <el-select v-model="goods.typeId" placeholder="Please select category">
                <div v-for="item in productTypeList" :key="item.id">
                  <el-option
                      :label="item.code"
                      :value="item.id"
                    v-if="item.code != 'All'"
                  ></el-option>
                </div>
              </el-select>
            </el-form-item>
            <!-- information -->
            <el-form-item label="information" style="width: 30%" prop="description">
              <el-input type="textarea" v-model="goods.description"></el-input>
            </el-form-item>
            <el-form-item style="width: 30%">
              <el-button type="primary" @click="submit()">提交</el-button>
            </el-form-item>
          </el-form>
        </div>
      </div>
    </div>
    <!--  底部  -->
    <div class="el-login-footer">
      <span>Copyright © 2018-2023 ruoyi.vip All Rights Reserved.</span>
    </div>
  </div>
</template>

<script>
import {
  uploadPicture,
  getProductType,
  productAdd,
  getCollectProductList
} from "@/api/project/product.js";
import { getInfo } from "@/api/login.js";
export default {
  props: {
    // 值
    value: [String, Object, Array],
    // 数量限制
    limit: {
      type: Number,
      default: 5,
    },
    // 大小限制(MB)
    fileSize: {
      type: Number,
      default: 5,
    },
    // 文件类型, 例如['png', 'jpg', 'jpeg']
    fileType: {
      type: Array,
      default: () => ["png", "jpg", "jpeg"],
    },
    // 是否显示提示
    isShowTip: {
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      tab: 1,
      labelPosition: "right",
      form: {},
      dialogImageUrl: "",
      dialogVisible: false,
      fileList: [],
      // headers: { "Content-Type": "multipart/form-data" }
      // url: process.env.VUE_APP_BASE_API + "/product/upload", //上传图片的地址
      headers: { "Content-Type": "multipart/form-data" },
      productTypeList: [], //商品分类列表
      urls: [],
      goods: {
        description: "",
        name: "",
        price: null,
        typeId: "",
        url: ""
      }, //商品详细信息
      collectProductList: [], //收藏列表
      pageSize: 18, //一页的数量
      total: null, //数据总量
      pageIndex: 1, //分页当前页数
      // 表单校验
      rules: {
        name: [
          { required: true, message: "Please enter the product name", trigger: "blur" }
        ],
        price: [
          { required: true, message: "Please enter the product price", trigger: "blur" },
          {
            pattern: /^\d+(\.\d{1,2})?$|^\d+$/,
            message: "Price format error",
            trigger: "blur"
          }
        ],
        classify: [
          { required: true, message: "Please select category", trigger: "blur" },
        ],
        description: [
          { required: true, message: "Please enter the product description", trigger: "blur" },
        ]
      },
      //登录人信息
      userName: "",
      userProfile: ""
    };
  },
  created() {
    this.getCollectProductList();
    this.getGoodsProductList();
    this.getUserInfo();
  },
  computed: {
    // 是否显示提示
    showTip() {
      return this.isShowTip && (this.fileType || this.fileSize);
    },
  },
  methods: {
    // 获取登陆人信息
    getUserInfo() {
      getInfo().then((res) => {
        // console.log(res.user.avatar)
        this.userName = res.user.nickName;
        this.userProfile = process.env.VUE_APP_BASE_API + res.user.avatar;
      });
    },
    // 文件个数超出
    handleExceed() {
      this.$modal.msgError(`The number of uploaded files cannot exceed ${this.limit}!`);
    },
    // 上传失败
    handleUploadError(err) {
      this.$modal.msgError("Failed to upload the file. Please try again");
      this.$modal.closeLoading();
    },
    handleRemove(file, fileList) {
      // console.log(file, fileList);
    },
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url;
      this.dialogVisible = true;
    },
    // 图片上传前
    hanleBeforeUpload(file) {
      // 校检文件类型
      if (this.fileType) {
        const fileName = file.name.split('.');
        const fileExt = fileName[fileName.length - 1];
        const isTypeOk = this.fileType.indexOf(fileExt) >= 0;
        if (!isTypeOk) {
          this.$modal.msgError(`File format is not correct, please upload ${this.fileType.join("/")} format file!`);
          return false;
        }
      }
      // 校检文件大小
      if (this.fileSize) {
        const isLt = file.size / 1024 / 1024 < this.fileSize;
        if (!isLt) {
          this.$modal.msgError(`The size of the uploaded file cannot exceed ${this.fileSize} MB!`);
          return false;
        }
      }
      this.$modal.loading("uploading...");
      return true;
    },
    //上传图片
    uploadPicture: function (file) {
      const formData = new FormData();
      formData.append("file", file.file)
      uploadPicture(formData).then((res) => {
        this.urls.push(res.urls);
      });
      this.$modal.closeLoading();
    },
    hanleSuccess(file) {

    },
    //图片状态改变
    hanleChangeUpload(file) {
      this.fileList.push(file);
    },
    //提交商品新增
    submit() {
      if(this.fileList.length == 0){
        this.$modal.msgWarning("Please upload pictures first");
        return;
      }
      this.goods.url = this.urls.join(", ");
      productAdd(this.goods).then((res) => {
        if (res.code === 200) {
          this.$modal.msgSuccess("New product success");
        }else{
          this.$modal.msgError("New product fail");
        }
      });
      this.fileList = [];
      this.goods = {
        description: "",
        name: "",
        price: null,
        typeId: "",
        url: ""
      }
    },
    // 获取商品分类列表
    getGoodsProductList() {
      getProductType().then((res) => {
        if (res && res.data) {
          this.productTypeList = res.data;
        }
      });
    },
    // 获取收藏列表
    getCollectProductList() {
      getCollectProductList().then((res) => {
        this.collectProductList = res.rows;
      });
    },
    goDetail(id) {
      this.$router.push({
        path: "/detail",
        query: {
          id: id
        }
      });
    },
    handleCurrentChange(e) {
      this.pageIndex = e;
      // if (this.pitchOn) {
      //   this.handleListClick(this.pitchOn);
      // }
      this.handleListClick();
    },
    // 返回首页
    goIndex() {
      this.$router.push("/index");
    }
  }
};
</script>

<style scoped>
.wrap {
  width: 100%;
}
.header {
  width: 100%;
  display: flex;
  justify-content: space-between;
  padding: 10px 0 10px;
}
.logoIndex{
  width: 25%;
  height: 60px;
  line-height: 60px;
  color: #00afff;
  font-weight: 600;
  font-size: 17px;
  display: flex;
  margin-left: 10px;
  margin-right: 100px;
}
.imgDiv {
  width: 100%;
  height: 60px;
  display: flex;
  justify-content: center;
  align-items: center;
}
.imgLogo{
  width: 90%;
  height: 50px ;
  line-height: 60px;
}
.btn-box {
  width: 200px;
  display: flex;
  align-items: center;
}
.btn-box div {
  background: #67c3a9;
  color: #fff;
  border-radius: 40px;
  padding: 10px;
}
.home {
  margin-right: 20px;
}
.content {
  border-top: 5px solid #e4edec;
  padding: 0 10px;
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
}
.nav {
  width: 10%;
}
.user-name {
  text-align: center;
  width: 100%;
  height: 100%;
  margin-top: 50px;
  border-radius: 50px;
}
.colletion {
  width: 100%;
  height: 70px;
  text-align: center;
  line-height: 70px;
  background: #62c6aa;
  color: #fff;
  font-size: 20px;
  font-weight: bold;
  margin-top: 50px;
}
.publish {
  width: 100%;
  height: 70px;
  text-align: center;
  line-height: 70px;
  background: #62c6aa;
  color: #fff;
  font-size: 20px;
  font-weight: bold;
  margin-top: 2px;
}
.colletion,
.publish::hover {
  color: #62c6aa;
  background: #fff;
}
.user-name img {
  width: 80px;
  height: 80px;
  border-radius: 50px;
}
.commodity-area {
  height: 600px;
  margin-left: 10px;
  background: #e4edec;
  flex: 1;
}
.commodity {
  display: flex;
  flex-wrap: wrap;
  align-content: flex-start;
  /* justify-content: space-between; */
  /* gap: 3%; */
  width: 1300px;
  margin: 0 auto;
}
.commodity-box {
  display: flex;
  flex-direction: column;
  /* margin-right: 10px; */
  text-align: center;
  padding: 10px 30px;
}
.commodity-image {
  width: 198px;
  height: 198px;
}
.paging {
  padding-bottom: 16px;
  position: fixed;
  bottom: 20px;
  left: 54%;
  transform: translateX(-50%);
}
.publish-area {
  margin-left: 10px;
  background: #e4edec;
  flex: 1;
  padding: 10px 30px;
}
.el-login-footer {
  height: 20px;
  line-height: 20px;
  position: fixed;
  bottom: 0;
  width: 100%;
  text-align: center;
  color: #000000;
  font-family: Arial;
  font-size: 12px;
  letter-spacing: 1px;
  background-color: #1ab394;
}
</style>
