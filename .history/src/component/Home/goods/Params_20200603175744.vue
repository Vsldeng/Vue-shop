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
          <el-button type="primary" size="mini" :disabled="isBtnDisabled">添加参数</el-button>
        </el-tab-pane>
        <!-- 添加静态属性的面板 -->
        <el-tab-pane label="静态属性" name="only">
          <el-button type="primary" size="mini" :disabled="isBtnDisabled">添加属性</el-button>
        </el-tab-pane>
      </el-tabs>
    </el-card>
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
      manyCateList:[],
      onlyCateList:[]
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
      const { data: res } = await this.$http.get(
        'categories/' + this.seletedCateKeys[2] + '/attributes',
        { params: { sel: this.activeName } }
      )
      if(res.meta.status!==200){
          this.$message.error('获取商品动态参数失败')
      }
      if(this.activeName==='many'){
          this.manyCateList=res.data
      }else{
          this.onlyCateList=res.data
      }
      console.log(res)
    },
    // tab页签点击事件的处理函数
    handleTabClick() {
      this.handleChange()
    }
  },
  computed: {
    isBtnDisabled() {
      if (this.seletedCateKeys.length !== 3) {
        return true
      }
      return false
    }
  }
}
</script>

<style lang="less" scoped>
.cat_opt {
  margin: 15px 0;
}
</style>