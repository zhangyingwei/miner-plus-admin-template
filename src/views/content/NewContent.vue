<template>
  <div class="app-container">
    <!-- 搜索条件 -->
    <div class="filter-container">
      <el-input @keyup.enter.native="handleFilter" style="width: 200px;" class="filter-item" placeholder="标题" v-model="listQuery.title">
      </el-input>

      <el-input @keyup.enter.native="handleFilter" style="width: 200px;" class="filter-item" placeholder="来源" v-model="listQuery.sitename">
      </el-input>

      <el-button class="filter-item" type="primary" v-waves icon="search" @click="handleFilter">搜索</el-button>
      <el-button :disabled="!canClick" class="filter-item" type="primary" @click="handleCheck"  icon="edit">批量审核</el-button>

      <el-button :disabled="!canClick" class="filter-item" type="primary" @click="handleDelAll"  icon="edit">添加到黑名单</el-button>     
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
                  <el-tag type="success" v-if="scope.row.state == 1">正常</el-tag>
                  <el-tag type="warning" v-if="scope.row.state == 2">失效</el-tag>
                  <el-tag type="warning" v-if="scope.row.state == 3">审核通过</el-tag>
                  <el-tag type="warning" v-if="scope.row.state == 4">审核不通过</el-tag>
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
    <!-- 审核弹窗 -->
    <el-dialog :title="dialogTitle" :visible.sync="dialogFormVisible">
      <div>
        <el-table
          :data="temp.data"
          border
          style="width: 100%"
          height="250">
          <el-table-column
            prop="id"
            label="编号"
            width="150">
          </el-table-column>
          <el-table-column
            prop="title"
            label="标题">
          </el-table-column>
          <el-table-column
            prop="pubdate"
            label="发布时间"
            width="200">
          </el-table-column>
        </el-table>
      </div>
      <div class="atypepanel">
        类型: 
        <el-autocomplete
          class="inline-input"
          v-model="atype"
          :fetch-suggestions="queryAtype"
          placeholder="请输入类型"
          @select="handleAtypeSelect"
        ></el-autocomplete>
      </div>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="handleCheckSubmit">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import {global} from 'src/global/global';
import {api} from 'src/global/api';

export default {
  data() {
    return {
        // list: null,
        // listLoading: true,
        list: null,//表格list
        total: null,
        listLoading: true,
        atype: "", //审核时，文章类型
        listQuery: {
            currPage: 1,
            pageSize: 10,
            title: '',
            sitename: '',//类型
        },
        dialogTitle: "",
        temp: {
          showPushTime: true,
          data: []
        },
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
    //获取列表数据
    getList() {
        var vm = this;
        console.log(vm.listQuery)
        vm.listLoading = true;
        global.get( api.contentList,{params: vm.listQuery },function(res){
                //console.log('-------获取到数据：',JSON.stringify(res) )
                var data = res.body;
               if(data.resultCode == 0){ 
                 vm.$message("加载数据成功")
                    vm.list = data.data.data;
                    console.log('列表数据：',vm.list);
                    vm.listQuery.currPage = data.data.currPage;
                    vm.listQuery.pageSize = data.data.pageSize;
                    vm.total = data.data.total;
                    vm.listLoading = false;
               }else{
                    //alert(res.body.resultMsg)
                    vm.$message("加载失败了")
                    vm.listLoading = false;
               }
               
                
        },function(res){
            
            vm.listLoading = false;
        },false)

    },
    //批量删除
    handleDelAll(){
        this.dialogFormVisible = true;
        this.dialogTitle = "加入黑名单"
        this.temp.data = this.multipleSelection;
        this.temp.showPushTime = false;
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
    //批量审核
    handleCheck() {
        this.dialogFormVisible = true;
        this.dialogTitle = "批量审核"
        this.temp.data = this.multipleSelection;
        this.temp.showPushTime = true;
    },
    //新增提交
    handleCheckSubmit(){
        var vm = this;
        console.log('新增入参：',vm.temp,vm.list)
        //这里作演示用，正式新增 请提交到接口
        vm.list.push(vm.temp)
        console.log('新增后',vm.list)
        this.dialogFormVisible = false;
    },
    handleSelectionChange(val) {
        this.multipleSelection = val;
        if(val.length > 0){
          this.canClick = true
        }else{
          this.canClick = false
        }
    },
    queryAtype(queryString, cb){
      var result = [
        {value: "java"},
        {value: "javascript"},
        {value: "go"},
        {value: "php"}
      ]
      cb(result)
    },
    handleAtypeSelect(item){
      console.log(item)
    }
  }
};
</script>

<<style>
  .atypepanel {
    padding-top: 20px;
  }
</style>
