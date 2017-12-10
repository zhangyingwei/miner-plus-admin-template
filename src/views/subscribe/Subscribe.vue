<template>
  <div class="app-container">
    <!-- 搜索条件 -->
    <div class="filter-container">
      <el-autocomplete @keyup.enter.native="handleFilter" @select="handleFilter" :fetch-suggestions="querySearch" style="width: 200px;" class="filter-item" placeholder="电子邮箱" v-model="listQuery.email">
      </el-autocomplete>

      <el-select clearable class="filter-item" style="width: 130px" v-model="listQuery.flag" placeholder="状态">
        <el-option v-for="item in  typeOptions" :key="item.key" :label="item.display_name+'('+item.key+')'" :value="item.key">
        </el-option>
      </el-select>

      <el-button class="filter-item" type="primary" v-waves icon="search" @click="handleFilter">搜索</el-button>
    </div>
    <!-- 表格 -->
    <el-table ref="multipleTable" @selection-change="handleSelectionChange" :data="list" v-loading.body="listLoading" element-loading-text="拼命加载中" border fit highlight-current-row  >
         <el-table-column type="selection" width="65">
         </el-table-column>

          <el-table-column align="center" label='编号' width="80">
            <template scope="scope">
              {{scope.row.id}}
            </template>
          </el-table-column>

          <el-table-column label="订阅邮箱" width="200">
            <template scope="scope">
              {{scope.row.email}}
            </template>
          </el-table-column>

          <el-table-column label="订阅主题" width="">
            <template scope="scope">
              <el-tag :type="getType()" v-for="t in scope.row.topics" key="topic"><span>{{t}}</span></el-tag>
            </template>
          </el-table-column>

          <el-table-column label="订阅时间"  align="center" width="180">
            <template scope="scope">
              {{scope.row.subdate}}
            </template>
          </el-table-column>

          <el-table-column align="center" prop="created_at" label="取消订阅时间" width="180">
            <template scope="scope">
              <!-- <i class="el-icon-time"></i> -->
              <span>{{scope.row.dissubdate}}</span>
            </template>
          </el-table-column>

            <el-table-column align="center"  label="状态" width="100" >
                <template scope="scope">
                  <el-tag v-if="scope.row.flag==0" type="primary">待验证</el-tag>
                  <el-tag v-if="scope.row.flag==1" type="warning">已通知</el-tag>
                  <el-tag v-if="scope.row.flag==2" type="success">已验证</el-tag>
                  <el-tag v-if="scope.row.flag==3" type="danger">黑名单</el-tag>
                  <el-tag v-if="scope.row.flag==9" type="gray">已退订</el-tag>
                </template>
            </el-table-column>

            <el-table-column align="center"  label="操作" >
                <template scope="scope">
                   <el-button size="small" :disabled="canBlack(scope)" :type="getBlackType(scope)" @click="handleBlack(scope.$index, scope.row)">{{getBlackName(scope)}}</el-button>
                </template>
            </el-table-column>
    </el-table>
    <!-- 分页 -->
    <div v-show="!listLoading" class="pagination-container">
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page.sync="listQuery.currentPage"
        :page-sizes="[5,10,20,30, 50]" :page-size="listQuery.pageSize" layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </div>

  </div>
</template>

<script>
// import { getList } from 'api/article';
import {global} from 'src/global/global';
import {api} from 'src/global/api';

export default {
  data() {
    return {
        list: null,//表格list
        total: null,
        listLoading: true,
        restaurants: [],
        listQuery: {
            currentPage: 1,
            pageSize: 10,
            email: null,
            flag: null,//类型
        },
        temp: {
            "chnlId": "22",
            "hisChnlId": "",
            "chnlName": "",
            "state": "",
            "isavailable": "",
            "orderNum": 10
        },
        typeOptions:[
          { key: '0', display_name: '待验证' },
          { key: '1', display_name: '已通知' },
          { key: '2', display_name: '已验证' },
          { key: '3', display_name: '黑名单' },
          { key: '9', display_name: '已退订' }
        ],
        dialogFormVisible: false,
        multipleSelection: [],
        canClick: false
    }
  },
  mounted() {
    var vm = this;
    vm.getList();
  },
  methods: {
    getType() {
        const types = ['','gray','primary','success','warning','danger']
        const index = Math.round(Math.random()*types.length)
        return types[index]
    },
    //获取列表数据
    getList() {
        var vm = this;
        console.log("listQuery",vm.listQuery)

        vm.listLoading = true;

        global.get( api.subscribeList,{params: vm.listQuery },function(res){
                // console.log('获取到数据：',res )
                var data = res.body;
               if(data.state == 200){
                    vm.list = data.data;
                    vm.list = data.data.map(row => {
                      if(row.topics){
                        row.topics = row.topics.split(",")
                      }
                      return row
                    })

                    new Set(vm.list.map(row => {
                      return row.email;
                    })).forEach(row => {
                      vm.restaurants.push({"value":row})
                    });
                    vm.listQuery.currentPage = data.page.currentPage;
                    vm.listQuery.pageSize = data.page.size;
                    vm.total = data.page.total;
                    vm.listLoading = false;
                    console.log("vm",vm);
               }else{
                    Message({
                        showClose: true,
                        message: res.body.message,
                        type: 'error'
                    });
                    vm.listLoading = false;
               }
        },function(res){
            vm.listLoading = false;
        },false)

    },
    querySearch(queryString, cb) {
        var restaurants = this.restaurants;
        console.log(restaurants)
        cb(restaurants);
    },
    //单个删除
    handleBlack(index,row){
        var vm = this;
        // console.log('单个删除选择的row：',index,'-----',row);
        // //前端删除。
        // vm.list.splice(index,1)
        // this.$message('黑名单 '+row.email);
        vm.listLoading = true;
        if(row.flag === 3){
          vm.updateFlagToGreen(vm,row);
        }else{
          vm.updateFlagToBlack(vm,row);
        }
    },
    updateFlagToGreen(vm,row){
      global.put(api.backtogreen+row.id,{},function(res){
        // console.log("res",res);
        vm.listLoading = false;
        row.flag = 2;
      },function(res){
        vm.listLoading = false;
      })
    },
    updateFlagToBlack(vm,row){
      global.put(api.addtoblack+row.id,{},function(res){
        // console.log("res",res);
        vm.listLoading = false;
        row.flag = 3;
      },function(res){
        vm.listLoading = false;
      })
    },
    getBlackName(scope){
      return scope.row.flag === 3?"洗白白":"黑名单";
    },
    getBlackType(scope){
      return scope.row.flag === 3?"":"danger";
    },
    canBlack(scope){
      return scope.row.flag === 1?true:false;
    },
    //搜索
    handleFilter() {
      this.getList();
    },
    //操作分页
    handleSizeChange(val) {
      this.listQuery.pageSize = val;
      this.getList();
    },
    //操作分页
    handleCurrentChange(val) {
        console.log('--------',val)
      this.listQuery.currentPage = val;
      this.getList();
    },
    handleSelectionChange(val) {
        this.multipleSelection = val;
        if(val.length > 0){
            this.canClick = true
        }else{
          this.canClick = false
        }
    },
    formatJson(filterVal, jsonData) {
      return jsonData.map(v => filterVal.map(j => v[j]))
    }
  }
};
</script>
