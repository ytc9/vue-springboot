<template>
<div>
  
  <div style="margin: 10px 0">
    <!--搜索框图标suffix-icon-->
    <el-input style="width:200px;margin-left: 5px"
              placeholder="请输入名称"
              suffix-icon="el-icon-search"
              v-model="name"
    >
    </el-input>
   
    <el-button style="margin-left: 5px" type="primary" @click="load">搜索</el-button>
    <el-button style="margin-left: 5px" type="warning" @click="reset">重置</el-button>
  </div>
  <div style="margin: 10px 0">
    <el-button type="primary" @click="handleAdd()">新增<i class="el-icon-circle-plus-outline"></i></el-button>
    
    <el-popconfirm
        style="margin-left: 5px"
        confirm-button-text='好的'
        cancel-button-text='我再想想'
        icon="el-icon-info"
        icon-color="red"
        title="您确定批量删除这些数据吗？"
        @confirm="delBatch"
    >   <!--删除确认框 这里有确认框就不用@click用@confirm-->
      <el-button type="danger" slot="reference">批量删除<i class="el-icon-remove-outline"></i></el-button>
    </el-popconfirm>
  </div>
  <!--border stripe表格分割线
  element ui的bug  :header-cell-style="{'background-color': '#ccc'}"
  row-key="id" 按照后端传来的id 形成树形结构表
  default-expand-all
  -->
  <el-table :data="tableData"
            :header-cell-style="{'background-color': '#ccc'}"
            border stripe
            @selection-change="handleSelectionChange"
            row-key="id"
            default-expand-all
  >
    <el-table-column
        type="selection"
        width="55">
    </el-table-column>
    <el-table-column prop="id" label="ID" width="80">
    </el-table-column>
    <el-table-column prop="name" label="名称" >
    </el-table-column>
     <el-table-column prop="path" label="路径" >
     </el-table-column>
    <el-table-column prop="pagePath" label="页面路径" >
    </el-table-column>
     <!-- class-name="fontSize18" 图标样式
          align="center"
          label-class-name="fontSize12"  头部字体  -->
     <el-table-column prop="icon"
                      label="图标"
                      class-name="fontSize18"
                      align="center"
                      label-class-name="fontSize12">
       <template slot-scope="scope">
         <i :class="scope.row.icon"></i>
       </template>
     </el-table-column>
    <el-table-column prop="description" label="描述" >
    </el-table-column>
    
    <el-table-column label="操作" width="280" align="center">
      <template slot-scope="scope">
        <!--这里必须要绑定slot scope不然没法获取编辑数据
           scope.row就是获取的属性
        -->
        <el-button type="primary" @click="handleAdd(scope.row.id)" v-if="!scope.row.pid && !scope.row.path">新增子菜单<i class="el-icon-plus"></i></el-button>
        <el-button type="success" @click="handleEdit(scope.row)">编辑<i class="el-icon-edit"></i></el-button>
        <el-popconfirm
            style="margin-left: 5px"
            confirm-button-text='好的'
            cancel-button-text='我再想想'
            icon="el-icon-info"
            icon-color="red"
            title="您确定删除吗？"
            @confirm="handleDelete(scope.row.id)"
        >   <!--删除确认框 这里有确认框就不用@click用@confirm-->
          <el-button type="danger" slot="reference">删除<i class="el-icon-remove-outline"></i></el-button>
        </el-popconfirm>
      </template>
    </el-table-column>
  </el-table>
  
  <div style="padding: 10px 0">
    <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="pageNum"
        :page-sizes="[10, 20]"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
    </el-pagination>
  </div>
  
  
  <!--visible.sync绑定false属性默认不展示  编辑框 -->
  <el-dialog title="菜单信息" :visible.sync="dialogFormVisible" width="30%" style="border-radius: 20px" >
    <el-form label-width="80px" size="small">
      <el-form-item label="名称" >
        <el-input v-model="form.name" autocomplete="off"></el-input>
      </el-form-item>
       <el-form-item label="路径" >
          <el-input v-model="form.path" autocomplete="off"></el-input>
       </el-form-item>
      <el-form-item label="页面路径" >
        <el-input v-model="form.pagePath" autocomplete="off"></el-input>
      </el-form-item>
       <el-form-item label="图标" >
         <el-select clearable v-model="form.icon" placeholder="请选择" style="width: 100%">
             <el-option v-for="item in options" :key="item.name" :label="item.name" :value="item.value">
               <i :class="item.value">{{item.name}}</i>
             </el-option>
         </el-select>
       </el-form-item>
       
      <el-form-item label="描述" >
        <el-input v-model="form.description" autocomplete="off"></el-input>
      </el-form-item>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button @click="dialogFormVisible = false">取 消</el-button>
      <el-button type="primary" @click="save">确 定</el-button>
    </div>
  </el-dialog>
</div>
</template>

<script>
import request from "@/utils/request";

export default {
    name: "User",
    data(){
        return{
            tableData:[],
            total:0,
            pageNum:1,
            pageSize:10,
            name:"",
            nickname:"",
            form:{},
            multipleSelection:[],
            dialogFormVisible:false,
            options:[]
        }
    },
    created() {//生命钩子里面发送请求
        this.load()
    },
    methods:{
        reset(){
            this.name=""
            this.load()
        },
    
        load(){
            request.get("/menu",{
                params:{
                    name:this.name,
                }
            }).then(res=>{
               console.log(res)
                this.tableData=res.data//这里参数需要在浏览器的网络监视器里面去看
            })
        },
    
        save(){
            request.post("/menu",this.form).then(res=>{
                if (res.code==="200"){
                    this.$message.success("保存成功")//成功弹窗
                    this.dialogFormVisible=false
                    this.load()
                }else{
                    this.$message.error("保存失败")
                }
            })
        },
    
        handleDelete(id){
            request.delete("/menu/"+id).then(res=>{
                if (res.code==="200"){
                    this.$message.success("删除成功")//成功弹窗
                    this.dialogFormVisible=false
                    this.load()
                }else{
                    this.$message.error("删除失败")
                }
            })
        },
        handleSelectionChange(val){//这个函数用于选择返回多选框的id
            this.multipleSelection=val
        },
        delBatch(){
            let ids=this.multipleSelection.map(v=>v.id)//把对象是数组转换为纯数组
            request.post("/menu/del/batch",ids).then(res=>{
                if (res.code==="200"){
                    this.$message.success("批量删除成功")//成功弹窗
                    this.dialogFormVisible=false
                    this.load()
                }else{
                    this.$message.error("批量删除失败")
                }
            })
        },
       exp(){
         window.open('http://localhost:9090/menu/export')
       },
   
       handleExcelImportSuccess(){
          this.$message.success("文件导入成功")
          this.load()
       },
       
        handleEdit(row){
          this.form=row
          this.dialogFormVisible=true
          //请求图标数据
          request.get("/menu/icons").then(res=>{
            console.log(res)
            this.options=res.data
          })
        },
    
        handleAdd(id){
          this.dialogFormVisible=true
          this.form={}
          if (id){
            this.form.pid=id
          }
        },
    
        handleSizeChange(pageSize){
            this.pageSize=pageSize
            this.load()
        
        },
        handleCurrentChange(pageNum){
            this.pageNum=pageNum
            this.load()
        },
       
    }
}
</script>

<style>
.fontSize18{
  font-size: 30px;
}
.fontSize12{
  font-size: 12px;
}
</style>
