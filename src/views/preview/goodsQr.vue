<template>
    <div>
      <basic-container>
        <avue-crud
          :option="tableOption"
          :data="tableData"
          :table-loading="tableLoading"
          :page="page"
          ref="crud"
          @search-change="searchChange"
          @refresh-change="refreshChange(page)"
          @row-save="handleSave"
          @row-update="handleUpdate"
          @size-change="pageSizeChange"
          @current-change="pageChange"
          @error="error"
        >
          <template slot-scope="scope" slot="menuLeft">
            <el-button
              type="primary"
              icon="el-icon-plus"
              size="small"
              plain
              @click.stop="handleAdd()"
              >新增</el-button
            >
          </template>
          <template slot-scope="scope" slot="menu">
            <!-- 如果是未支付状态显示立即支付按钮3244 -->
            <el-button
              type="success"
              icon="el-icon-goods"
              size="mini"
              plain
              v-if="scope.row.payStatus === 'WAIT_PAY'"
              @click.stop="payNom(scope.row)"
              >支付</el-button
            >
            <!-- 如果状态是PAY_SUCCESS则显示立即下载doc的按钮 -->
            <el-button
              type="success"
              icon="el-icon-download"
              size="mini"
              plain
              v-if="scope.row.payStatus === 'PAY_SUCCESS'"
              @click.stop="downloadQr(scope.row)"
              >下载</el-button
            >
            <el-button
              type="primary"
              icon="el-icon-view"
              size="mini"
              plain
              @click.stop="func(scope.row)"
              >查看</el-button
            >
          </template>
          <template slot-scope="scope" slot="brandNameForm">
            <el-select
              v-model="brand"
              placeholder="请选择品牌"
              required
              filterable
              value-key="id"
              @change="changeBrandName"
            >
              <el-option
                v-for="brand in brands"
                :key="brand.id"
                :label="brand.name"
                :value="brand"
              ></el-option>
            </el-select>
          </template>
          <!-- 是否全分类，如果选择是则隐藏二级分类和商品 ,默认为否-->
          <template slot-scope="scope" slot="fullClassificationForm">
            <el-radio-group
              v-model="fullClassification"
              placeholder="请选择是否全分类"
              required
              @change="IsAllClassify(fullClassification)"
              value-key="id"
            >
              <el-radio
                :label="dic.value"
                v-for="dic in yesOrNos"
                :key="dic.value"
              >
                {{ dic.label }}
              </el-radio>
            </el-radio-group>
          </template>
  
          <template slot-scope="scope" slot="firstClassifyNameForm">
            <el-select
              v-model="classify"
              placeholder="请选择商品一级分类"
              required
              @change="chooseSecondClassify"
              value-key="id"
            >
              <el-option
                v-for="classify in classifies"
                :key="classify.id"
                :label="classify.name"
                :value="classify"
              ></el-option>
            </el-select>
          </template>
          <template slot-scope="scope" slot="secondClassifyNameForm">
            <el-select
              v-model="secondClassify"
              placeholder="请选择商品二级分类"
              required
              @change="findGoodsByRootDepartId"
              value-key="id"
            >
              <el-option
                v-for="secondClassify in secondClassifies"
                :key="secondClassify.id"
                :label="secondClassify.name"
                :value="secondClassify"
              ></el-option>
            </el-select>
          </template>
          <template slot-scope="scope" slot="shopNameForm">
            <el-select
              v-model="department"
              placeholder="请选择店铺"
              required
              filterable
              value-key="id"
              @change="changeGoodsId"
              :filter-method="handleFilter"
            >
              <el-option
                v-for="department in departments"
                :key="department.id"
                :label="department.shopName"
                :value="department"
              ></el-option>
            </el-select>
          </template>
          <template slot-scope="scope" slot="goodsNameForm">
            <el-select
              v-model="goods"
              placeholder="请选择商品"
              required
              value-key="id"
            >
              <el-option
                v-for="goods in goodsList"
                :key="goods.id"
                :label="goods.name"
                :value="goods"
              ></el-option>
            </el-select>
          </template>
          <template slot-scope="scope" slot="status">
            <el-tag v-for="obj of QRStatus" :key="obj.value">{{
              obj.label
            }}</el-tag>
          </template>
          <!-- 支付状态 -->
          <template slot-scope="scope" slot="payStatus">
            <el-tag
              size="mini"
              v-for="statusObj of payStatuss"
              :key="statusObj.value"
              v-if="statusObj.value === scope.row.payStatus"
              :type="getStatusTagType(statusObj.value)"
              >{{ statusObj.label }}</el-tag
            >
          </template>
        </avue-crud>
        <!-- 请支付弹窗 -->
        <template>
          <el-dialog
            title="请支付"
            :visible.sync="paymentPopupVisible"
            @close="paymentPopupVisible = false"
            width="450px"
          >
            <div class="payment-popup-content">
              <el-radio>
                余额支付(账户余额:
                {{ balance }}
                ,积分余额:{{ totalIntegral }})
              </el-radio>
              <div class="payment-details">
                <div class="payment-info">
                  <div class="info-label">支付金额:</div>
                  <div class="info-value">
                    {{ payData.money ? payData.money : payData.promotionCost }}元
                  </div>
                </div>
                <div class="payment-info">
                  <div class="info-label">支付积分:</div>
                  <div class="info-value">{{ payData.integral }}积分</div>
                </div>
                <div class="payment-info">
                  <div class="info-label">平台抽成:</div>
                  <div
                    class="info-value"
                    v-if="
                      !payData.platform &&
                        payData.platformCommissionAmount != null
                    "
                  >
                    {{ payData.platformCommissionAmount }}元
                  </div>
                  <div class="info-value" v-else-if="payData.platform">
                    {{ payData.platform }}元
                  </div>
                  <div class="info-value" v-else>0元</div>
                </div>
              </div>
            </div>
            <span slot="footer" class="dialog-footer">
              <el-button @click="paymentPopupVisible = false">取 消</el-button>
              <el-button type="primary" @click="payConfirm">
                确认支付
              </el-button>
            </span>
          </el-dialog>
        </template>
      </basic-container>
    </div>
  </template>
  
  <script>
  import { mapGetters } from "vuex";
  import { goodsQrOption } from "@/const/salesCloud/goodsQrOption";
  import {
    findParentAll,
    addGoodsQr,
    downloadQr,
    findBrand,
    findClassify,
    findAllSuperiorDepartment,
    findGoodsByRootDepartId,
    chooseSecondClassify,
    findAccountBalance,
    findIntegralBalance,
    pay,
  } from "@/api/salesCloud/goodsQr";
  import { findByLabel, formatterTime } from "@/util/util";
  import { DIC } from "@/const/dic";
  
  export default {
    name: "goodsQr",
    data() {
      return {
        tableOption: goodsQrOption, //表格设置属性
        tableData: [], //表格的数据
        tablePage: 1,
        tableLoading: false,
        tabelObj: {},
        page: {
          total: 0, //总页数
          currentPage: 1, //当前页数
          pageSize: 10, //每页显示多少条
        },
        grade: {
          box: false,
          check: [],
        },
        detailVisible: false,
        detailData: [],
        selectRedPacket: {},
        detailSearch: {},
        detailPage: {
          total: 0,
          currentPage: 1,
          pageSize: 10,
        },
        brands: [],
        brand: {},
        classifies: [],
        classify: {},
        departments: [],
        department: {},
        originalOptions: [], // 添加一个用于备份原始选项的数组
        secondClassifies: [],
        secondClassify: {},
        goodsList: [],
        goods: {},
        QRStatus: DIC.QR_STATUS,
        yesOrNos: DIC.YES_NO,
        payStatuss: DIC.PAY_STATUS,
        payData: {},
        balance: 0,
        totalIntegral: 0,
        fullClassification: "NO", // 设置默认值为 "NO"
        paymentPopupVisible: false,
      };
    },
    created() {
      findBrand().then((res) => {
        let data = JSON.parse(res.data);
        this.brands = data.rows;
      });
      findClassify().then((res) => {
        let data = JSON.parse(res.data);
        this.classifies = data;
      });
      this.getDepartment();
      this.handleList();
    },
    watch: {},
    mounted() {
      if (!this.fullClassification || this.fullClassification === "NO") {
        //   console.log(this.$refs.crud.tableOption);
        const columns = this.$refs.crud.tableOption.column;
        columns[5].addVisdiplay = true;
        columns[5].formsolt = true;
        columns[6].addVisdiplay = true;
        columns[6].formsolt = true;
      }
    },
    computed: {
      ...mapGetters(["permission"]),
    },
    props: [],
    methods: {
      // 获取店铺列表
      getDepartment() {
        findAllSuperiorDepartment().then((res) => {
          let data = JSON.parse(res.data).rows;
          this.departments = data;
          // console.log(this.departments);
        });
      },
      handleFilter(value, option) {
        // 如果原始选项为空，就将其初始化为当前选项
        if (this.originalOptions.length === 0) {
          // console.log(this.originalOptions);
          this.getDepartment();
          this.originalOptions = [...this.departments];
        }
  
        // 如果输入值为空，则恢复原始选项，否则过滤选项，只保留包含输入值的选项
        this.departments = value
          ? this.originalOptions.filter((department) => {
              return department.shopName
                .toLowerCase()
                .includes(value.toLowerCase());
            })
          : [...this.originalOptions];
        if (!option) {
          this.department = value;
          this.$refs.crud.$refs.tableForm.form.shopName = value;
        }
      },
  
      changeGoodsId() {
        this.$refs.crud.$refs.tableForm.form.shopName = this.department.shopName;
        this.findGoodsByRootDepartId();
      },
      changeFirstClassifyName() {
        this.$refs.crud.$refs.tableForm.form.firstClassifyName = this.classify;
      },
      changeBrandName() {
        this.$refs.crud.$refs.tableForm.form.brandName = this.brand;
      },
      pageSizeChange(pageSize) {
        this.tableLoading = true;
        let params = this.$refs.crud.searchForm;
        (params.page = this.page.currentPage), (params.size = pageSize);
        findParentAll(params).then((res) => {
          let data = JSON.parse(res.data);
          this.tableData = data.rows;
          this.page = {
            total: data.total,
            currentPage: this.page.currentPage,
            pageSize: pageSize,
          };
          this.tableLoading = false;
        });
      },
      pageChange(page) {
        this.tableLoading = true;
        let params = this.$refs.crud.searchForm;
        params.page = page;
        params.size = this.page.pageSize;
        findParentAll(params).then((res) => {
          let data = JSON.parse(res.data);
          this.tableData = data.rows;
          this.page = {
            total: data.total,
            currentPage: page,
            pageSize: this.page.pageSize,
          };
          this.tableLoading = false;
        });
      },
      searchChange(params) {
        this.tableLoading = true;
        // let params = this.$refs.crud.searchForm
        params.page = this.page.currentPage;
        params.size = this.page.pageSize;
        findParentAll(params).then((res) => {
          let data = JSON.parse(res.data);
          this.tableData = data.rows;
          this.page = {
            total: data.total,
            currentPage: params.page,
            pageSize: params.size,
          };
          this.tableLoading = false;
        });
      },
      refreshChange(page) {
        this.tableLoading = true;
        let params = this.$refs.crud.searchForm;
        params.page = page.currentPage;
        params.size = page.pageSize;
        findParentAll(params).then((res) => {
          let data = JSON.parse(res.data);
          this.tableData = data.rows;
          this.page = {
            total: data.total,
            currentPage: page.currentPage,
            pageSize: page.pageSize,
          };
          this.tableLoading = false;
        });
      },
      clean() {
        this.classify = "";
        this.secondClassify = "";
        this.department = "";
        this.brand = "";
        this.goods = "";
        this.secondClassifies = [];
        this.goodsList = [];
        this.fullClassification = "NO";
      },
      /**
       * @title 打开新增窗口
       * @detail 调用crud的handleadd方法即可
       *
       **/
      handleAdd() {
        this.clean();
        this.$refs.crud.rowAdd();
      },
      handleEdit(row, index) {
        if (
          null != row.startDate &&
          undefined != row.startDate &&
          null != row.endDate &&
          undefined != row.endDate
        ) {
          let time = new Array();
          time.push(row.startDate);
          time.push(row.endDate);
          row.startDate_endDate = time;
        }
        this.$refs.crud.rowEdit(row, index);
      },
      // 是否全分类，如果选择是则隐藏二级分类和商品>
      IsAllClassify(val) {
        const columns = this.$refs.crud.tableOption.column;
        if (val === "YES") {
          columns[5].addVisdiplay = false;
          columns[5].formsolt = false;
          columns[6].addVisdiplay = false;
          columns[6].formsolt = false;
          // 若是全分类，则清空二级分类和商品
          this.secondClassify = "";
          this.goods = "";
        } else {
          columns[5].addVisdiplay = true;
          columns[5].formsolt = true;
          columns[6].addVisdiplay = true;
          columns[6].formsolt = true;
        }
      },
      /**
       * @title 获取数据
       * @detail 赋值为tableData表格即可
       *
       **/
      handleList() {
        this.tableLoading = true;
        let parmas = {
          page: this.page.currentPage,
          size: this.page.pageSize,
        };
        findParentAll(parmas).then((res) => {
          let data = JSON.parse(res.data);
          this.tableData = data.rows;
          this.page = {
            total: data.total,
            currentPage: this.page.currentPage,
            pageSize: this.page.pageSize,
          };
          findAccountBalance().then((res) => {
            let data = JSON.parse(res.data);
            console.log(data);
            this.balance = data.balance;
            console.log(this.balance);
          });
          let parmas = {
            page: this.page.currentPage,
            size: this.page.pageSize,
          };
          findIntegralBalance(parmas).then((res) => {
            let data = JSON.parse(res.data);
            this.totalIntegral = data.balance;
          });
          this.tableLoading = false;
        });
      },
      /**
       * @title 数据添加
       * @param row 为当前的数据
       * @param done 为表单关闭函数
       *
       **/
      handleSave(row, done) {
        //   console.log("!!!!!!", row);
        let data = {};
        if (null != row.startDate_endDate && undefined != row.startDate_endDate) {
          data.startTime = row.startDate_endDate[0];
          data.endTime = row.startDate_endDate[1];
        }
        data.shopName = row.shopName;
        data.goodsBrandId = this.brand.id;
        data.goodsId = this.goods.id;
        data.firstClassify = this.classify.id;
        data.secondClassify = this.secondClassify.id;
        data.stuffId = this.department.dutyStuff;
        data.fullClassification = row.fullClassification;
        data.shopId = this.department.id ? this.department.id : "";
        data.integral = row.integral;
        data.promotionCost = row.promotionCost;
        data.num = row.num;
        data.name = row.name;
        data.typeName = row.typeName;
        data.ratioType = row.ratioType;
        data.platformRatio = row.platformRatio;
        addGoodsQr(data).then((response) => {
          if (response.code == "200") {
            //   console.log(JSON.parse(response.data));
            this.payData = JSON.parse(response.data);
            this.paymentPopupVisible = true;
            this.$message({
              showClose: true,
              message: "添加成功",
              type: "success",
            });
            console.log('this.payData', this.payData);
          } else {
            this.$message({
              showClose: true,
              message: response.message,
              type: "error",
            });
          }
        });
        done();
      },
      // 确认支付
      payConfirm() {
        pay(this.payData.id).then((response) => {
          if (response.code == "200") {
            this.$message({
              showClose: true,
              message: "支付成功",
              type: "success",
            });
            this.paymentPopupVisible = false; 
            this.handleList();
            this.IsAllClassify()
          } else {
            this.paymentPopupVisible = false;
            this.handleList();
            this.IsAllClassify()
            this.$message({
              showClose: true,
              message: response.message,
              type: "error",
            });
          }
        });
      },
      getStatusTagType(status) {
        if (status === "WAIT_PAY") {
          return "danger";
        } else {
          return "success";
        }
      },
      /**
       * @title 数据删除
       * @param row 为当前的数据
       * @param index 为当前更新数据的行数
       *
       **/
      handleDel(row) {
        console.log(row);
        // 确认删除商品二维码（name）吗
        this.$confirm("确认删除商品二维码（" + row.name + "）吗？", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        })
          .then(() => {
            deleteQr(row.id).then((res) => {
              let data = JSON.parse(res.data);
              console.log(data);
              if (data == null) {
                this.$message({
                  showClose: true,
                  message: "删除失败",
                  type: "error",
                });
              } else {
                this.$message({
                  showClose: true,
                  message: "删除成功",
                  type: "success",
                });
                this.handleList();
              }
            });
          })
          .catch(() => {
            this.$message({
              showClose: true,
              message: "已取消删除",
              type: "info",
            });
          });
      },
      /**
       * @title 数据更新
       * @param row 为当前的数据
       * @param index 为当前更新数据的行数
       * @param done 为表单关闭函数
       *
       **/
      handleUpdate(row, index, done) {},
      chooseSecondClassify() {
        this.changeFirstClassifyName();
        this.secondClassify = "";
        this.goodId = "";
        chooseSecondClassify(this.classify.id).then((res) => {
          let data = JSON.parse(res.data);
          this.secondClassifies = data;
          // this.findGoodsByRootDepartId();
        });
      },
      findGoodsByRootDepartId() {
        let data = {};
        this.goods = "";
  
        if (typeof this.classify.id != "undefined") {
          data.firstClassify = this.classify.id;
        }
        if (typeof this.secondClassify.id != "undefined") {
          data.secondClassify = this.secondClassify.id;
        }
        if (typeof this.department.id != "undefined") {
          data.agentId = this.department.id;
        }
        findGoodsByRootDepartId(data).then((res) => {
          let data = JSON.parse(res.data).rows;
          console.log(data);
          if (data == null) {
            this.goodsList = [];
          } else {
            this.goodsList = data;
          }
        });
      },
      // 立即支付
      payNom(row) {
        this.payData = row;
        console.log(this.payData);
        this.paymentPopupVisible = true;
      },
      //立即下载
      downloadQr(row) {
        downloadQr(row.id).then((response) => {
          console.log(response);
          // 提取文件名
          const filename = response.headers["content-disposition"].match(
            /filename=(.*)/
          )[1];
          // 将二进制流转为blob
          const blob = new Blob([response.data], {
            type: "application/octet-stream",
          });
          if (typeof window.navigator.msSaveBlob !== "undefined") {
            // 兼容IE，window.navigator.msSaveBlob：以本地方式保存文件
            window.navigator.msSaveBlob(blob, decodeURI(filename));
          } else {
            // 创建新的URL并指向File对象或者Blob对象的地址
            const blobURL = window.URL.createObjectURL(blob);
            // 创建a标签，用于跳转至下载链接
            const tempLink = document.createElement("a");
            tempLink.style.display = "none";
            tempLink.href = blobURL;
            tempLink.setAttribute("download", decodeURI(filename));
            // 兼容：某些浏览器不支持HTML5的download属性
            if (typeof tempLink.download === "undefined") {
              tempLink.setAttribute("target", "_blank");
            }
            // 挂载a标签
            document.body.appendChild(tempLink);
            tempLink.click();
            document.body.removeChild(tempLink);
            // 释放blob URL地址
            window.URL.revokeObjectURL(blobURL);
            this.handleList();
          }
        });
      },
      func(row) {
        this.$router.push({
          name: "商品二维码详情",
          path: "/sales/goodsQrDetails",
          query: { id: row.id },
        });
      },
      error(err) {
        this.$message.success("请查看控制台");
        console.log(err);
      },
    },
  };
  </script>
  
  <style lang="scss" scoped>
  .table-container {
    padding: 8px 10px;
  }
  
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
  // ::v-deep样式穿透.el-dialog__body
  ::v-deep .el-dialog__body {
    text-align: center;
  }
  ::v-deep .el-dialog__header {
    padding: 20px 20px 10px;
    text-align: center;
    font-family: fangsong;
    font-weight: bold;
  }
  .payment-popup-content {
    margin: 20px 0;
  }
  
  .payment-details {
    margin: 20px 0 0 60px;
  }
  
  .payment-info {
    display: flex;
  }
  .info-label {
    color: #99a9bf;
  }
  .info-value {
    color: #606266;
    margin-left: 10px;
  }
  .dialog-footer {
    text-align: center;
    margin-top: 20px;
  }
  </style>
  
