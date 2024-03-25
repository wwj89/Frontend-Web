
<template>
  <div class="wrap">
    <div class="header">
      <div class="logoIndex">
        <div class="imgDiv">
          <img src="../assets/images/logo.png" class="imgLogo"/>
        </div>
      </div>
      <div class="search">
        <el-input v-model="search" background="#000" placeholder="Please enter product name search">
          <i slot="prefix" class="el-input__icon el-icon-search"></i>
        </el-input>
        <div class="search-btn" @click="searchHandle()">Search</div>
      </div>
      <div class="user-info" v-if="isLogin" @click="goUserInfo()">
        <img :src="loginImgUrl" />
        <div class="name">{{ loginUserInfo.userName }}</div>
      </div>
      <div class="login-and-register">
        <div class="login-index" v-if="!isLogin" @click="goLogin()">Login</div>
        <div class="login-index" v-if="isLogin" @click="LogOut()">Logout</div>
        <div class="register-index" v-if="!isLogin" @click="goRegister()">
          Sign
        </div>
      </div>
      <div></div>
    </div>
    <div class="content">
      <!-- :span="4" -->
      <div class="nav">
        <el-col class="menu-wrap">
          <el-menu
            :default-active="pitchOn"
            class="el-menu-vertical-demo"
            @open="handleOpen"
            @close="handleClose"
            background-color="#fff"
            text-color="#000"
            active-text-color="#ffd04b"
          >
            <div v-for="item in productTypeList" :key="item.id" class="typeMenuDiv">
              <el-menu-item
                :index="item.code"
                @click="handleListClick(item.id)"
                class="typeMenu"
              >
                <i class="el-icon-menu"></i>
                <span slot="title">{{ item.code }}</span>
              </el-menu-item>
            </div>
          </el-menu>
        </el-col>
      </div>
      <div class="commodity-area">
        <div class="commodity" v-for="item in productList" :key="item.id">
          <div class="commodity-box" @click="goDetail(item.id)">
            <img :src="item.path" alt="" class="commodity-image" />
            <div class="commodity-name">
              {{ item.name }}
            </div>
            <div class="commodity-price">￥{{ item.price }}</div>
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
    </div>
    <!--  底部  -->
    <div class="el-login-footer">
      <span>Copyright © 2018-2023 ruoyi.vip All Rights Reserved.</span>
    </div>
  </div>
</template>

