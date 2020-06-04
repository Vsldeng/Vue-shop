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
            <el-alert
                title="注意：只允许为第三方分类设置相关参数！"
                type="warning"
                :closable="false"
                show-icon>
            </el-alert>

            <!-- 选择商品分类区域 -->
            <el-row class="cat_opt">
                <el-col>
                    <span>选择商品分类</span>
                    <el-cascader
                        v-model="seletedCateKeys"
                        :options="cateList"
                        :props="{ expandTrigger: 'hover' ,value:'cat_id',label:'cat_name',children:'children'}"
                        @change="handleChange">
                    </el-cascader>
                </el-col>
            </el-row>
        </el-card>
    </div>
</template>

<script>
export default {
    data(){
        return{
            cateList:[],
            seletedCateKeys:[]
        }
    },
    created(){
        this.getCateList()
    },
    methods:{
        async getCateList(){
            const {data:res}=this.$http.get('categories')
            if(res.meta.status!==200){
                return this.$message.error('获取商品分配失败')
            }
            this.cateList=res.data
            
        },
        // 商品分类级联框改变时调用
        handleChange(){
            console.log(this.seletedCateKeys)
        }
    }
}
</script>

<style lang="less" scoped>
.cat_opt{
    margin:15px 0;
}
</style>