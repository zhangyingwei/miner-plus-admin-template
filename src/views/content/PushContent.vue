<template>
  <div class="app-container">
    <!-- 搜索条件 -->
    <div class="filter-container">
      <el-input @keyup.enter.native="handleFilter" style="width: 200px;" class="filter-item" placeholder="标题" v-model="listQuery.title">
      </el-input>

      <el-select clearable class="filter-item" style="width: 130px" v-model="listQuery.atype" placeholder="类型">
        <el-option v-for="item in  typeOptions" :key="item.key" :label="item.display_name+'('+item.key+')'" :value="item.key">
        </el-option>
      </el-select>

      <el-button class="filter-item" type="primary" v-waves icon="search" @click="handleFilter">搜索</el-button>

      <el-button :disabled="!canPush" class="filter-item" type="primary" @click="handleDelAll"  icon="edit">批量删除</el-button>

      <el-button class="filter-item" type="primary" @click="handlePush" :disabled="!canPush" icon="upload2">推送</el-button>
    </div>
   
    <!-- 表格 -->
    <el-table ref="multipleTable" @selection-change="handleSelectionChange" :data="list" v-loading.body="listLoading" element-loading-text="拼命加载中" border fit highlight-current-row  >
         <el-table-column type="selection" width="65">
         </el-table-column>

          <el-table-column align="center" label='编号' width="100">
            <template scope="scope">
              {{scope.row.id}}
            </template>
          </el-table-column>

          <el-table-column label="来源" width="">
            <template scope="scope">
              <a target="_blank" :href="scope.row.site">{{scope.row.sitename}}</a>
            </template>
          </el-table-column>

          <el-table-column label="标题" width="">
            <template scope="scope">
              <a target="_blank" :href="scope.row.url">{{scope.row.title}}</a>
            </template>
          </el-table-column>

          <el-table-column label="作者"  align="center" width="">
            <template scope="scope">
              {{scope.row.author}}
            </template>
          </el-table-column>

          <el-table-column align="center" prop="created_at" label="发布时间" width="">
            <template scope="scope">
              <!-- <i class="el-icon-time"></i> -->
              <span>{{scope.row.pubdate}}</span>
            </template>
          </el-table-column>

            <el-table-column align="center"  label="状态" >
                <template scope="scope">
                  <el-tag v-if="scope.row.state == 0">初始</el-tag>
                  <el-tag type="gray" v-if="scope.row.state == 1">正常</el-tag>
                  <el-tag type="primary" v-if="scope.row.state == 2">失效</el-tag>
                  <el-tag type="warning" v-if="scope.row.state == 3">审核通过</el-tag>
                  <el-tag type="danger" v-if="scope.row.state == 4">审核不通过</el-tag>
                  <el-tag type="danger" v-if="scope.row.state == 9">删除</el-tag>
                </template>
            </el-table-column>
    </el-table>
    <!-- 分页 -->
    <div v-show="!listLoading" class="pagination-container">
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page.sync="listQuery.currPage"
        :page-sizes="[10,20,30, 50]" :page-size="listQuery.pageSize" layout="total, sizes, prev, pager, next, jumper" :total="total">
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
        listQuery: {
            currPage: 1,
            pageSize: 10,
            title: '',
            atype: null,//类型
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
          { key: '001', display_name: '类型1' },
          { key: '002', display_name: '类型2' },
          { key: '003', display_name: '类型3' }
        ],
        multipleSelection: [],
        canPush: false

    }
  },
  mounted() {
    var vm = this;
    vm.getList();
  },
  methods: {
    //获取列表数据
    getList() {
        var vm = this;

        console.log(vm.listQuery)

        vm.listLoading = true;

        global.get( api.contentList,{params: vm.listQuery },function(res){
                //console.log('-------获取到数据：',JSON.stringify(res) )
                var data = res.body;
               if(data.resultCode == 0){ 
                    
                    vm.list = data.data.data;
                    console.log('列表数据：',vm.list);
                    vm.listQuery.currPage = data.data.currPage;
                    vm.listQuery.pageSize = data.data.pageSize;
                    vm.total = data.data.total;

                    vm.listLoading = false;
                    
               }else{
                    //alert(res.body.resultMsg)
                    Message({
                        showClose: true,
                        message: res.body.resultMsg,
                        type: 'error'
                    });
                    vm.listLoading = false;
               }
               
                
        },function(res){
            
            vm.listLoading = false;
        },false)

    },
    //批量删除
    handleDelAll(){
        var vm = this;
        console.log('批量删除选择的row：',vm.multipleSelection)
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
      this.listQuery.currPage = val;
      this.getList();
    },
    handlePush() {
        var vm = this;
        const size = vm.multipleSelection.length
        this.$confirm('共选中 '+size+' 条内容，确定为明天的推送内容吗？', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'success'
        }).then(() => {
          this.$message({
            type: 'success',
            message: '推送成功!'
          });
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消推送'
          });          
        });
    },
    handleSelectionChange(val) {
        this.multipleSelection = val;
        if(this.multipleSelection.length > 0){
            this.canPush = true
        }else{
            this.canPush = false
        }
    },
    formatJson(filterVal, jsonData) {
      return jsonData.map(v => filterVal.map(j => v[j]))
    }
  }
};
</script>
