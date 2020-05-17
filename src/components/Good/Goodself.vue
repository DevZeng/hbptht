<template>
  <el-row class="warp" style="padding:20px 0 0 20px;">
    <el-col :span="24" class="warp-breadcrum">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item :to="{ path: '/' }"><b>首页</b></el-breadcrumb-item>
        <el-breadcrumb-item>自提地址</el-breadcrumb-item>
        <el-breadcrumb-item>地址列表</el-breadcrumb-item>
      </el-breadcrumb>
    </el-col>

    <el-col :span="24" class="warp-main">


     <el-form :inline="true">
      <el-form-item>
        <el-button type="primary" size="mini" @click="newone">新增地址</el-button>
      </el-form-item>
    </el-form>

    <el-table :data="list" border stripe style="width:901px" size="small">
      <el-table-column prop="id" label="编号" width="150" align="center">
      </el-table-column>

      <el-table-column prop="store_name" label="商家名称" width="150" align="center">
      </el-table-column>
      <el-table-column prop="address" label="详细地址" width="450" align="center">
      </el-table-column>


      <el-table-column label="操作" width="150" align="center">
       <template slot-scope="scope">
        <el-button type="primary" size="mini" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>

        <el-button type="danger" size="mini" @click="handleDelete(scope.$index, scope.row)">删除</el-button>
      </template>
    </el-table-column>

  </el-table>

  <el-pagination style="float:left;margin-top:20px;" :current-page="currentPage" :page-sizes="[10, 20, 30, 40]" :page-size="limit" @current-change="handleCurrentChange" @size-change="handleSizeChange" layout="total,sizes, prev, pager, next, jumper" :total="count" prev-text="上一页" next-text="下一页">
  </el-pagination>



</el-col>

<el-col>
  <el-dialog :title="diatitle" :visible.sync="dialogNewVisible" width="500" center style="min-width: 500px">
    <el-form ref="newadv" :model="newadv" label-width="120px">



      <el-form-item label="商家名称：">
        <el-input v-model="newadv.store_name" style="max-width: 300px;" placeholder="请输入商家名称："></el-input>
      </el-form-item>
      <el-form-item label="详细地址：">
        <el-input v-model="newadv.address" style="max-width: 300px;" placeholder="请输入详细地址："></el-input>
      </el-form-item>

      <el-form-item style="margin-left: calc(50% - 200px);">
        <el-button size="small" type="primary" @click="save()">保 存</el-button>
        <el-button size="small" @click="dialogNewVisible = false">取 消</el-button>
      </el-form-item>
    </el-form>
  </el-dialog>
</el-col>

<el-col>
  <el-dialog title="删除不可恢复，是否确定删除？" :visible.sync="dialogDelVisible" width="30%">
    <div slot="footer" class="dialog-footer">
      <el-button type="primary" @click="submitdel()">确 定</el-button>
      <el-button @click="dialogDelVisible = false">取 消</el-button>
    </div>
  </el-dialog>
</el-col>

</el-row>
</template>



<script>


  import { typeGet } from '../../api/api';
  import { typePost } from '../../api/api';
  import { typeDel } from '../../api/api';
  import { getself } from'../../api/api';
  import { addself } from'../../api/api';
  import { delself } from'../../api/api';
  import qiniu from '../../api/qiniu';

  import { Message } from 'element-ui';

  export default {
    data() {
      return {

        loading:false,
        uptoken:{
          token:qiniu.token,
        },
        upurl:qiniu.upurl,

        currentPage: 1,
        list:[],
        count:0,
        limit:10,
        dialogNewVisible:false,
        dialogDelVisible:false,

        putorup:'up',
        imgSrc:"../static/images/default.png",
        newadv:{
         store_name:'',
         address:'',
       },

       diatitle:'新增地址',

       editId:'',
       delId:'',

     };
   },

   methods:{


    getlist(){
      var id = "";
      getself(id).then((res) => {
        console.log(res.data);
        this.list=res.data;
        //this.count=res.data.count
      });
    },


    newone(){
     this.putorup='up';
     this.imgSrc="../static/images/default1.png";
     this.diatitle='新增地址',
     this.dialogNewVisible=true,

     this.newadv={
     store_name:'',
         address:'',
     }
   },

  save(){

    if(this.newadv.store_name=='' && this.nothree==true){
      this.$message({
        message:"请输入商家名称",
        type:'error'
      })
    }else if(this.newadv.address==''){
      this.$message({
        message: '请输入详细地址',
        type: 'error'
      });
    }
    else{
      if( this.putorup=='put'){
        var allParams = {
          store_name:this.newadv.store_name,
          address:this.newadv.address,
          id:this.editId,
          
        };
      }else{
        var allParams = {
          store_name:this.newadv.store_name,
          address:this.newadv.address,
        };
      }
      
      addself(allParams).then((res) => {
        if (res.msg === "ok") {
         this.$message({
          message: '提交成功',
          type: 'success'
        });
         this.getlist();
         this.dialogNewVisible=false
       } else {
         this.$message({
          message: res.msg,
          type: 'error'
        });
       }
     });
    }
  },

  handleEdit(index, row){
    this.diatitle='编辑分类';
    this.dialogNewVisible = true;
    this.putorup='put';
    this.editId = row.id;
    this.newadv.store_name=row.store_name;
    this.newadv.address=row.address
  },

  handleDelete(index, row) {
    this.dialogDelVisible = true;
    this.delId = row.id;
  },

  submitdel(){
    this.dialogDelVisible = false;
    var allParams={id:this.delId}
    console.log(allParams)
    delself(allParams).then((res) => {
      if (res.msg === "ok") {
       this.$message({
        message: '删除成功',
        type: 'success'
      });
       this.getlist();
       this.dialogDelVisible = false;
     } else {
       this.$message({
        message: "有商品绑定了该地址",
        type: 'error'
      });
     }
   });
  },

  handleCurrentChange(val) {
    this.currentPage = val;
    this.getlist();
  },

  handleSizeChange(val){
    this.limit = val;
    this.getlist();
  },
},

mounted: function () {
  this.getlist();
}
}
</script>


<style>
  .icon{
    width: 20px;
    height: 20px;
    margin: 2px;
  }
</style>
