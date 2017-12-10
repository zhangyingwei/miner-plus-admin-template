<template>
  <div class="app-container">
   
    <!-- 表格 -->
    <el-table ref="multipleTable" @selection-change="handleSelectionChange" :data="list" v-loading.body="listLoading" element-loading-text="拼命加载中" border fit highlight-current-row  >
         <el-table-column type="selection" width="65">
         </el-table-column>

          <el-table-column align="center" label='编号' width="80">
            <template scope="scope">
              {{scope.row.id}}
            </template>
          </el-table-column>

          <el-table-column label="资源地址" width="">
            <template scope="scope">
              {{scope.row.resources}}
              <el-button @click="handleOpenResources(scope.row.resources)" style="margin-left:10px;" icon="search" type="info" size="mini"></el-button>
            </template>
          </el-table-column>

          <el-table-column align="center" prop="created_at" label="创建时间" width="">
            <template scope="scope">
              <span>{{scope.row.createdate}}</span>
            </template>
          </el-table-column>

            <el-table-column align="center"  label="状态"  width="100" >
                <template scope="scope">
                  <el-tag v-if="scope.row.flag == 0">未审核</el-tag>
                  <el-tag type="success" v-if="scope.row.flag == 1">正常</el-tag>
                  <el-tag type="warning" v-if="scope.row.flag == 2">失效</el-tag>
                  <el-tag type="danger" v-if="scope.row.flag == 9">删除</el-tag>
                </template>
            </el-table-column>

            <el-table-column align="center"  label="操作" >
                <template scope="scope">
                   <el-button size="small" @click="handleEdit(scope.$index, scope.row)">审核</el-button>
                   <el-button size="small" type="danger" @click="handleBlack(scope.$index, scope.row)">黑名单</el-button>
                </template>
            </el-table-column>
    </el-table>
    <!-- 分页 -->
    <div v-show="!listLoading" class="pagination-container">
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page.sync="listQuery.currentPage"
        :page-sizes="[10,20,30, 50]" :page-size="listQuery.pageSize" layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </div>
    <!-- 新增弹窗 -->
    <el-dialog title="资源审核" :visible.sync="dialogFormVisible">
          <el-form class="small-space" :inline="true" :model="temp" label-position="left">

            <el-form-item>
              <el-input :disabled=true v-model="temp.resources">
                <template slot="prepend">资源地址</template>
                <el-button slot="append" @click="handleOpenResources(temp.resources)" icon="search">访问地址</el-button>
              </el-input>
            </el-form-item>

            <el-form-item label="资源分组">
              <el-select v-model="temp.rgroup" placeholder="资源分组">
                <el-option label="rss" value="rss"></el-option>
                <el-option label="site" value="site"></el-option>
              </el-select>
            </el-form-item>

            <el-form-item label="资源类型">
            <el-autocomplete
              class="inline-input"
              v-model="temp.rtype"
              :fetch-suggestions="querySearch"
              placeholder="请输入内容"
              @select="handleSelect"
            ></el-autocomplete>
            </el-form-item>
          </el-form>

          <div slot="footer" class="dialog-footer">
            <el-button @click="dialogFormVisible = false">取 消</el-button>
            <el-button type="primary" @click="handleCreateSubmit">确 定</el-button>
          </div>
    </el-dialog>

  </div>
</template>

<script>
// import { getList } from 'api/article';
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
        listQuery: {
            currentPage: 1,
            pageSize: 10,
            restype:"new"
        },
        temp: {
            id: "",
            resources: "",
            rtype: "",
            rgroup: ""
        },
        typeOptions:[
          { key: '0', display_name: '未审核' },
          { key: '1', display_name: '正常' },
          { key: '2', display_name: '失效' },
          { key: '9', display_name: '删除' }
        ],
        dialogFormVisible: false,
        multipleSelection: []

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
        global.get( api.resourcesList,{params: vm.listQuery },function(res){
                console.log('-------获取到数据：',res )
                var data = res.data;
               if(data.state == 200){
                    vm.list = data.data;
                    console.log('列表数据：',vm.list);
                    vm.listQuery.currentPage = data.page.currentPage;
                    vm.listQuery.pageSize = data.page.size;
                    vm.total = data.page.total;
                    vm.listLoading = false;
               }else{
                    //alert(res.body.resultMsg)
                    Message({
                        showClose: true,
                        message: res.data.message,
                        type: 'error'
                    });
                    vm.listLoading = false;
               }
        },function(res){
            vm.listLoading = false;
        },false)

    },
    //打开超链接
    handleOpenResources(resources){
        window.open(resources,"_blank")
    },
    //编辑
    handleEdit(index,row){
        var vm = this;
        console.log('编辑的row：',index,'-----',row);
        vm.temp.resources = row.resources
        vm.temp.id = row.id
        vm.dialogFormVisible = true
    },
    //单个删除
    handleBlack(index,row){
        var vm = this;
        vm.listLoading = true;
        console.log(api.addresourcesToBlack+row.id)
        global.put(api.addresourcesToBlack+row.id,{},function(res){
          if(res.data.state == 200){
            vm.$message(res.data.message);
          }else{
            vm.$message(res.data.message);
          }
          vm.listLoading = false;
          vm.getList()
        },function(res){
          vm.$message("返回结果错误")
          vm.listLoading = false;
          vm.getList()
        })
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
    handleSelect(item){
      console.log(item)
    },
    querySearch(queryString, cb) {
        var restaurants = this.restaurants;
        var vm = this;
        global.get(api.listResourcesTypes,{},function(res){
          var data = res.data
          var results = data.data.map(item => {
            return {value: item}
          })
          cb(results)
        },function(res){
          vm.$message("获取类型列表失败");
        })
    },
    //新增
    handleCreate() {
        this.dialogFormVisible = true;
    },
    //新增提交
    handleCreateSubmit(){
        var vm = this;
        console.log('新增入参：',vm.temp)
        global.put(api.validresources+vm.temp.id,vm.temp,function(res){
          console.log(res.data.message)
          vm.dialogFormVisible = false;
          vm.handleFilter()
        },function(res){
          vm.$message("审核失败")
          vm.dialogFormVisible = false;
          vm.handleFilter()
        })
    },
    handleSelectionChange(val) {
        this.multipleSelection = val;
    },
    handleDownload() {
      var vm = this;
      require.ensure([], () => {
        const { export_json_to_excel } = require('vendor/Export2Excel');
        const tHeader = ['字段1', '字段2', '字段3', '字段4', '字段5'];
        const filterVal = ['chnlId', 'hisChnlId', 'chnlName', 'state', 'isavailable'];
        const list = vm.list;
        const data = vm.formatJson(filterVal, list);
        export_json_to_excel(tHeader, data, '导出的列表excel');
      })
    },
    formatJson(filterVal, jsonData) {
      return jsonData.map(v => filterVal.map(j => v[j]))
    }
  }
};
</script>
