<template>
  <section>
      <!--工具条-->
    <el-form :inline="true" :model="searchForm" class="toolbar" label-position="right" label-width="70px">
      <el-row>
        <el-form-item label="订单编号">
          <el-input v-model="searchForm.uuid"></el-input>
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
        <el-form-item label=" ">
          <el-button @click="search" type="info" style="width: 170px">查询</el-button>
        </el-form-item>
      </el-row>
    </el-form>

    <template>
      <el-table :data="gridData" highlight-current-row v-loading="gridLoading" class="grid" @selection-change="handleSelectionChange">
        <el-table-column type="selection" width="55"></el-table-column>
        <el-table-column label="订单编号" prop="uuid"  width="180">
        </el-table-column>
        <el-table-column label="姓名" prop="realName"  width="80">
        </el-table-column>
    <!--     <el-table-column label="手机号" prop="mobileNumber"  width="130">
        </el-table-column> -->
        <el-table-column label="复借订单" prop="isAgain">
          <template scope="scope">
            <el-tag :type="scope.row.isAgain==1? 'success' : 'danger'" close-transition>{{scope.row.isAgain==1?'是':'否'}}</el-tag>
          </template>
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

    <el-form :inline="true" :model="distributeForm" class="toolbar" label-position="right" label-width="70px">
      <el-form-item label="催收人员" prop="status">
        <el-select v-model="distributeForm.thirdCompanyTag" placeholder="请选择" filterable clearable>
          <el-option v-for="item in thirdCompanyList" :label="item.name" :key="item.code" :value="item.code"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button @click="distribute" type="info" :loading="distributeLoading">分配</el-button>
      </el-form-item>
      <el-form-item v-if="this.dialogTitle == 'D1~D7'">
        <el-button @click="testAssginDistribute" type="info" :loading="autoDistributeLoading">自动测试订单分配</el-button>
      </el-form-item>
      <el-form-item>
        <el-button @click="autoDistribute" type="info" :loading="autoDistributeLoading">自动分配</el-button>
      </el-form-item>
    </el-form>
    <el-pagination class="pager" @size-change="pageSizeChange" @current-change="pageIndexChange" :current-page="pageIndex" :page-size="pageSize"
                   layout="total, sizes, prev, pager, next, jumper" :total="dataTotal">
    </el-pagination>
  

    <el-dialog :title="commonForm.title" v-model="editFormDigVisible" :close-on-click-modal="false">
      <div style="margin:0 0 15px;">已分配 <el-tag type="success">{{distributeCount}}</el-tag> 单，其中老户 <el-tag type="success">{{oldDistributeCount}}</el-tag> 单，新户 <el-tag type="success">{{newDistributeCount}}</el-tag> 单</div>
      <div style="margin:0 0 15px;">待分配 <el-tag type="danger">{{needDistributeCount}}</el-tag> 单，其中老户 <el-tag type="danger">{{oldNeedDistributeCount}}</el-tag> 单，新户 <el-tag type="danger">{{newNeedDistributeCount}}</el-tag> 单</div>
      <el-table :data="userListData" highlight-current-row v-loading="editGridLoading" class="grid" max-height="360">
        <el-table-column label="催收人员" prop="name" >
        </el-table-column>
        <el-table-column label="已分配老户订单数" prop="oldDistributeCount">
        </el-table-column>
        <el-table-column label="已分配新户订单数" prop="newDistributeCount">
        </el-table-column>
        <el-table-column label="新增分配老户订单数" prop="oldNeedDistributeCount" >
          <template scope="scope">
            <el-input v-model="scope.row.oldNeedDistributeCount"></el-input>
          </template>
        </el-table-column>
        <el-table-column label="新增分配新户订单数" prop="newNeedDistributeCount" >
          <template scope="scope">
            <el-input v-model="scope.row.newNeedDistributeCount"></el-input>
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

    <el-dialog title="测试订单分配" v-model="testAssignFormDigVisible" :close-on-click-modal="false">
      <div style="margin:0 0 15px;">已分配 <el-tag type="success">{{assignSum}}</el-tag> 单，待分配 <el-tag type="danger">{{unAssignSum}}</el-tag> 单</div>
      <el-table :data="userAssignList" highlight-current-row v-loading="testAssignGridLoading" class="grid" max-height="360">
        <el-table-column label="催收人员" prop="realName" >
        </el-table-column>
        <el-table-column label="已分配测试订单数" prop="assignCount">
        </el-table-column>
        <el-table-column label="未分配测试订单数" prop="unAssignCount" >
          <template scope="scope">
            <el-input v-model="scope.row.unAssignCount"></el-input>
          </template>
        </el-table-column>
      </el-table>
      <div slot="footer" class="dialog-footer">
        <el-button type="" @click="testAssignFormDigVisible = false">取 消</el-button>
        <el-button type="primary" @click.native.prevent="testAssignFormSubmit" :loading="testAssignconfirmLoading">确 定</el-button>
      </div>
    </el-dialog>
  </section>