<script>
import { getProductList, getProductType } from "@/api/project/product.js";
import { getToken } from "@/utils/auth";
import {getInfo} from "@/api/login";
export default {
  data() {
    return {
      search: "",
      productTypeList: [], //商品分类列表
      pitchOn: "1", //当前选中的分类
      productList: [], //商品列表,
      isLogin: false, //是否登录
      pageIndex: 1, //分页当前页数
      pageSize: 18, //一页的数量
      total: null, //数据总量
      loginUserInfo: {}, //当前登陆人信息
    };
  },
  created: function () {
    this.getGoodsProductList();
    this.getUserInfo();
    // this.$nextTick(() => {});
    if (getToken()) {
      this.isLogin = true;
    } else {
      this.isLogin = false;
    }
  },
  components: {},
  computed: {
    pageNum: function () {
      let s = this.total / this.pageSize;
      let y = this.total % this.pageSize;
      if (y != 0) {
        s += 1;
      }
      return s;
    },
    loginImgUrl: function (params) {
      return process.env.VUE_APP_BASE_API + this.loginUserInfo.avatar;
    }
  },
  methods: {
    // 获取登陆人信息
    getUserInfo() {
      getInfo().then((res) => {
        this.loginUserInfo = res.user;
      });
    },
    handleOpen(key, keyPath) {
    },
    handleClose(key, keyPath) {
    },
    // 搜索
    searchHandle() {
      let obj = {
        productName: this.search
      };
      getProductList(obj).then((res) => {
        if (res && res.rows) {
          this.productList = res.rows;
          this.total = res.total;
          this.productList.forEach(item => {
            let urls =item.url.split(",");
            item.path = urls[0];
          })
        }
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
    // 获取商品分类列表
    getGoodsProductList() {
      getProductType().then((res) => {
        if (res && res.data) {
          this.productTypeList = res.data;
          // this.productTypeList.map((item) => {
          //   if (item.code == "All") {
          this.handleListClick();
          //   }
          // });
        }
      });
    },
    // 菜单点击
    handleListClick(id) {
      let obj = {
        pageNum: this.pageIndex,
        pageSize: this.pageSize
      };
      if (id != 1) {
        obj.typeId = id;
      } else {
        obj.typeId = null;
      }
      getProductList(obj).then((res) => {
        this.productList = res.rows;
        this.total = res.total;
        this.productList.forEach(item => {
          let urls =item.url.split(",");
          item.path = urls[0];
        })
      });
    },
    // 去登陆
    async goLogin() {
      this.$router.push("/login");
      // this.$store.dispatch("LogOut").then(() => {
      //   location.href = "/login";
      // });
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
    handleCurrentChange(e) {
      this.pageIndex = e;
      // if (this.pitchOn) {
      //   this.handleListClick(this.pitchOn);
      // }
      this.handleListClick();
    },
    goUserInfo() {
      this.$router.push("/userinfo");
    },
  }
};
</script>

<style lang="scss" scoped>
.wrap {
  height: 100%;
}
.header {
  display: flex;
  justify-content: flex-start;
  background: #e4edec;
  padding: 10px 100px 20px;
  border-bottom: 5px solid #83b9e0;
}
.search {
  display: flex;
  align-items: center;
  border: 1.5px solid #b2e9b7;
  width: 40%;
  height: 60px;
  padding: 10px;
  border-radius: 15px;
  margin-right: 40%;
  line-height: 50px;
}
::v-deep .el-input__inner {
  height: 25px !important;
  line-height: 36px !important;
  background: #b2e9b7;
}
.el-input--medium .el-input__icon {
  line-height: 25px;
}
.search-btn {
  width: 80px;
  border: 1px solid #dcdfe6;
  height: 25px;
  text-align: center;
  border-radius: 5px;
  font-size: 16px;
  margin-left: 10px;
  line-height: 25px;
  background: #62c6aa;
  color: #fff;
}
.login-and-register {
  display: flex;
  align-items: center;
  color: #66c7ac;
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
  width: 150%;
  height: 50px;
  line-height: 60px;
}
.register {
  margin-left: 30px;
}
.content {
  height: 100%;
  display: flex;
  padding: 0px 100px 10px;
}
.nav {
  height: 100%;
}
.menu-wrap {
  width: 140px;
  min-width: 120px;
  margin-top: 20px;
}
.el-menu-vertical-demo {
  height: 100%;
}
.typeMenuDiv{
  &:hover{
    background-color: #00afff !important;
  }
}
.typeMenu{
  border-left: 5px solid rgba(113,177,238,0.54);
  border-right: 5px solid rgba(113,177,238,0.54);
  border-top: 1px solid rgba(113,177,238,0.54);
  border-bottom: 1px solid rgba(113,177,238,0.54);
  height: 50px;
  line-height: 45px;
  margin-bottom: 5px;

}
.commodity-area {
  width: 100%;
  margin-left: 10px;
  background: #e4edec;
  display: flex;
  flex-wrap: wrap;
  align-content: flex-start;
  /* justify-content: space-between; */
  padding: 10px;
}

.commodity {
  /* display: flex;
  flex-wrap: wrap;
  align-content: flex-start;
  justify-content: space-between; */
}

.commodity-box {
  display: flex;
  flex-direction: column;
  /* margin: 0 10px 0 10px; */
  margin-right: 10px;
  text-align: center;
  padding: 10px;
}

.commodity-image {
  width: 198px;
  height: 198px;
}

.paging {
  padding-bottom: 16px;
  position: fixed;
  bottom: 3px;
  left: 54%;
  transform: translateX(-50%);
}
.register-index {
  margin-left: 60px;
}
.el-login-footer {
  height: 20px;
  line-height: 20px;
  bottom: 0;
  width: 100%;
  text-align: center;
  color: #000000;
  font-family: Arial;
  font-size: 12px;
  letter-spacing: 1px;
  background-color: #1ab394;
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
</style>
