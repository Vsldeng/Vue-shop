<template>
    <div>
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品分类</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区域 -->
        <el-card>
            <el-row>
                <el-col :span="4">
                    <el-button type="primary">添加分类</el-button>
                </el-col>
            </el-row>

            <!-- 表格 -->
            
            <!--分页 -->
        </el-card>
    </div>
</template>

<script>
export default {
    data(){
        return{
            querInfo:{
                type:3,
                pagenum:1,
                pagesize:5
            },
            cateList:[],
            total:0
        }
    },
    created(){
        this.getCateList()
    },
    methods:{
        async getCateList(){
            const {data:res}=await this.$http.get('categories',{ params:this.querInfo })
            if(res.meta.status!==200){
                this.$message.error('获取商品分类失败')
            }
            this.cateList=res.data.result
            this.total=res.data.total
            console.log(res)
        }
    }
}
</script>

<style lang="less" scoped>

</style>