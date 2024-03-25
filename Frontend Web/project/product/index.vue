<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryForm" size="small" :inline="true" v-show="showSearch" label-width="150px">
      <el-form-item label="Product Category" prop="status">
        <el-select
          v-model="queryParams.typeId"
          placeholder="Product Category"
          clearable
          style="width: 240px"
        >
          <el-option
            v-for="type in productType"
            :key="type.id"
            :label="type.typeName"
            :value="type.id"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="Product Name" prop="name">
        <el-input
          v-model="queryParams.name"
          placeholder="Please enter the product name"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="Issue Time">
        <el-date-picker
          v-model="daterangeCreateTime"
          style="width: 240px"
          value-format="yyyy-MM-dd"
          type="daterange"
          range-separator="-"
          start-placeholder="Start Time"
          end-placeholder="End Time"
        ></el-date-picker>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">Search</el-button>
        <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">Reset</el-button>
      </el-form-item>
    </el-form>

    <el-table v-loading="loading" :data="productList">
      <el-table-column label="id" prop="id"  align="center" v-if="false"/>
      <el-table-column label="Product Name" align="center" prop="name" />
      <el-table-column label="Category Name" align="center" prop="typeName" />
      <el-table-column label="Publisher" align="center" prop="createName" />
      <el-table-column label="Price" align="center" prop="price" />
      <el-table-column label="Issue Time" align="center" prop="createTime" width="180" />
      <el-table-column label="Number views" align="center" prop="pageViewNum" />
      <el-table-column label="Operate" align="center" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="text"
            icon="el-icon-view"
            @click="handleDetails(scope.row)"
          >Details</el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total>0"
      :total="total"
      :page.sync="queryParams.pageNum"
      :limit.sync="queryParams.pageSize"
      @pagination="getList"
    />

    <!-- 详情对话框 -->
    <el-dialog :title="title" :visible.sync="open" width="500px" append-to-body>
      <el-form ref="form" :model="form" :rules="rules" label-width="145px">
        <el-form-item label="Product Name：" prop="name">{{form.name}}</el-form-item>
        <el-form-item label="Product Category：" prop="typeName">{{form.typeName}}</el-form-item>
        <el-form-item label="Publisher：" prop="createName">{{form.createName}}</el-form-item>
        <el-form-item label="Price：" prop="price">{{form.price}}</el-form-item>
        <el-form-item label="Issue Time：" prop="createTime">{{form.createTime}}</el-form-item>
        <el-form-item label="Page View：" prop="pageViewNum">{{form.pageViewNum}}</el-form-item>
        <el-form-item label="Product picture：" >
          <template>
            <div style="width: 100%;display: flex;flex-wrap: wrap;">
              <div v-for="item in form.imgUrls" style="padding-right: 20px;">
                <img :src="item" style="width: 100px; height: 100px" />
              </div>
            </div>
          </template>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="cancel">Cancel</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { listProduct, getProduct, getProductType} from "@/api/project/product";

export default {
  name: "Product",
  data() {
    return {
      // 遮罩层
      loading: true,
      // 选中数组
      ids: [],
      // 非单个禁用
      single: true,
      // 非多个禁用
      multiple: true,
      // 显示搜索条件
      showSearch: true,
      // 总条数
      total: 0,
      // 商品表格数据
      productList: [],
      //商品分类数据
      productType: [],
      // 弹出层标题
      title: "",
      // 是否显示弹出层
      open: false,
      // 商品图片路径时间范围
      daterangeCreateTime: [],
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        typeId: null,
        name: null,
        price: null,
        intro: null,
        description: null,
        createTime: null,
        userName: null,
        pageViewNum: null,
      },
      // 表单参数
      form: {},
      // 表单校验
      rules: {
      }
    };
  },
  created() {
    this.getList();
    this.getProductTypeList();
  },
  methods: {
    /** 查询商品列表 */
    getList() {
      this.loading = true;
      this.queryParams.params = {};
      if (null != this.daterangeCreateTime && '' != this.daterangeCreateTime) {
        this.queryParams.params["beginCreateTime"] = this.daterangeCreateTime[0];
        this.queryParams.params["endCreateTime"] = this.daterangeCreateTime[1];
      }
      listProduct(this.queryParams).then(response => {
        this.productList = response.rows;
        this.total = response.total;
        this.loading = false;
      });
    },
    getProductTypeList(){
      getProductType().then(res => {
        this.productType = res.data;
      });
    },
    // 取消按钮
    cancel() {
      this.open = false;
      this.reset();
    },
    // 表单重置
    reset() {
      this.form = {
        id: null,
        typeName: null,
        name: null,
        price: null,
        intro: null,
        description: null,
        createTime: null,
        userName: null,
        pageViewNum: null,
        imgUrls: [],
        url: null
      };
      this.resetForm("form");
    },
    /** 搜索按钮操作 */
    handleQuery() {
      this.queryParams.pageNum = 1;
      this.getList();
    },
    /** 重置按钮操作 */
    resetQuery() {
      this.daterangeCreateTime = [];
      this.resetForm("queryForm");
      this.handleQuery();
    },
    // 多选框选中数据
    handleSelectionChange(selection) {
      this.getList();
    },
    /** 查看按钮操作 */
    handleDetails(row) {
      const id = row.id;
      console.log("id",id)
      getProduct(id).then(response => {
        this.form = response.data;
        this.open = true;
        this.title = "Product Details";
      });
    }
  }
};
</script>
