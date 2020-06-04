<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片 -->
    <el-card>
      <!-- 警告区域 -->
      <el-alert title="注意：只允许为第三方分类设置相关参数！" type="warning" :closable="false" show-icon></el-alert>

      <!-- 选择商品分类区域 -->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类</span>
          <el-cascader
            v-model="seletedCateKeys"
            :options="cateList"
            :props="{ expandTrigger: 'hover' ,value:'cat_id',label:'cat_name',children:'children'}"
            @change="handleChange"
            clearable
          ></el-cascader>
        </el-col>
      </el-row>
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!-- 添加动态参数的面板 -->
        <el-tab-pane label="动态参数" name="many">
          <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addParams">添加参数</el-button>
          <!-- 动态参数表格 -->
          <el-table :data="manyCateList" border stripe>
            <el-table-column type="expand"></el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template v-slot:default="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
                {{scope.row}}
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 添加静态属性的面板 -->
        <el-tab-pane label="静态属性" name="only">
          <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addParams">添加属性</el-button>
          <!-- 静态属性表格 -->
          <el-table :data="onlyCateList" border stripe>
            <el-table-column type="expand"></el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="属性名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template v-slot:default="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
                {{scope.row}}
              </template>
            </el-table-column>
          </el-table>          
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加参数的对话框 -->
    <el-dialog
      :title="'添加'+titleText"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addParamsClosed"
      >
      <el-form :model="addParamsList" :rules="paramsRules" ref="paramsRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addParamsList.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="handleAddParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      cateList: [],
      seletedCateKeys: [],
      // 被激活页签的名称
      activeName: 'many',
      manyCateList: [],
      onlyCateList: [],
      addDialogVisible:false,
      addParamsList:{
        attr_name:''
      },
      paramsRules:{
        attr_name:[
          {required:true,message:'请输入参数名称',trigger:'blur'}
        ]
      }
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分配失败')
      }
      this.cateList = res.data
    },
    // 商品分类级联框改变时调用
    async handleChange() {
      // 根据所选id和面板，从后台获取数据
      if(this.seletedCateKeys.length!==3){
        return 
      }
      const {
        data: res
      } = await this.$http.get(`categories/${this.cateId}/attributes`, {
        params: { sel: this.activeName }
      })
      if (res.meta.status !== 200) {
        this.$message.error('获取商品动态参数失败')
      }
      if (this.activeName === 'many') {
        this.manyCateList = res.data
      } else {
        this.onlyCateList = res.data
      }
      console.log(res)
    },
    // tab页签点击事件的处理函数
    handleTabClick() {
      this.handleChange()
    },
    addParams(){
      this.addDialogVisible=true
    },
    addParamsClosed(){
      this.$refs.paramsRef.resetFields()
    },
    handleAddParams(){
      this.$refs.paramsRef.validate(async valid=>{
        if(!valid) return 
        const {data:res}=await this.$http.post(`categories/${this.cateId}/attributes`,{
          attr_name:this.addParamsList.attr_name,
          attr_sel:this.activeName
        })
        if(res.meta.status!==201){
          this.$message.error('添加参数失败')
        }
        this.$message.success('添加参数成功')
        this.addDialogVisible=false
        this.handleChange()
      })
    }
  },
  computed: {
    isBtnDisabled() {
      if (this.seletedCateKeys.length !== 3) {
        return true
      }
      return false
    },
    cateId() {
      if (this.seletedCateKeys.length === 3) {
        return this.seletedCateKeys[2]
      }
      return null
    },
    titleText(){
      if(this.activeName==='many'){
        return '动态参数'
      }
      return '静态属性'
    }
  }
}
</script>

<style lang="less" scoped>
.cat_opt {
  margin: 15px 0;
}
</style>