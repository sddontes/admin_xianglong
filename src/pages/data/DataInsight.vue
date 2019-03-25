/**
* Created by alan on 17-5-11.
*/
<template>
  <section>
    <div style="height: 20px;"></div>
    <el-row>
      <el-col :span="7" :offset="1">
        <el-card>
          <div class="dataContet">
            <div class="text-center">
              <span class="bigText">累积放款金额</span>（万元）
              <p class="bigAmount">{{parseFloat(amountTotal.sumLoanAmount || '0.00').toFixed(2)}}</p>
            </div>
          </div>
          <div style="padding: 14px;">
            <el-button type="text" class="button" @click="searchDetailData">查看详情>></el-button>
          </div>
        </el-card>
      </el-col>
      <el-col :span="7" :offset="1">
        <el-card>
          <div class="dataContet">
            <div class="text-center">
              <span class="bigText">累积放款订单</span>（笔）
              <p class="bigAmount">{{amountTotal.loanOrderCount || '0.00'}}</p>
            </div>
          </div>
          <div style="padding: 14px;">
            <!--<el-button type="text" class="button">查看详情>></el-button>-->
          </div>
        </el-card>
      </el-col>
      <el-col :span="7" :offset="1">
        <el-card>
          <div class="dataContet">
            <div class="text-center">
              <span class="bigText">在贷余额</span>（万元）
              <p class="bigAmount">{{parseFloat(amountTotal.loaningAmount || '0.00').toFixed(2)}}</p>
            </div>
          </div>
          <div style="padding: 14px;">
            <el-button type="text" class="button">查看详情>></el-button>
          </div>
        </el-card>
      </el-col>
    </el-row>
    <div style="height: 20px;"></div>
    <el-row>
      <el-tabs type="border-card">
        <el-tab-pane label="放款">
          <el-table :data="loanDaily" highlight-current-row border stripe>
            <el-table-column prop="sumLoanAmount" label="今日放款金额（万元）"></el-table-column>
            <el-table-column prop="loanOrderCount" label="今日放款订单数（笔）"></el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="还款">
          <el-table :data="repaymentDaily" highlight-current-row border stripe>
            <el-table-column prop="sumLoanAmount" label="今日还款金额（万元）"></el-table-column>
            <el-table-column prop="loanOrderCount" label="今日还款订单数（笔）"></el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="逾期">
          <el-table :data="overDueDaily" highlight-current-row border stripe>
            <el-table-column prop="sumLoanAmount" label="今日逾期金额（万元）"></el-table-column>
            <el-table-column prop="loanOrderCount" label="今日逾期订单数（笔）"></el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-row>
    <div style="height: 20px;"></div>
    <el-row>
      <el-col :span="12">
        <div id="dailyLoanChartLine" style="width:100%; height:400px;"></div>
      </el-col>
      <el-col :span="12">
        <div id="dailyOrderchartLine" style="width:100%; height:400px;"></div>
      </el-col>
    </el-row>
  </section>
</template>

<script>
  import echarts from 'echarts'
  export default{
    data(){
      return {
        amountTotal: {},
        loanDaily: [],
        repaymentDaily: [],
        overDueDaily: [],
        chartLine: {},
        weeklyLoanAmount: []   //一周放款统计
      }
    },
    methods: {
      searchDetailData() {   //查看放款数据详情
        window.open('#/DetailLoanData');
      },
      getLoanData(){
        this.$http.post('manage/loanDataAmountCount', {}).then(response => {
          let {body} = response;
          if (1 == body.code && body.data) {
            this.amountTotal = body.data;
          }
        });
        /*查询今日放款数据*/
        this.$http.post('manage/dailyLoanAmount', {}).then(response => {
          let {body} = response;
          if (1 == body.code && body.data) {
            let arr = [];
            arr.push(body.data);
            this.loanDaily = arr;
          }
        });
        /*查询今日还款数据*/
        this.$http.post('manage/repaymentDailyAmount', {}).then(response => {
          let {body} = response;
          if (1 == body.code && body.data) {
            let arr = [];
            arr.push(body.data);
            this.repaymentDaily = arr;
          }
        });
        this.$http.post('manage/overdueDailyAmount', {}).then(response => {
          let {body} = response;
          if (1 == body.code && body.data) {
            let arr = [];
            arr.push(body.data);
            this.overDueDaily = arr;
          }
        });
      },
      drawLineChart(){
        this.dailyLoanChartLine = echarts.init(document.getElementById('dailyLoanChartLine'));
        this.dailyOrderchartLine = echarts.init(document.getElementById('dailyOrderchartLine'));
        /*一周数据统计*/
        this.dailyLoanChartLine.showLoading();
        this.dailyOrderchartLine.showLoading();
        this.$http.post('manage/weeklyLoanAmount', {}).then(response => {
          let {body} = response;
          this.dailyLoanChartLine.hideLoading();
          this.dailyOrderchartLine.hideLoading();
          if (1 == body.code && body.data) {
            let result = body.data;
            let weeklyLoanAmountList = [];
            let weeklyLoanCountList = [];
            let weeklyDate = this.get7DaysBefore();
            for (let i = result.length - 1; i >= 0; i--) {
              weeklyLoanAmountList.push(result[i].sumLoanAmount);
              weeklyLoanCountList.push(result[i].loanOrderCount);
            }

            this.dailyLoanChartLine.setOption({
              title: {
                text: '每日放款金额(万元)'
              },
              tooltip: {
                trigger: 'axis'
              },
              legend: {
                data: ['每日放款金额(万元)']
              },
              grid: {
                left: '3%',
                right: '4%',
                bottom: '3%',
                containLabel: true
              },
              xAxis: {
                type: 'category',
                boundaryGap: false,
                data: weeklyDate
              },
              yAxis: {
                type: 'value'
              },
              series: [
                {
                  name: '每日放款金额(万元)',
                  type: 'line',
                  stack: '总量',
                  data: weeklyLoanAmountList
                }
              ]
            });

            this.dailyOrderchartLine.setOption({
              title: {
                text: '每日放款订单数'
              },
              tooltip: {
                trigger: 'axis'
              },
              legend: {
                data: ['每日放款订单数']
              },
              grid: {
                left: '3%',
                right: '4%',
                bottom: '3%',
                containLabel: true
              },
              xAxis: {
                type: 'category',
                boundaryGap: false,
                data: weeklyDate
              },
              yAxis: {
                type: 'value'
              },
              series: [
                {
                  name: '每日放款订单数',
                  type: 'line',
                  stack: '总量',
                  data: weeklyLoanCountList
                }
              ]
            });
          }
        });

      },
      get7DaysBefore(){
        let myDate = new Date(); //获取今天日期
        myDate.setDate(myDate.getDate() - 6);
        let dateArray = [];
        let dateTemp;
        let flag = 1;
        for (let i = 0; i <= 6; i++) {
          dateTemp = (myDate.getMonth()+1)+"-"+myDate.getDate();
          dateArray.push(dateTemp);
          myDate.setDate(myDate.getDate() + flag);
        }
        return dateArray;
      }

  },
  mounted()
  {
    this.getLoanData();
    this.drawLineChart();
  }
  }
</script>
<style scoped>
  .button {
    padding: 0;
    float: right;
  }

  .dataContet {
    height: 180px;
  }

  .text-center {
    text-align: center;
  }

  .bigText {
    font-size: 20px;
    font-weight: bolder;
  }

  .bigAmount {
    font-size: 40px;
    font-weight: bolder;
    padding-top: 39px;
  }
</style>
