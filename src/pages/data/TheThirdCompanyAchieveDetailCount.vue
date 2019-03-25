<template>
  <section>
    <!--工具条-->
    <el-form :inline="true" :model="searchForm" class="toolbar">
      <el-form-item label="入催批次">
        <el-date-picker v-model="searchForm.startDate" type="date" :editable="false" :picker-options="pickerOptions1" placeholder="选择日期"></el-date-picker>
      </el-form-item>
      <el-form-item label="催收机构">
          <el-select v-model="searchForm.companyId" placeholder="请选择" filterable clearable @change="selectChange">
            <el-option v-for="item in thirdCompanyList" :label="item.companyName" :key="item.companyId" :value="item.companyId"></el-option>
          </el-select>
      </el-form-item>
      <el-form-item label="催收人员">
          <el-select v-model="searchForm.userId" placeholder="请选择" filterable clearable>
            <el-option v-for="item in collectionPresonList" :label="item.realName" :key="item.userId" :value="item.userId"></el-option>
          </el-select>
      </el-form-item>
      <el-form-item>
        <el-button @click="search" type="info">查询</el-button>
      </el-form-item>
      <el-form-item>
        <el-button @click="exportExl" type="info">导出</el-button>
      </el-form-item>
    </el-form>

    <!-- 设置目标完成率栏 -->
    <el-row :span="24" class="setBar">
      <el-col :span="5"><el-button type="success" @click="setTarget">设置时间节点目标完成率</el-button></el-col>
      <el-col :span="4"><el-button type="text">是否显示汇总栏</el-button>  <el-switch @change="switchchange" v-model="showtotalrow" on-color="#13ce66" off-color="#ff4949"></el-switch></el-col>
      <el-col :span="3"><el-button type="text">分案总量：{{dailyCaseTotal}}</el-button></el-col>
      <el-col :span="3"><el-button type="text">完成总量：{{dailyFinishCase}}</el-button></el-col>
      <el-col :span="4"><el-button type="text">实际完成率：{{dailyFinishCaseRate}}%</el-button></el-col>
      <el-col :span="4"><el-button type="text">目标剩余量：{{dailyunfinishCase}}</el-button></el-col>
    </el-row>

    <!--列表-->
    <template>
      <el-table :data="gridData" highlight-current-row v-loading="gridLoading" :row-class-name="tableRowClassName" >
        <el-table-column label="入催批次" prop="batchEntry" width="120px"></el-table-column>
        <el-table-column label="催收机构" prop="companyName"></el-table-column>
        <el-table-column label="催收人员" prop="realName"></el-table-column>
        <el-table-column label="催收阶段" prop="stage"></el-table-column>
        <el-table-column label="排名" prop="ranking"></el-table-column>
        <el-table-column label="分案量" prop="collectionCount"></el-table-column>
        <el-table-column label="回收量" prop="finishCount"></el-table-column>
        <el-table-column label="目标剩余量" prop="targetnotFinishCount"></el-table-column>
        <el-table-column label="实际回收率%" prop="actualFinisRate"></el-table-column>
        <el-table-column label="目标差值" prop="targetDifferenceRate"></el-table-column>
        <el-table-column label="9点回收率%" prop="targetFinisRate9"></el-table-column>
        <el-table-column label="12点回收率%" prop="targetFinisRate12"></el-table-column>
        <el-table-column label="14点回收率%" prop="targetFinisRate14"></el-table-column>
        <el-table-column label="16点回收率%" prop="targetFinisRate16"></el-table-column>
        <el-table-column label="18点回收率%" prop="targetFinisRate18"></el-table-column>
        <el-table-column label="21点回收率%" prop="targetFinisRate21"></el-table-column>
        <el-table-column label="24点回收率%" prop="targetFinisRate24"></el-table-column>
      </el-table>
    </template>

      <!-- 设置目标完成率弹框 -->
    <el-dialog title="设置时间节点目标完成率" v-model="commonRadioDigVisible" :close-on-click-modal="false" size="tiny">
      <el-form :model="commonRadioForm" ref="commonRadioForm">
         <el-row>
           <el-col v-for="item in commonRadioForm.ratelsit" :key="item.dicItemId" :span="10" :offset="1">
              <el-form-item>
                <el-input v-model="item.dicItemValue">
                  <el-button slot="prepend">{{item.dicItemName.slice(0,3)}}</el-button>
                  <el-button slot="append">%</el-button>
                </el-input>
              </el-form-item>
            </el-col>
          </el-row>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="" @click="commonRadioDigVisible = false">取 消</el-button>
        <el-button type="primary" @click.native.prevent="commonRadioFormSubmit" :loading="commonRadioFormLoading">确 定</el-button>
      </div>
    </el-dialog>
 
  </section>
</template>

