<template>
  <el-row>
    <el-col :span="24" class="warp-breadcrum">
      <el-breadcrumb separator=">" class="navCss">
          <el-breadcrumb-item>总工时统计表</el-breadcrumb-item>
      </el-breadcrumb>
    </el-col>

    <el-card class="box-card">
      <el-col :span="24" class="btnBox">
        <el-date-picker
            v-model="searchData.time"
            type="month"
            placeholder="选择月">
        </el-date-picker>
        <el-button type="primary" @click="getWorkLogList(searchData)">查询</el-button>
      </el-col>
      <el-col :span="24" class="tableBox">
        <el-table
          :data="workLogHashList"
          border
          style="width: 100%">
          <el-table-column
            prop="member"
            fixed
            label="人员"
            width="70">
          </el-table-column>
          <el-table-column
            prop="type"
            fixed
            label="类型"
            width="70">
          </el-table-column>
          <el-table-column
            prop=""
            fixed
            label="年月"
            width="100">
            <template slot-scope="scope">
              <span>{{scope.row.dateRange|dateFormat('yyyy-MM')}}</span>
            </template>
          </el-table-column>
          <el-table-column v-for="item in departmentList" :key="item.departmentName" :label="item.departmentName">
            <el-table-column
              v-for="items in item.childProjects"
              :key="items.projectFirstName"
              :label="items.projectFirstName">
                <el-table-column
                  v-for="(itemss,index) in items.childProjects"
                  prop=""
                  :key="itemss.projectSecondName"
                  :label="itemss.projectSecondName">
                  <template slot-scope="scope">
                    <span 
                      v-show="checkName(itemss.projectSecondName,scope.row.projectInfoList)!=-1"
                      >{{scope.row.projectInfoList[checkName(itemss.projectSecondName,scope.row.projectInfoList)]&&scope.row.projectInfoList[checkName(itemss.projectSecondName,scope.row.projectInfoList)].workDays}}</span>
                  </template>
              </el-table-column>
            </el-table-column>
          </el-table-column>
          <el-table-column
            prop="totalTime"
            fixed="right"
            label="个人工时总计"
            width="50">
          </el-table-column>
          <el-table-column
            prop="remarks"
            fixed="right"
            label="任务说明"
            width="50">
          </el-table-column>
        </el-table>
      </el-col>
    </el-card>
  </el-row>
</template>

<script>

  export default {
    data() {
      return {
        departmentList:[],
        workLogList:[],
        workLogHashList:[],
        searchData:{
          time:'',
        },
      }
    },
    methods: {
      check(arr,item){
        let i = 0;
        let len = arr.length;
        for(i;i<len;i++){
          if(arr[i].memberId==item.memberId&&arr[i].dateRange==item.dateRange){
            return i;
          }
        }
        return -1;
      },
      checkName(name,list){
        for(var i=0;i<list.length;i++){
          if(list[i].projectInfo==name){
            return i;
          }
        }
        return -1;
      },
      getDepartmentList(){
        this.$http.get('/api/department/departmentRelationList').then((data) => {
          let datas=data.data.data;
          this.departmentList=datas;
    	  });
      },
      getWorkLogList(searchData){
        let para = Object.assign({}, this.searchData);
        let options = {
            params: para
        };
        this.$http.get('/api/worklog/getWorkLog',options).then((data) => {
          let datas=data.data.data;
          this.workLogList=datas;
          this.workLogHashList=[];
          let that=this;
          this.workLogList.forEach(item=>{
            console.log(this.workLogHashList,item)
            let index=this.check(this.workLogHashList,item);
            console.log(index);
          	if(index!=-1){
          		this.workLogHashList[index].projectInfoList.push({
          			projectInfo:item.projectInfo,
          			workDays:item.workDays,
              })
              this.workLogHashList[this.workLogHashList.length-1].totalTime+=item.workDays;
              this.workLogHashList[this.workLogHashList.length-1].remarks+=item.remarks;
          	}else{
          		this.workLogHashList.push(item);
          		this.workLogHashList[this.workLogHashList.length-1].projectInfoList=[{
                projectInfo:item.projectInfo,
          			workDays:item.workDays,
              }];
              this.workLogHashList[this.workLogHashList.length-1].totalTime=item.workDays;
              this.workLogHashList[this.workLogHashList.length-1].remarks=item.remarks||'';
              console.log(this.workLogHashList)
          	}
          });
    	  });
      },
      getCurrentDate(){
        let currentDate=new Date();
        let currentMonthStr=currentDate.getFullYear()+'-'+(currentDate.getMonth()+1);
        currentDate=new Date(currentMonthStr);
        this.searchData.time=currentDate;
        this.getWorkLogList(this.searchData);
      },
    },
    mounted() {
      this.getDepartmentList();
      this.getCurrentDate();
    }
  }
</script>

<style scoped lang="scss">
.tableBox{
  margin-top: 20px;
}
.btnBox{
  text-align: left;
}
</style>
