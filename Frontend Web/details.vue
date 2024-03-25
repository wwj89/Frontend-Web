
<template>
  <div class="wrap">
    <div class="header">
      <div class="logoIndex">
        <div class="imgDiv">
          <img src="../assets/images/profile.png" class="imgLogo"/>
        </div>
        <div style="font-family: Lucida Handwriting">Campus second-hand market</div>
      </div>
      <div class="user-info" @click="goUserInfo()">
        <img :src="loginImgUrl" />
        <div class="name">{{ loginUserInfo.userName }}</div>
      </div>
      <div class="login-and-register" style="margin-right: 50px"  >
        <div class="login-index" @click="LogOut()">logout</div>
        <!-- <div class="register" @click="goRegister()">注册</div> -->
      </div>
      <div class="btn-box">
        <div class="home" @click="goIndex()">Home</div>
      </div>
    </div>
    <div class="box">
      <div class="content">
        <div class="banner">
          <img :src="path" alt="" />
          <div class="slideshow">
            <div v-for="(imgUrl, index) in urls" :key="index" style="width: 20%; margin-right: 10px;">
              <img  :src="imgUrl" alt="" @click="switchImg(imgUrl)"/>
            </div>
          </div>
        </div>
        <div class="detailed-information">
          <div class="name">{{ goodsDetail.name }}</div>
          <div class="price">
            <span class="red">￥{{ goodsDetail.price }}元</span>
            <!-- <span>不可小刀</span> -->
          </div>
          <div class="cut-off-rule">
            -------------------It's been viewed {{
              goodsDetail.pageViewNum
            }} times-------------------
          </div>
          <div class="seller wrap-box">
            <div class="left">Seller</div>
            <div class="right">{{ goodsDetail.createName }}</div>
          </div>
          <div class="phone wrap-box">
            <div class="left">Phone</div>
            <div class="right">11111111111</div>
          </div>
          <div class="qq wrap-box">
            <div class="left">Q&nbsp;&nbsp;&nbsp;&nbsp;Q</div>
            <div class="right">{{ goodsDetail.qq }}</div>
          </div>
          <div class="weixin-name wrap-box">
            <div class="left">WeChat</div>
            <div class="right">{{ goodsDetail.wx }}</div>
          </div>
          <div class="release-time wrap-box">
            <div class="left">Time</div>
            <div class="right">{{ goodsDetail.createTime }}</div>
          </div>
          <div class="buy-btn">
            <!-- <div>Buy</div> -->
            <div @click="handleCollect()" v-if="!goodsDetail.isCollect">
              Collect
            </div>
            <div @click="handleCollect()" v-else>Cancel Collect</div>
          </div>
        </div>
      </div>
      <div class="article-introduction">
        <div class="name">
          <img :src="detailsImgUrl" alt="" />
        </div>
        <div class="introduce">
          {{ goodsDetail.description }}
        </div>
      </div>
      <div class="comments-section">
        <h2>comments</h2>
        <div class="infinite-list" style="overflow: auto">
          <div
            v-for="(i, index) in commentList"
            :key="index"
            class="infinite-list-item"
          >
            <div class="head-portrait">
              <img :src="avatars[index]" alt="" />
            </div>
            <div class="comment">
              <div class="comment-head">
                <div class="user-name">{{ i.userName }}</div>
                <div class="comment-auther" v-if="i.userId == i.commentUserId">Author</div>
                <div class="user-time">{{ i.createTime }}</div>
              </div>
              <div class="comment-content">{{ i.content }}</div>
            </div>
          </div>
        </div>
      </div>
      <div class="make-comments">
        <img :src="loginImgUrl" alt="" />
        <input type="text" v-model="comment" />
        <div class="send" @click="sendComment()">send</div>
      </div>
    </div>
  </div>
</template>