<script>
  import DataUtil from '../../common/dataUtil'
    import Config from '../../common/config'
  export default {
    data () {
      let compareDate = (time, form, txt, type) => {
        let _cp1 = time.getTime() >= (Date.now()- 3600 * 1000 * 24)
        let _cp2 = false
        let _input = this[form][txt]
        if (_input) {
          let _time = Date.parse(new Date(_input))
          if ((type == "s" && time.getTime() > _time) || (type == "e" && time.getTime() < _time)) {
            _cp2 = true
          }
        }
        return _cp1 || _cp2
      }
      return {
        showtotalrow: true,
        dailyCaseTotal: '',
        dailyFinishCase: '',
        dailyFinishCaseRate: '',
        dailyunfinishCase: '',
        searchForm:{
          companyId:'',
          startDate:'',
          userId: ''
        },
        gridLoading: false,
        gridData: [],
        thirdCompanyList: [],
        collectionPresonList: [],
        commonRadioDigVisible: false,
        commonRadioForm: {
          ratelsit: []
        },
        commonRadioFormLoading: false,
        pickerOptions1: {
          disabledDate(time) {
            return compareDate(time, "searchForm", "endDate", "s")
          }
        },
        pickerOptions2: {
          disabledDate(time) {
            return compareDate(time, "searchForm", "startDate", "e")
          }
        }
      }
    },
    methods: {
      tableRowClassName(row, index) {
        if (row.batchEntry === "汇总") {
          return 'info-row';
        }
        return '';
      },
      selectChange(val) {     //根据催收机构获取对应催收人员
        this.getCollectionPersonList(val);
      },
      getCollectionList() {    //催收机构
          this.$http.post('manage/companyList',{}).then(response =>{
            let {body} = response;
            if(body.code == 1){
              this.thirdCompanyList = body.data;
            }else{
              this.thirdCompanyList = [];
            }
          },response=>{});
      },
      getCollectionPersonList(_companyId) {    //催收人员
          this.$http.post('manage/userList',{companyId:_companyId}).then(response =>{
            let {body} = response;
            if(body.code == 1){
              this.collectionPresonList = body.data;
            }else{
              this.collectionPresonList = [];
            }
          },response=>{});
      },

      switchchange(res) {   //是否显示汇总开关
        this.gridData = [];
        this.tableDataEdit(res);
      },

      tableDataEdit(res){       //table数据处理
        if(res == true){     //显示汇总行

            this.allData.dateList.forEach(obj => {
              obj.childrenList.forEach(e =>{
                  this.gridData.push(e);
              });
              obj.batchEntry = '汇总';
              this.gridData.push(obj);
            });

        }else{                    //不显示汇总行
      
          this.allData.dateList.forEach(obj => {
            obj.childrenList.forEach(e =>{
                this.gridData.push(e);
            });
          });
        }
      },
      bindGrid() {
        this.gridLoading = true;
        this.gridData = [];
        let _data = Object.assign({}, this.searchForm,{
          startDate:DataUtil.formatTime(this.searchForm.startDate)|| DataUtil.formatTime(new Date(Date.now()-3600 * 1000 * 24))  //默认昨天
        });
        this.$http.post('manage/collectionPerformanceStatisticsDetail', _data).then(response => {
          this.gridLoading = false;
          let {body} = response;
          if (1 == body.code) {
            this.allData = body.data;
    
            this.dailyCaseTotal = body.data.sumCollectionCount;
            this.dailyFinishCase = body.data.sumFinishCount;
            this.dailyFinishCaseRate = body.data.sumActualFinisRate;
            this.dailyunfinishCase = body.data.sumTargetnotFinishCount;

            this.tableDataEdit(true);
          } else {
            this.$message.error(body.message);
          }
        }).catch((e) =>{
          this.gridLoading = false;
        });
      },
      search() {
        this.bindGrid()
      },
      exportExl() {
        window.open(Config.host + "/manage/collectionPerformanceStatisticsDetailExcel?sessionId="+DataUtil.sid()+"&startDate="+DataUtil.formatTime(this.searchForm.startDate)+"&companyId="+this.searchForm.companyId+"&userId="+this.searchForm.userId);
      },
      setTarget() {
        this.commonRadioDigVisible = true; 
        this.getCompleteRate();
      },
      getCompleteRate() {
        this.$http.post('manage/setTargetCompletionRate',{"setTargetType":"time"}).then(response=>{
          let {body} = response;
          if(body.code == 1){
            this.commonRadioForm.ratelsit = body.data;
          }else{
            this.$message.error(body.message)
          }

        },response=>{});
      },
      commonRadioFormSubmit() {
        let submitobj = {
          tmpdicItemIds: [],
          tmpdicItemValues: []
        }

        for(let i=0;i<this.commonRadioForm.ratelsit.length;i++){
          if(this.commonRadioForm.ratelsit[i].dicItemValue == ""){
            this.$message.error('请输入目标完成率~');
            return;
          }
          submitobj.tmpdicItemIds.push(this.commonRadioForm.ratelsit[i].dicItemId);
          submitobj.tmpdicItemValues.push(this.commonRadioForm.ratelsit[i].dicItemValue);
        }

        let _data = {
          dicItemIds: submitobj.tmpdicItemIds.join(','),
          dicItemValues: submitobj.tmpdicItemValues.join(',')
        }
        this.commonRadioFormLoading = true;
        this.$http.post('manage/saveSetTargetCompletionRate',_data).then(response=>{
          this.commonRadioFormLoading = false;
          let {body} = response;
          if(body.code == 1){
            this.commonRadioDigVisible = false;
          }else{
            this.$message.error(body.message);
          }
        },response=>{this.commonRadioFormLoading = false;});
      }
    },
    mounted: function () {
      this.getCollectionList();
      // this.getCollectionPersonList();
      this.bindGrid();
    }
  }
</script>
<style>
  .setBar{
    padding: 0px 10px;
    line-height: 60px;
  }
  .el-table .info-row {
    background: #edf7ff;
  }
  .aaa{
    /*max-height: calc(100vh - 150px)*/
  }
</style>
