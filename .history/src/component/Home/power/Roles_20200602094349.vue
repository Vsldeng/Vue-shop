<template>
  <div class="rights">
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="handleAddRoles">添加角色</el-button>
        </el-col>
      </el-row>

      <el-table :data="rolesList" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
            <template v-slot:default="scope">
                <el-row v-for="(item1,i1) in scope.row.children" :key="item1.id" :class="['bdbottom',i1===0? 'bdtop':0,'vcenter']">
                    <!-- 渲染一级权限 -->
                    <el-col :span="5">
                        <el-tag closable @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                        <i class="el-icon-caret-right"></i>
                    </el-col>
                    <!-- 二级三级权限 -->
                    <el-col :span="19">
                        <el-row v-for="(item2,i2) in item1.children" :key="item2.id" :class="['bdbottom',i2===0? 'bdtop':0,'vcenter']">
                            <el-col :span="6">
                                <el-tag type="success" closable @close="removeRightById(scope.row,item2.id)">{{item2.authName}}</el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <el-col :span="18">
                                <el-tag type="warning" closable @close="removeRightById(scope.row,item3.id)" v-for="(item3,i3) in item2.children" :key="item3.id" :class="['bdbottom',i3===0? 'bdtop':0]">{{item3.authName}}</el-tag>
                            </el-col>
                        </el-row>
                    </el-col>
                </el-row>
            </template>
        </el-table-column>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" prop="level" width="350px">
          <template v-slot:default="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit" @click="handleRewriteRole(scope.row.id)">编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete" @click="handleDelete(scope.row.id)">删除</el-button>
            <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
<!-- 创建角色信息 -->
    <el-dialog title="添加角色" :visible.sync="RolesVision" width="50%" @close="rolesClosed">
      <!-- 内容主体区域 -->
      <el-form :model="addRolesList" :rules="addRolesRules" ref="RolesFormRef" label-width="70px">
        <el-form-item label="角色名" prop="roleName">
          <el-input v-model="addRolesList.roleName"></el-input>
        </el-form-item>
        <el-form-item label="描述" prop="roleDesc">
          <el-input v-model="addRolesList.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="rolesClosed">取 消</el-button>
        <el-button type="primary" @click="handleHandleAddRole">确 定</el-button>
      </span>
    </el-dialog>
<!-- 编辑角色信息 -->
    <el-dialog title="编辑角色" :visible.sync="RewriteRoleVision" width="50%" @close="RewriteRoleClosed">
      <!-- 内容主体区域 -->
      <el-form :model="RewriteRole" :rules="addRolesRules" ref="RewriteRoleRef" label-width="70px">
        <el-form-item label="角色名" prop="roleName">
          <el-input v-model="RewriteRole.roleName"></el-input>
        </el-form-item>
        <el-form-item label="描述" prop="roleDesc">
          <el-input v-model="RewriteRole.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="RewriteRoleClosed">取 消</el-button>
        <el-button type="primary" @click="handleHandleRewriteRole">确 定</el-button>
      </span>
    </el-dialog>
  <!-- 分配权限弹出框 -->
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%">
      <!-- 内容主体区域 -->
      <!-- 树形控件 -->
      <el-tree :data="rightslist" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defkeys"></el-tree>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible=false">取 消</el-button>
        <el-button type="primary" @click="setRightDialogVisible=false">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      rolesList: [],
      addRolesList: {
        roleName: '',
        roleDesc: ''
      },
      RewriteRole:{
          roleId:'',
          roleName:'',
          roleDesc:''
      },
      RolesVision: false,
      addRolesRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' }
        ],
      },
      RewriteRoleVision:false,
      //控制权限分配对话框
      setRightDialogVisible:false,
      //所有权限的数据
      rightslist:[],
      // 树形控件的属性绑定对象
      treeProps:{
        label:'authName',
        children:'children'
      },
      defkeys:[105,116]
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取列表失败')
      }
      this.rolesList = res.data
    },
    rolesClosed() {
      this.$refs.RolesFormRef.resetFields()
      this.RolesVision = false
    },
    handleAddRoles() {
      this.RolesVision = true
    },
    async handleHandleAddRole() {
      const { data: res } = await this.$http.post('roles', this.addRolesList)
      if (res.meta.status !== 201) {
        return this.$message.error('添加失败')
      }
      this.RolesVision=false
      this.getRolesList()
      this.$message.success('添加成功')
    },
    RewriteRoleClosed(){
      this.$refs.RewriteRoleRef.resetFields()
      this.RewriteRoleVision = false
    },
    async handleRewriteRole(id){
        this.RewriteRoleVision=true
        const {data:res} = await this.$http.get('roles/'+id);
        this.RewriteRole=res.data
    },
    handleHandleRewriteRole(){
        this.$refs.RewriteRoleRef.validate(async valid=>{
            if(!valid)  return
            const {data:res}= await this.$http.put('roles/'+this.RewriteRole.roleId,{
                roleName:this.RewriteRole.roleName,roleDesc:this.RewriteRole.roleDesc
            })
            if(res.meta.status!==200){
                return this.$message.error("修改失败")
            }
            this.RewriteRoleVision=false
            this.getRolesList()
            this.$message.success('修改成功');
        })
    },
    async handleDelete(id){
       const Removeres=await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err=>{
          return err
        })
        if(Removeres!=='confirm'){
          return this.$message.info('已取消删除')
        }
        const {data:res} = await this.$http.delete('roles/'+ id )
        if(res.meta.status!==200){
          return this.$message.error('删除失败')
        }
        this.getRolesList()
        this.$message.success('删除成功')
    },
    async removeRightById(role,rightId){
        const ress=await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err=>err)
        if(ress!=='confirm'){
            return this.$message.info('取消了删除')
        }
        const {data:res}=await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
        if(res.meta.status !==200){
            return this.$message.error('删除权限失败!')
        }
        role.children=res.data
    },
    //展示分配权限的对话框
    async showSetRightDialog(){
      // 获取所有权限数据
        const {data:res}=await this.$http.get('rights/tree')
        if(res.meta.status!==200){
            return this.$message.error("获取权限数据失败")
        }
        this.rightslist=res.data
        this.setRightDialogVisible=true;
    }
  }
}
</script>

<style lang="less" scoped>
.bdtop{
    border-top:1px solid #eee;
}
.bubottom{
    border-bottom:1px solid #eee;
}
.vcenter{
    display:flex;
    align-items:center;
}
</style>