<script>
import {
  getProductList,
  getProductType,
  commentAdd,
  getGoodsDetail,
  getCommentList,
  collectAdd,
  collectCancel
} from "@/api/project/product.js";
import { getInfo } from "@/api/login.js";
export default {
  data() {
    return {
      search: "",
      count: 10,
      comment: "",
      id: null, //当前商品Id
      goodsDetail: {}, // 商品详细信息
      loginUserInfo: {}, //当前登陆人信息
      commentList: [], //评论列表
      path: "",//大图链接,
      urls: [], //图片集合
      avatars : []
    };
  },
  mounted() {
    this.id = this.$route.query.id;
    this.getGoodsDetail();
    this.getUserInfo();
    this.getCommentList();
  },
  components: {},
  computed: {
    loginImgUrl: function (params) {
      return process.env.VUE_APP_BASE_API + this.loginUserInfo.avatar;
    },
    detailsImgUrl: function (params) {
      return process.env.VUE_APP_BASE_API + this.goodsDetail.avatar;
    }
  },
  methods: {
    switchImg(imgUrl){
      this.path = imgUrl;
    },
    goUserInfo() {
      this.$router.push("/userinfo");
    },
    // 退出登录
    async LogOut() {
      this.$confirm("Are you sure you want out?", "Tips", {
        confirmButtonText: "OK",
        cancelButtonText: "Cancel",
        type: "warning"
      })
        .then(() => {
          this.$store.dispatch("LogOut").then(() => {
            location.href = "/login";
          });
        })
        .catch(() => {});
    },
    // 去注册
    goRegister(id) {
      this.$router.push("/register");
    },
    // 搜索
    searchHandle() {
      getProductList(this.search).then((res) => {
        if (res && res.rows) {
          this.productList = res.rows;
        }
      });
    },
    // 发布评论
    sendComment() {
      let time = new Date().getTime();
      let obj = {
        content: this.comment,
        createTime: time,
        productId: this.goodsDetail.id
      };
      // obj = JSON.stringify(obj);
      commentAdd(obj).then((res) => {
        if(res.code === 200){
          this.comment = "";
          //刷新评论列表
          this.getCommentList();
          this.$message.success("评论成功！")
        }
      });
    },
    // 获取商品详情
    getGoodsDetail() {
      getGoodsDetail(this.id).then((res) => {
        console.log("res",res.data)
        this.goodsDetail = res.data;
        if (res.data.url) {
          this.path = res.data.url.split(",")[0];
          this.urls = res.data.url.split(",");
        }
      });
    },
    // 获取登陆人信息
    getUserInfo() {
      getInfo().then((res) => {
        this.loginUserInfo = res.user;
      });
    },
    // 获取评论区
    getCommentList() {
      getCommentList(this.id).then((res) => {
        res.data.forEach(item => {
          this.avatars.push(process.env.VUE_APP_BASE_API + item.avatar);
        })
        this.commentList = res.data;
      });
    },
    // 收藏
    handleCollect() {
      let time = new Date().getTime();
      let obj = {
        collectTime: time,
        productId: this.id
      };
      if (this.goodsDetail.isCollect) {
        collectCancel(this.id).then((res) => {
          this.$set(this.goodsDetail, "isCollect", false);
        });
      } else {
        collectAdd(obj).then((res) => {
          this.$set(this.goodsDetail, "isCollect", true);
        });
      }
      // this.getGoodsDetail();
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
.box {
  background: #e4edec;
  padding: 10px 100px 20px;
}
.header {
  padding: 10px 100px 10px;
  width: 100%;
  display: flex;
  justify-content: flex-start;
  background-color: #e4edec;
  border-bottom: 5px solid #83b9e0;
}
.logoIndex{
  width: 50%;
  height: 60px;
  line-height: 60px;
  color: #00afff;
  font-weight: 600;
  font-size: 17px;
  display: flex;
  margin-left: 10px;
}
.imgDiv {
  width: 50px;
  height: 60px;
  display: flex;
  justify-content: center;
  align-items: center;
}
.imgLogo{
  width: 40px;
  height: 40px;
  line-height: 60px;
  border-radius: 50px ;
}
.search {
  display: flex;
  align-items: center;
  border: 1px solid #c5e8c8;
  width: 400px;
  height: 80px;
  padding: 10px;
  border-radius: 30px;
  color: #fff;
}
.search-btn {
  width: 100px;
  border: 1px solid #dcdfe6;
  height: 40px;
  text-align: center;
  border-radius: 5px;
  font-size: 20px;
  margin-left: 10px;
  line-height: 40px;
  background: #62c6aa;
}
.user-info {
  display: flex;
  align-items: center;
  width: 20%;
}
.user-info img {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  margin-right: 10px;
}
.user-info .name {
  background: #67c3a9;
  color: #fff;
  border-radius: 40px;
  padding: 10px;
}
.login-and-register {
  display: flex;
  align-items: center;
  color: #66c7ac;
}
.register {
  margin-left: 30px;
}
.content {
  width: 100%;
  display: flex;
  align-items: top;
  margin-top: 50px;
}
.banner {
  width: 60%;
}
.banner img {
  width: 100%;
  height: 500px;
}
.slideshow {
  width: 500px;
  display: flex;
  padding: 10px 0;
}
.slideshow img {
  width: 100px;
  height: 100px;
  margin-left: 10px;
}
.detailed-information {
  width: 30%;
  padding: 10px;
}
.detailed-information .name {
  color: #28827a;
  font-size: 30px;
  font-weight: bold;
}
.red {
  color: red;
  margin-right: 10px;
}
.detailed-information .price {
  font-size: 18px;
  margin-top: 30px;
}
.wrap-box {
  width: 100%;
  display: flex;
  margin-top: 30px;
  font-size: 18px;
}
.left {
  width: 20%;
  height: 30px;
  margin-right: 10px;
  text-align: center;
  background: #64c8a8;
  border-radius: 10px;
  color: #fff;
  line-height: 20px;
  padding: 5px;
}
.right {
  flex: 1;
  text-align: left;
}
.buy-btn {
  display: flex;
  justify-content: space-between;
  width: 100%;
  margin-top: 20px;
}
.buy-btn div {
  width: 210px;
  height: 80px;
  background: #feb405;
  color: #fff;
  border-radius: 26px;
  margin-top: 100px;
  font-size: 24px;
  line-height: 80px;
  text-align: center;
}
.article-introduction {
  background: #fff;
  padding: 50px;
  display: flex;
  align-items: center;
}
.article-introduction .name img {
  width: 80px;
  height: 80px;
  border-radius: 50%;
  margin-right: 30px;
}
.comments-section {
  height: 200px;
  background: #fff;
  overflow-y: scroll;
  margin-top: 20px;
}
.comments-section h2 {
  background: #63cab2;
  width: 140px;
  height: 50px;
  color: #fff;
  margin: 0;
  line-height: 50px;
  text-align: center;
  border-radius: 10px;
}
.infinite-list-item {
  display: flex;
  align-items: center;
  padding: 10px;
}
.head-portrait {
  margin-right: 10px;
}
.head-portrait img {
  width: 80px;
  height: 80px;
  border-radius: 50%;
}
.comment {
  width: 500px;
  height: 60px;
  line-height: 30px;
}
.comment-head{
  width: 100%;
  height: 30px;
  display: flex;
}
.comment .user-name {
  width: 40px;
  height: 30px;
  color: #61c9ae;
  font-size: 14px;
}
.comment-auther{
  width: 30px;
  height: 15px;
  align-self: center;
  background-color: #ff6600;
  font-size: 8px;
  border-radius: 5px;
  color: #ffffff;
  line-height: 15px;
  text-align: center;
}
.comment .user-time {
  font-size: 14px;
  margin-left: 10px;
}
.comment-content {
  font-weight: bold;
}
.make-comments {
  display: flex;
  align-items: center;
  padding: 10px;
  background: #fff;
}
.make-comments img {
  width: 80px;
  height: 80px;
  border-radius: 50%;
  margin-right: 10px;
}
.make-comments input {
  font-size: 20px;
  width: 70%;
  height: 80px;
  background: #c4e8c8;
  border: 1px solid #a8c6b4;
  border-radius: 10px;
  text-indent: 10px;
}
.send {
  width: 80px;
  height: 80px;
  text-align: center;
  line-height: 80px;
  border-radius: 10px;
  background: #63c7af;
  color: #fff;
  margin-left: 5px;
}
.cut-off-rule {
  width: 100%;
  text-align: center;
  margin-top: 10px;
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
</style>
