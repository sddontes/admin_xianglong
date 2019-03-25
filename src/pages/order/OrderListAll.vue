/**
* Created by alan on 17-5-11.
*/
<template>
  <section>
    <!--工具条-->
    <el-form :inline="true" :model="searchForm" class="toolbar" label-position="right" label-width="70px">
      <el-row>
        <el-form-item label="订单编号">
          <el-input v-model="searchForm.uuid" width="200"></el-input>
        </el-form-item>
        <el-form-item label="姓名">
          <el-input v-model="searchForm.realName"></el-input>
        </el-form-item>
        <el-form-item label="手机号">
          <el-input v-model="searchForm.mobile"></el-input>
        </el-form-item>
        <el-form-item label="订单状态" prop="status">
          <el-select v-model="searchForm.status" placeholder="请选择" clearable>
            <el-option v-for="item in orderStatus" :label="item.name" :key="item.code" :value="item.code"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="复借订单">
          <el-select v-model="searchForm.isAgain" placeholder="请选择" clearable>
            <el-option v-for="item in isAgainOrder" :label="item.name" :key="item.code" :value="item.code"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="来源">
         <el-select v-model="searchForm.channel" placeholder="请选择" clearable>
            <el-option v-for="item in qudaolist" :label="item.dicItemName" :key="item.dicId" :value="item.dicItemValue.split('#')[0]"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="">
          <el-button @click="search" type="info" style="width: 170px">查询</el-button>
        </el-form-item>
      </el-row>
    </el-form>
    <!--列表-->
    <template>
      <el-table :data="gridData" highlight-current-row v-loading="gridLoading" class="grid">
        <el-table-column label="订单编号" prop="uuid" width="180">
        </el-table-column>
        <el-table-column label="姓名" prop="realName" width="80">
        </el-table-column>
  <!--       <el-table-column label="手机号" prop="mobileNumber" width="130">
        </el-table-column> -->
        <el-table-column label="复借订单" prop="isAgain">
         <template scope="scope">
            <el-tag :type="scope.row.isAgain==1? 'success' : 'danger'" close-transition>{{scope.row.isAgain==1?'是':'否'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="订单状态" prop="status">
          <template scope="scope">
            <span>{{getOrderType(scope.row.status)}}</span>
          </template>
        </el-table-column>
        <el-table-column label="申请金额" prop="amountApply">
        </el-table-column>
        <el-table-column label="申请期限" prop="borrowingTerm">
        </el-table-column>
        <el-table-column label="申请时间" prop="createTime" min-width="116">
          <template scope="scope">
            <span>{{getUnixTime(scope.row.createTime)}}</span>
          </template>
        </el-table-column>
        <el-table-column label="来源" prop="channel">
          <template scope="scope">
            {{getChannelName(scope.row.channel)}}
          </template>
        </el-table-column>
        <el-table-column label="银行卡号" width="100" prop="status">
          <template scope="scope">
            <el-button
              type="primary"
              size="small"
              @click="getBankNo(scope.row)">获取
            </el-button>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="140" prop="status">
          <template scope="scope">
            <el-button
              type="primary"
              size="small"
              @click="rowClick(scope.row)">查看
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </template>

    <el-dialog
      title="用户银行卡信息"
      :visible.sync="bankInfseen"
      width="30%"
      :before-close="handleClose">
      <span>姓名：{{userBankInf.bankCardName}}<br>手机号：{{userBankInf.bankNumberMobile}}<br>开户行：{{userBankInf.bankName}}<br>银行卡号：{{userBankInf.bankNumberNo}}</span>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="bankInfseen = false">确 定</el-button>
      </span>
    </el-dialog>
    <!--分页-->
    <el-pagination class="pager" @size-change="pageSizeChange" @current-change="pageIndexChange" :current-page="pageIndex" :page-size="pageSize"
                   layout="total, sizes, prev, pager, next, jumper" :total="dataTotal">
    </el-pagination>
  </section>
</template>

<script>
  import enums from '../../common/Enum'
  import DataUtil from '../../common/dataUtil'
  export default {
    data () {
      return {
        sex: enums.sex,
        searchForm:{
          uuid:'',
          realName:'',
          mobile:'',
          username:'',
          status:'',
          isAgain:'',
          channel: ''
        },
        userBankInf: {},
        bankInfseen: false,
        //订单状态
        qudaolist: [],
        orderStatus:enums.orderPro,
        isAgainOrder:enums.isAgainOrder,
        gridLoading: false,
        gridData: [],
        pageIndex: this.$store.getters.getOrderListAllIndex,
        pageSize: 10,
        dataTotal: 0
      }
    },
    methods: {
      getChannelName(type) {
         let re = '';
          this.qudaolist.forEach(obj => {
            if (obj.dicItemValue.split('#')[0] == type) {
              re = obj.dicItemName;
              return
            }
          });
          return re
      },
      getQudaoList() {
        this.$http.post('manage/dicItemListByDicCode',{dicCode:'REGISTER_CHANNEL'}).then(response=>{
          let {body} = response;
          if(body.code == 1){
            this.qudaolist = body.data || [];
          }
        },response=>{});
      },
      getOrderType(type){
          return enums.getOrderPro(type);
      },
      getUnixTime(time){
          return DataUtil.formatUnixTime(time);
      },
      getSex(val) {
        return enums.getSex(val)
      },
      search() {
        this.pageIndex = 1;
        this.bindGrid()
      },
      // check(row){
      //   window.open('#/OrderDetail?seen=true&userUuid='+row.userUuid+
      //   '&uuid='+row.uuid);
      // },
      rowClick(row){
        window.open('#/OrderDetail?seen=true&userUuid='+row.userUuid+
          '&uuid='+row.uuid);
      },
      getBankNo(row){
        this.gridLoading = true;
        let _data={"mobile":row.mobileNumber};
        this.$http.post('manage/userBankInfoByMobile', _data).then(response => {
          let {body} = response;
          this.gridLoading = false;
          if (1 == body.code) {
            if(response.body.data[0]){
              this.userBankInf=response.body.data[0];
              this.bankInfseen=true;
            }else{
              this.$message.error("银行卡未认证");
            }
          } else {
            this.$message.error(body.message);
          }
        }, response => {

        });
      },
      pageSizeChange(val) {
        this.pageSize = val;
        this.pageIndex = 1;
        this.bindGrid()
      },
      pageIndexChange(val) {
        this.pageIndex = val;
        this.$store.dispatch('setOrderListAllIndex',val)
        this.bindGrid()
      },
      getOrderStatus() {
        this.$http.post('dicItem/getDicItemsByDicCode.json', {dicCode:'ORDER_STATE'}).then(response => {
          if (1 == response.body.code) {
            this.orderStatus = response.body.result;
          } else {
            this.$message.error(response.body.message);
          }
        }, response => {
        });
      },




      bindGrid() {
        this.gridLoading = true;
        let _data = Object.assign({
          pageNo:this.pageIndex,
          pageSize:this.pageSize
        }, this.searchForm);
        this.$http.post('manage/orderList', _data).then(response => {
          if (1 == response.body.code) {
            this.gridLoading = false;
            if(response.body.data){
              this.gridData = response.body.data.data;
              this.dataTotal = response.body.data.recordsTotal;
            }else {
              this.gridData = [];
              this.dataTotal = 0;
            }
          } else {
            this.$message.error(response.body.message);
          }
        }, response => {
          this.gridLoading = false;
        });
      }
    },
    mounted: function () {
      this.bindGrid();
      this.getQudaoList();
      //this.getOrderStatus();

    }
  }
</script>
