/**
* Created by alan on 17-6-23.
*/
<template>
  <section>
    <!--工具条-->
    <el-form :inline="true" :model="searchForm" class="toolbar" label-position="right" label-width="70px">
      <el-row>
        <el-form-item label="订单编号">
          <el-input v-model="searchForm.uuid" width="200"></el-input>
        </el-form-item>
        <el-form-item label="订单标签" prop="status">
          <el-select v-model="searchForm.orderTag" placeholder="请选择" clearable>
            <el-option v-for="item in orderTagList" :label="item.name" :key="item.code" :value="item.code"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="催收人员" prop="outsourceId">
          <el-select v-model="searchForm.outsourceId" placeholder="请选择" filterable clearable>
            <el-option v-for="item in thirdCompanyList" :label="item.name" :key="item.code" :value="item.code"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="是否被分配" prop="" label-width="90">
          <el-select v-model="searchForm.thirdDistribute" placeholder="请选择" clearable>
            <el-option  label="是" key="" value="1"></el-option>
            <el-option  label="否" key="" value="0"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="复借订单">
          <el-select v-model="searchForm.isAgain" placeholder="请选择" clearable>
            <el-option v-for="item in isAgainOrder" :label="item.name" :key="item.code" :value="item.code"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="测试订单" prop="" label-width="90">
          <el-select v-model="searchForm.isTestOrder" placeholder="请选择" clearable>
            <el-option  label="是" key="" value="1"></el-option>
            <el-option  label="否" key="" value="0"></el-option>
          </el-select>
        </el-form-item>
         <el-form-item label="复借次数区间" label-width="100px">
          <el-col :span="11"><el-input type="number" v-model="searchForm.repeTitionsMin" placeholder="复借次数下限"></el-input></el-col>
          <el-col :span="2" class="line">~</el-col>
          <el-col :span="11"><el-input type="number" v-model="searchForm.repeTitionsMax" placeholder="复借次数上限"></el-input></el-col>
        </el-form-item>
        <el-form-item>
          <el-button @click="search" type="info" style="width: 170px">查询</el-button>
        </el-form-item>
      </el-row>
    </el-form>
    <!--列表-->
    <template>
      <el-table :data="gridData" highlight-current-row v-loading="gridLoading" class="grid" @selection-change="handleSelectionChange">
        <el-table-column type="selection" width="55"></el-table-column>
        <el-table-column label="订单编号" prop="uuid" width="180">
        </el-table-column>
        <el-table-column label="姓名" prop="realName" width="80">
        </el-table-column>
        <el-table-column label="复借订单" prop="isAgain">
          <template scope="scope">
            <el-tag :type="scope.row.isAgain==1? 'success' : 'danger'" close-transition>{{scope.row.isAgain==1?'是':'否'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="复借次数" prop="repeTitions">
        </el-table-column>
        <el-table-column label="申请金额" prop="amountApply">
        </el-table-column>
        <el-table-column label="申请期限" prop="borrowingTerm">
        </el-table-column>
        <el-table-column label="订单标签" prop="orderTag" min-width="116">
          <template scope="scope">
            <span>{{getOrderTag(scope.row.orderTag)}}</span>
          </template>
        </el-table-column>
        <el-table-column label="测试订单">
          <template scope="scope">
            <el-tag :type="scope.row.isTestOrder==1? 'success' : 'danger'" close-transition>{{scope.row.isTestOrder==1?'是':'否'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="催收人员" prop="outsourceId" min-width="116">
          <template scope="scope">
            <span>{{getThirdCompany(scope.row.outsourceId)}}</span>
          </template>
        </el-table-column>
        <el-table-column label="逾期天数" prop="overDueDay">
        </el-table-column>
        <el-table-column label="操作" min-width="140">
          <template scope="scope">
            <el-button size="small" @click="check(scope.row)">详情</el-button>
            <el-button size="small" @click="test(scope.row)">测试</el-button>
          </template>
        </el-table-column>
      </el-table>
    </template>

    <!--分页-->
    <el-pagination class="pager" @size-change="pageSizeChange" @current-change="pageIndexChange" :current-page="pageIndex" :page-size="pageSize"
                   layout="total, sizes, prev, pager, next, jumper" :total="dataTotal">
    </el-pagination>

    <el-form :inline="true" :model="distributeForm" class="toolbar" label-position="right" label-width="70px">
      <el-form-item label="催收人员" prop="status">
        <el-select v-model="distributeForm.thirdCompanyTag" placeholder="请选择" filterable clearable>
          <el-option v-for="item in thirdCompanyList" :label="item.name" :key="item.code" :value="item.code"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button @click="distribute" type="info" v-loading="distributeLoading">分配</el-button>
      </el-form-item>
      <el-form-item>
        <el-button @click="autoDistribute" type="info" :loading="autoDistributeLoading">自动分配</el-button>
      </el-form-item>
    </el-form>

    <el-dialog title="自动分配（D0）" v-model="editFormDigVisible" :close-on-click-modal="false" size="large">
       <div style="margin:0 0 15px;">已分配 <el-tag type="success">{{orderD0Assign.assignSum}}</el-tag> 单，其中新户 <el-tag type="success">{{orderD0Assign.assign0Count}}</el-tag> 单，复借1次 <el-tag type="success">{{orderD0Assign.assign1Count}}</el-tag> 单，复借2次 <el-tag type="success">{{orderD0Assign.assign2Count}}</el-tag> 单，复借3次 <el-tag type="success">{{orderD0Assign.assign3Count}}</el-tag> 单，复借3次以上 <el-tag type="success">{{orderD0Assign.assign4Count}}</el-tag> 单</div>
      <div style="margin:0 0 15px;">待分配 <el-tag type="danger">{{orderD0Assign.unAssignSum}}</el-tag> 单，其中新户 <el-tag type="danger">{{orderD0Assign.unAssign0Count}}</el-tag> 单，复借1次 <el-tag type="danger">{{orderD0Assign.unAssign1Count}}</el-tag> 单，复借2次 <el-tag type="danger">{{orderD0Assign.unAssign2Count}}</el-tag> 单，复借3次 <el-tag type="danger">{{orderD0Assign.unAssign3Count}}</el-tag> 单，复借3次以上 <el-tag type="danger">{{orderD0Assign.unAssign4Count}}</el-tag> 单</div>
      <el-table :data="userListData" highlight-current-row v-loading="editGridLoading" class="grid" max-height="360">
        <el-table-column label="催收人员" prop="realName" ></el-table-column>
        <el-table-column label="已分配新户订单数" prop="assign0Count"></el-table-column>
        <el-table-column label="已分配复借1次订单数" prop="assign1Count"></el-table-column>
        <el-table-column label="已分配复借2次订单数" prop="assign2Count"></el-table-column>
        <el-table-column label="已分配复借3次订单数" prop="assign3Count"></el-table-column>
        <el-table-column label="已分配复借3次以上订单数" prop="assign4Count"></el-table-column>
        <el-table-column label="新增分配新户订单数">
          <template scope="scope">
            <el-input v-model="scope.row.unAssign0Count"></el-input>
          </template>
        </el-table-column>
        <el-table-column label="新增分配复借1次订单数">
          <template scope="scope">
            <el-input v-model="scope.row.unAssign1Count"></el-input>
          </template>
        </el-table-column>
        <el-table-column label="新增分配复借2次订单数">
          <template scope="scope">
            <el-input v-model="scope.row.unAssign2Count"></el-input>
          </template>
        </el-table-column>
        <el-table-column label="新增分配复借3次订单数">
          <template scope="scope">
            <el-input v-model="scope.row.unAssign3Count"></el-input>
          </template>
        </el-table-column>
        <el-table-column label="新增分配复借3次以上订单数">
          <template scope="scope">
            <el-input v-model="scope.row.unAssign4Count"></el-input>
          </template>
        </el-table-column>
      </el-table>
      <div slot="footer" class="dialog-footer">
        <el-button type="" @click="editFormDigVisible = false">取 消</el-button>
        <el-button type="primary" @click.native.prevent="editFormSubmit" :loading="confirmLoading">确 定</el-button>
      </div>
    </el-dialog>

    <el-dialog title="测试订单设置" v-model="setTestFormDigVisible" :close-on-click-modal="false" size="tiny">
      <div>
        <el-form :model="setTestForm" label-position="left" label-width="0px">
          <el-row>
            <el-col :span="16" :offset="4">
            <el-form-item>设置为测试订单</el-form-item>
              <el-form-item>
                <el-radio-group v-model="setTestForm.isTestOrder">
                  <el-radio :label="1">是</el-radio>
                  <el-radio :label="0">否</el-radio>
                </el-radio-group>
              </el-form-item>
            </el-col>
          </el-row>
        </el-form>
      </div>
      <div slot="footer" class="dialog-footer">
        <el-button type="" @click="setTestFormDigVisible = false">取 消</el-button>
        <el-button type="primary" @click.native.prevent="setTestFormSubmit" :loading="setTestconfirmLoading">确 定</el-button>
      </div>
    </el-dialog>

  </section>
</template>

<script>
  import enums from '../../common/Enum'
  import DataUtil from '../../common/dataUtil'
  export default {
    data () {
      return {
        searchForm:{
          mobile:'',
          orderTag:'',
          realName: '',
          username: '',
          uuid: '',
          outsourceId: '',
          thirdDistribute: '',
          isAgain:'',
          repeTitionsMin:"",
          repeTitionsMax:"",
          isTestOrder: ""
        },
        distributeForm:{
          thirdCompanyTag:'',
        },
        distributeLoading:false,
        //订单状态
        isAgainOrder:enums.isAgainOrder,
        orderStatus:enums.orderPro,
        orderTagList:enums.orderTagList,
        thirdCompanyList:[],
        distributeData:[],
        selectData:{},
        gridLoading: false,
        gridData: [],
        pageIndex: 1,
        pageSize: 20,
        dataTotal: 0,
        editFormDigVisible: false,
        userListData: [],
        autoDistributeLoading: false,

        orderD0Assign:{
          assignSum: 0,
          assign0Count: 0,
          assign1Count: 0,
          assign2Count: 0,
          assign3Count: 0,
          assign4Count: 0,
          unAssignSum: 0,
          unAssign0Count: 0,
          unAssign1Count: 0,
          unAssign2Count: 0,
          unAssign3Count: 0,
          unAssign4Count: 0
        },
        editGridLoading: false,
        confirmLoading: false,
        postId:0,

        //设置测试订单
       setTestFormDigVisible: false,
       setTestForm: {
        isTestOrder: '',
        orderNo: '',
        userUuid: ''
       },
       setTestconfirmLoading: false
      }
    },
    methods: {
      getCollectionList() {       //催收岗位
        this.$http.post('manage/dicItemListByDicCode',{dicCode:'COLLECTION_POST'}).then(response => {
          let {body} = response;
          this.postId = body.data[0].id;
          this.$http.post('manage/searchCollectionUserList',{postId:this.postId}).then(response =>{
            let {body} = response;
            if(body.code == 1){
              this.thirdCompanyList = body.data;
            }else{
              this.thirdCompanyList = [];
            }
          },response=>{});
        },response =>{});
      },
      getOrderTag(type){       //列表订单标签
        return enums.getOrderTagList(type);
      },
      getThirdCompany(type){    //列表催收人员
         let re = '';
          this.thirdCompanyList.forEach(obj => {
            if (obj.code == type) {
              re = obj.name;
              return
            }
          });
          return re
      },
      search() {     //查询
        this.pageIndex = 1;
        if(this.searchForm.repeTitionsMin && !DataUtil.isInteger(this.searchForm.repeTitionsMin)){
          this.$message.error('请输入正整数格式');
          return;
        }
        if(this.searchForm.repeTitionsMax && !DataUtil.isInteger(this.searchForm.repeTitionsMax)){
          this.$message.error('请输入正整数格式');
          return;
        }
        if(this.searchForm.repeTitionsMin&&this.searchForm.repeTitionsMax && parseInt(this.searchForm.repeTitionsMin) > parseInt(this.searchForm.repeTitionsMax)){
          this.$message.error('复借次数上限应大于等于复借次数下限，请重新输入');
          this.searchForm.repeTitionsMin = '';
          this.searchForm.repeTitionsMax = '';
          return;
        }
        this.bindGrid()
      },
      check(row){    //进详情页
        window.open('#/OrderDetail?seen=true&userUuid='+row.userUuid+
          '&uuid='+row.uuid);
      },
      handleSelectionChange(val){
          this.selectData = val;
      },
      distribute(){              //手动分配
        if(!this.distributeForm.thirdCompanyTag){
            this.$message.error("请选择催收人员");
            return false;
        }
        let result = [];
        this.selectData.forEach((data) =>{
          let temp = {};
          temp.uuid = data.uuid;
          temp.uuidString = data.userUuid;
          temp.outsourceId = this.distributeForm.thirdCompanyTag;
          result.push(temp);
        });
        if(result.length <= 0){
          this.$message.error("请选择催收订单");
          return false;
        }
        this.distributeLoading = true;
        this.$http.post('manage/distributeOrder2ThirdCompany',result).then(response =>{
            this.distributeLoading = false;
            let {body} = response;
            if(1 == body.code){
              this.bindGrid();
            }
        })
      },
      autoDistribute(row) {      //自动分配返显
        this.editFormDigVisible = true;
        this.editGridLoading = true;
        this.$http.post('manage/findOrderD0AssignCountList',{}).then(response => {
          this.editGridLoading = false;
          let {body} = response;
          if(body.code == 1){
            this.userListData = [];
            this.orderD0Assign = body.data.orderD0Assign;
            this.userListData = body.data.userAssignList;
          }else{
            this.$message.error(body.message);
          }
        },response =>{});
      },
      editFormSubmit() {       //自动分配数据提交
        let _unAssign0Count = 0,_unAssign1Count = 0,_unAssign2Count = 0,_unAssign3Count = 0,_unAssign4Count = 0;

        for(let i=0;i<this.userListData.length;i++){
          if(this.userListData[i].unAssign0Count === '' || this.userListData[i].unAssign1Count === ''|| this.userListData[i].unAssign2Count === ''|| this.userListData[i].unAssign3Count === ''|| this.userListData[i].unAssign4Count === ''){
            this.$message.error('所有分配订单数不能为空');
            return;
          }else{
            _unAssign0Count += parseInt(this.userListData[i].unAssign0Count);   //计算每列单数总和
            _unAssign1Count += parseInt(this.userListData[i].unAssign1Count);
            _unAssign2Count += parseInt(this.userListData[i].unAssign2Count);
            _unAssign3Count += parseInt(this.userListData[i].unAssign3Count);
            _unAssign4Count += parseInt(this.userListData[i].unAssign4Count);
          }
        }
        console.log(this.userListData);
        if(_unAssign0Count > this.orderD0Assign.unAssign0Count || _unAssign1Count > this.orderD0Assign.unAssign1Count || _unAssign2Count > this.orderD0Assign.unAssign2Count || _unAssign3Count > this.orderD0Assign.unAssign3Count || _unAssign4Count > this.orderD0Assign.unAssign4Count){
          this.$message.error('剩余待分配订单数不足，请重新分配');
          return;
        }
        let _data = {
          userAssignList:this.userListData
        };
        this.confirmLoading = true;
        this.$http.post('manage/orderD0unAssignCountListUpdateBatch',_data).then(response =>{
          this.confirmLoading = false;
          let {body} = response;
          if(body.code == 1){
            this.$message({
              type: 'success',
              message: '分配成功'
            });
            this.editFormDigVisible = false;
            this.bindGrid();
          }else{
            this.$message.error(body.message);
          }
        },response =>{this.confirmLoading = false;});
      },
      pageSizeChange(val) {
        this.pageSize = val;
        this.pageIndex = 1;
        this.bindGrid()
      },
      pageIndexChange(val) {
        this.pageIndex = val;
        this.bindGrid()
      },
      bindGrid() {           //列表
        this.gridLoading = true;
        let _data = Object.assign({
          pageNo:this.pageIndex,
          pageSize:this.pageSize
        }, this.searchForm);
        this.$http.post('manage/willOverDueOrderByTag', _data).then(response => {
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
      },
      test(row){
        this.setTestFormDigVisible = true;
        this.setTestForm.isTestOrder = row.isTestOrder;
        this.setTestForm.orderNo = row.uuid;
        this.setTestForm.userUuid = row.userUuid;
      },
      setTestFormSubmit(){
        let _data = Object.assign({},this.setTestForm);
        this.setTestconfirmLoading = true;

        this.$http.post('manage/setTestOrder',_data).then(response=>{
          this.setTestconfirmLoading = false;
          let {body} = response;
          if(body.code == 1){
            this.$message({
              type: 'success',
              message: '操作成功'
            });
            this.setTestFormDigVisible = false;
            this.bindGrid();
          }else{
            this.$message.error(body.message);
          }
        },response=>{this.setTestconfirmLoading = false;});
      }
    },
    mounted() {
      this.bindGrid();
      this.getCollectionList();
    }
  }
</script>
<style scoped>
  .line{
    text-align: center;
  }
</style>
