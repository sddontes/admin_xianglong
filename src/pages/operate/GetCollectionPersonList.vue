<template>
  <section>
      <!--工具条-->
    <el-form :inline="true" :model="searchForm" class="toolbar">
      <el-form-item label="手机号">
        <el-input v-model="searchForm.mobile"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="bindGrid" type="info">查询</el-button>
      </el-form-item>
    </el-form>
     <!--列表-->
    <template>
      <el-table :data="gridData" highlight-current-row v-loading="gridLoading" class="grid">
        <el-table-column label="订单号" prop="uuid" ></el-table-column>
        <el-table-column label="申请时间" prop="createTime">
          <template scope="scope">
            {{formatTime(scope.row.createTime)}}
          </template>
        </el-table-column>
        <el-table-column label="催收人员" prop="remark"></el-table-column>
      </el-table>
    </template>
  </section>
</template>
<script>
  import DataUtil from '../../common/dataUtil'
  export default {
    data() {
      return {
        searchForm: {
          mobile: ''
        },
        gridData: [],
        gridLoading: false
      }
    },
    methods: {
      formatTime(time){
        return DataUtil.formatUnixTime(time);
      },
      bindGrid() {
        this.gridData = [];
        let _data = Object.assign({},this.searchForm);
        this.gridLoading = true;

        this.$http.post('manage/getOrderCollectionInfoByMobile', _data).then(response => {
          let {body} = response;
          this.gridLoading = false;
          if (1 == body.code) {
              this.gridData = response.body.data;
          } else {
            this.$message.error(body.message);
          }
        }, response => {

        });
      }
    },
    mounted() {

    }
  }
</script>
<style>
</style>