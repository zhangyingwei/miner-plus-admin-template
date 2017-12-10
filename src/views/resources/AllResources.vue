<template>
  <div class="app-container">
    <!-- 搜索条件 -->
    <div class="filter-container">
      <el-input @keyup.enter.native="handleFilter" style="width: 200px;" class="filter-item" placeholder="资源地址" v-model="listQuery.title">
      </el-input>

      <el-select clearable class="filter-item" style="width: 130px" v-model="listQuery.type" placeholder="资源类型">
        <el-option v-for="item in typeOptions" :key="item.key" :label="item.display_name+'('+item.key+')'" :value="item.key">
        </el-option>
      </el-select>

      <el-select clearable class="filter-item" style="width: 130px" v-model="listQuery.group" placeholder="资源分组">
        <el-option v-for="item in  groupOptions" :key="item.key" :label="item.display_name+'('+item.key+')'" :value="item.key">
        </el-option>
      </el-select>

      <el-button class="filter-item" type="primary" v-waves icon="search" @click="handleFilter">搜索</el-button>

      <el-button :disabled="!canClick" class="filter-item" type="primary" @click="handleDelAll"  icon="edit">批量删除</el-button>     
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

          <el-table-column label="资源地址" width="">
            <template scope="scope">
              {{scope.row.resources}}
            </template>
          </el-table-column>

          <el-table-column label="资源分组" width="100">
            <template scope="scope">
              <el-tag type="success">{{scope.row.rgroup}}</el-tag>
            </template>
          </el-table-column>

          <el-table-column label="资源类型"  align="center" width="100">
            <template scope="scope">
              <el-tag type="warning">{{scope.row.rtype}}</el-tag>
            </template>
          </el-table-column>

          <el-table-column align="center" prop="created_at" label="创建时间" width="">
            <template scope="scope">
              <!-- <i class="el-icon-time"></i> -->
              <span>{{scope.row.createdate}}</span>
            </template>
          </el-table-column>

          <el-table-column align="center" prop="created_at" label="修改时间" width="">
            <template scope="scope">
              <!-- <i class="el-icon-time"></i> -->
              <span>{{scope.row.updatedate}}</span>
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
                   <el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                   <el-button size="small" type="danger" @click="handleDelete(scope.$index, scope.row)">删除</el-button>
                </template>
            </el-table-column>
    </el-table>
    <!-- 分页 -->
    <div v-show="!listLoading" class="pagination-container">
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page.sync="listQuery.currPage"
        :page-sizes="[10,20,30, 50]" :page-size="listQuery.pageSize" layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </div>
    <!-- 新增弹窗 -->
    <el-dialog title="编辑资源" :visible.sync="dialogFormVisible">
          <el-form :inline="true" class="small-space" :model="temp" label-position="left" label-width="70px">

            <el-form-item label="资源地址">
              <el-input style="min-width:250px;" v-model="temp.resources"></el-input>
            </el-form-item>
            <el-form-item label="资源分组">
              <el-select clearable class="filter-item" v-model="temp.rgroup" placeholder="资源分组">
                <el-option v-for="item in  groupOptions" :key="item.key" :label="item.display_name+'('+item.key+')'" :value="item.key">
                </el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="资源类型">
              <el-select clearable class="filter-item" v-model="temp.rtype" placeholder="资源类型">
                <el-option v-for="item in typeOptions" :key="item.key" :label="item.display_name+'('+item.key+')'" :value="item.key">
                </el-option>
              </el-select>
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
            pageSize: 10
        },
        temp: {
            "resources": "",
            "rgroup": "",
            "rtype": ""
        },
        typeOptions:[
          { key: 'java', display_name: 'java' },
          { key: 'php', display_name: 'php' }
        ],
        groupOptions:[
          { key: 'rss', display_name: 'rss' },
          { key: 'site', display_name: 'site' }
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
                  vm.$message(res.data.message)
                  vm.listLoading = false;
               }
        },function(res){
            vm.listLoading = false;
        },false)

    },
    //编辑
    handleEdit(index,row){
        var vm = this;
        console.log('编辑的row：',index,'-----',row);
        vm.dialogFormVisible = true
        vm.temp.resources = row.resources
        vm.temp.rtype = row.rtype
        vm.temp.rgroup = row.rgroup
        //跳页面进行修改
        //this.$router.push('/example/form'); 
        //this.$router.push( { path: '/example/form', query: { id: row.chnlId } } ); //带参跳转
    },
    //单个删除
    handleDelete(index,row){
        var vm = this;
        console.log('单个删除选择的row：',index,'-----',row);
        //前端删除。
        vm.list.splice(index,1)
        this.$message("delete-"+row.resources);
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
      this.listQuery.currentPage = val;
      this.getList();
    },
    //新增提交
    handleCreateSubmit(){
        var vm = this;
        console.log('新增入参：',vm.temp)
        this.dialogFormVisible = false;
    },
    handleSelectionChange(val) {
        this.multipleSelection = val;
        if(val.length > 0){
            this.canClick = true;
        }else{
          this.canClick = false;
        }
    },
    formatJson(filterVal, jsonData) {
      return jsonData.map(v => filterVal.map(j => v[j]))
    }
  }
};
</script>