</template>
<script>
  import enums from '../common/Enum'
  import DataUtil from '../common/dataUtil'
  export default {
    props: ['dialogTitle','postId','itemValue'],
    data() {
      return {
        searchForm: {
          uuid: '',
          orderTag: '',
          outsourceId: '',
          thirdDistribute: '',
          isAgain: '',
          isTestOrder: ''
        },
        orderTagList:enums.orderTagList,
        thirdCompanyList: [],
        allpersonList:[],
        gridData: [],
        gridLoading: false,
        pageIndex: 1,
        pageSize: 20,
        dataTotal: 0,
        distributeForm:{
          thirdCompanyTag:'',
        },
        distributeLoading: false,
        commonForm:{
          title: ''
        },
        isAgainOrder:enums.isAgainOrder,
        editFormDigVisible: false,
        confirmLoading: false,
        editForm: {
          code: ''
        },
        autoDistributeLoading: false,
        userListData: [],
        editGridLoading: false,
        selectData: [],
        distributeCount:0,
        oldDistributeCount: 0,
        newDistributeCount: 0,
        needDistributeCount: 0,
        oldNeedDistributeCount: 0,
        newNeedDistributeCount: 0,
        setTestFormDigVisible: false,
        setTestForm: {
          isTestOrder: '',
          orderNo: '',
          userUuid: ''
        },
        setTestconfirmLoading: false,

        assignSum:0,     //测试订单分配
        unAssignSum: 0,  
        userAssignList: [],
        testAssignFormDigVisible: false,
        testAssignGridLoading: false,
        testAssignconfirmLoading: false
      }
    },
    methods: {
      getCollecttionPerson() {
        this.$http.post('manage/collectionUserList',{}).then(response => {
          let {body} = response;
          if(body.code == 1){
            this.allpersonList = body.data;
          }else{
            this.allpersonList = [];
          }
        },response => {});
      },
      getCollectionList() {
        this.$http.post('manage/innerOutCollectionUserList',{postId:this.postId}).then(response => {
          let {body} = response;
          if(body.code == 1){
            this.thirdCompanyList = body.data;
          }else{
            this.thirdCompanyList = [];
          }
        },response => {});
      },
      handleSelectionChange(val){
        this.selectData = val;
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
      getOrderTag(type){
        return enums.getOrderTagList(type);
      },
      getThirdCompany(type){
       let re = '';
        this.allpersonList.forEach(obj => {
          if (obj.code == type) {
            re = obj.name;
            return
          }
        });
        return re
      },
      check(row){
        window.open('#/OrderDetail?seen=true&userUuid='+row.userUuid+
          '&uuid='+row.uuid);
      },
      test(row){
        this.setTestFormDigVisible = true;
        this.setTestForm.isTestOrder = row.isTestOrder;
        this.setTestForm.orderNo = row.uuid;
        this.setTestForm.userUuid = row.userUuid;
      },
      search() {
        this.pageIndex = 1;
        this.bindGrid()
      },
      bindGrid() {
        this.gridLoading = true;
        let _data = Object.assign({
          pageNo:this.pageIndex,
          pageSize:this.pageSize,
          overdueDayMin: this.itemValue.split('#')[0],
          overdueDayMax: this.itemValue.split('#')[1]
        }, this.searchForm);
        this.$http.post('manage/overDueOrderListByTag', _data).then(response => {
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
      distribute() {
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
        });
      },
      autoDistribute(row) {              //自动分配
        this.editFormDigVisible = true;
        this.commonForm.title = '订单分配（'+this.dialogTitle+'）';
        this.editGridLoading = true;
        this.$http.post('manage/overDueOrderDistributeCountListByPostId',{postId:this.postId,section:this.itemValue}).then(response => {
          this.editGridLoading = false;
          let {body} = response;
          if(body.code == 1){
            this.userListData = [];
            this.distributeCount = body.data.distributeCount;
            this.oldDistributeCount = body.data.oldDistributeCount;
            this.newDistributeCount = body.data.newDistributeCount;
            this.needDistributeCount = body.data.needDistributeCount;
            this.oldNeedDistributeCount = body.data.oldNeedDistributeCount;
            this.newNeedDistributeCount = body.data.newNeedDistributeCount;
            this.userListData = body.data.result;
          }else{
            this.$message.error(body.message);
          }
        },response =>{});
      },
      editFormSubmit() {
        let _oldtotalNum = 0,_newtotalNum=0;
        for(let i=0;i<this.userListData.length;i++){
          if(this.userListData[i].oldNeedDistributeCount === '' || this.userListData[i].newNeedDistributeCount === ''){
            this.$message.error('所有分配订单数不能为空');
            return;
          }else{
            this.userListData[i].orderCount = this.userListData[i].oldNeedDistributeCount;
            this.userListData[i].newOrderCount = this.userListData[i].newNeedDistributeCount;
            _oldtotalNum += parseInt(this.userListData[i].oldNeedDistributeCount);
            _newtotalNum += parseInt(this.userListData[i].newNeedDistributeCount);
          }
        }
        if(_oldtotalNum > this.oldNeedDistributeCount || _newtotalNum > this.newNeedDistributeCount){
          this.$message.error('剩余待分配订单数不足，请重新分配');
          return;
        }

        let _data = {
          section: this.itemValue,
          staff:this.userListData
        };
        this.confirmLoading = true;
        this.$http.post('manage/overDueOrderAutoDistributeByNum',_data).then(response =>{
          this.confirmLoading = false;
          let {body} = response;
          if(body.code == 1){
            this.editFormDigVisible = false;
            this.$alert('分单完成，剩余未分配订单：老户'+body.data.oldNeedDistributeCount+'单，新户'+body.data.newNeedDistributeCount+'单', '分单成功', {
              confirmButtonText: '确定',
              callback: action => {
                this.$message({
                  type: 'success',
                  message:'分单成功'
                });
              }
            });
            this.bindGrid();
          }else{
            this.$message.error(body.message);
          }
        },response =>{this.confirmLoading = false;});
      },
     setTestFormSubmit(){                   //设置测试订单
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
      },
      testAssginDistribute(row) {                    //自动测试订单分配
        this.testAssignFormDigVisible = true;
        this.testAssignGridLoading = true;
        this.$http.post('manage/findTestOrderD1t7AssignCount',{}).then(response => {
          this.testAssignGridLoading = false;
          let {body} = response;
          if(body.code == 1){
            this.userAssignList = [];
            this.assignSum = body.data.orderD1t7Assign.assignSum;
            this.unAssignSum = body.data.orderD1t7Assign.unAssignSum;

            this.userAssignList = body.data.userAssignList;
          }else{
            this.$message.error(body.message);
          }
        },response =>{});
      },
      testAssignFormSubmit() {
        let _newtotalNum=0;
        for(let i=0;i<this.userAssignList.length;i++){
          if(this.userAssignList[i].unAssignCount === ''){
            this.$message.error('所有分配订单数不能为空');
            return;
          }else{
            _newtotalNum += parseInt(this.userAssignList[i].unAssignCount);
          }
        }
        if(_newtotalNum > this.unAssignSum){
          this.$message.error('剩余待分配订单数不足，请重新分配');
          return;
        }

        let _data = {
          userAssignList:this.userAssignList
        };
        this.testAssignconfirmLoading = true;
        this.$http.post('manage/testOrderD1t7AssignCountUpdateBatch',_data).then(response =>{
          this.testAssignconfirmLoading = false;
          let {body} = response;
          if(body.code == 1){
            this.testAssignFormDigVisible = false;
            this.$message({
              type: 'success',
              message: '分配成功'
            });
            this.bindGrid();
          }else{
            this.$message.error(body.message);
          }
        },response =>{this.testAssignconfirmLoading = false;});
      },
    },
    mounted() {
      this.getCollecttionPerson();
      this.getCollectionList();
      this.bindGrid();
    }
  }
</script>
<style></style>
