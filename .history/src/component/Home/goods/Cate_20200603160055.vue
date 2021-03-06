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
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>

      <!-- 表格 -->
      <el-table
        :data="cateList"
        style="width: 100%;margin-bottom: 20px;"
        row-key="cat_id"
        border
        :tree-props="{children: 'children', hasChildren: 'hasChildren'}"
      >
        <el-table-column prop="cat_name" label="分类名称" sortable width="180"></el-table-column>
        <el-table-column prop="cat_deleted" label="是否有效" sortable width="180">
          <template v-slot:default="scope">
            <i
              :class="scope.row.cat_deleted? 'el-icon-success':'el-icon-error'"
              :style="scope.row.cat_deleted? 'color:lightgreen':'color:red'"
            ></i>
          </template>
        </el-table-column>
        <el-table-column prop="cat_pid" label="排序" sortable width="180">
          <template v-slot:default="scope">
            <el-tag size="mini" v-if="scope.row.cat_level===0">一级</el-tag>
            <el-tag type="success" size="mini" v-else-if="scope.row.cat_level===1">二级</el-tag>
            <el-tag type="warning" size="mini" v-else>三级</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template v-slot:default="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="handleEdit(scope.row)"
            >编辑</el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="handleDelete(scope.row)"
            >删除</el-button>
          </template>
        </el-table-column>
      </el-table>

      <!--分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="querInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="querInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>

    <!-- 添加分类对话框 -->
    <el-dialog title="添加分类" :visible.sync="addCateVisible" width="50%" @close="addCateDialogClosed">
      <el-form
        :model="addCateForm"
        :rules="addCateFormRules"
        ref="addCateFormRef"
        label-width="100px"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="{ expandTrigger: 'hover',value:'cat_id',label:'cat_name',children:'children'}"
            @change="parentCateChanged"
            clearable
            change-on-select
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span class="dialog-footer" slot="footer">
        <el-button @click="addCateVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 删除操作对话框 -->
    <el-dialog
      title="提示"
      :visible.sync="deleteCateDialog"
      width="50%"
      @close="deleteCateDialogClosed"
    >
      <span>你确定删除该({{deleteCateList.cat_name}})分类吗?</span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="deleteCateDialog = false">取 消</el-button>
        <el-button type="primary" @click="deleteCate">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑操作对话框 -->
    <el-dialog
      title="编辑分类"
      :visible.sync="editCateDialog"
      width="50%"
      @close="editCateDialogClosed"
    >
      <el-form :model="editCateList" :rules="editCateRules" ref="editCateRef" label-width="100px">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="editCateList.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editCateDialog = false">取 消</el-button>
        <el-button type="primary" @click="editCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      querInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      cateList: [],
      total: 0,
      addCateVisible: false,
      addCateForm: {
        cat_name: '',
        cat_pid: 0,
        //默认添加1级分类
        cat_level: 0
      },
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      editCateRules: {
        cat_name: [
          { required: true, message: '请编辑分类名称', trigger: 'blur' }
        ]
      },
      parentCateList: [],
      selectedKeys: [],
      deleteCateDialog: false,
      deleteCateList: [],

      editCateDialog: false,
      editCateList: []
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.querInfo
      })
      if (res.meta.status !== 200) {
        this.$message.error('获取商品分类失败')
      }
      this.cateList = res.data.result
      this.total = res.data.total
      // console.log(this.cateList)
    },
    handleDelete(scope) {
      this.deleteCateDialog = true
      this.deleteCateList = scope
    },
    async deleteCate() {
      const { data: res } = await this.$http.delete(
        'categories/' + this.deleteCateList.cat_id
      )
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败')
      }
      this.deleteCateDialog = false
      this.$message.success('删除成功')
      this.getCateList()
    },

    handleEdit(scope) {
      this.editCateDialog = true
      this.editCateList = scope
    },
    editCate() {
      this.$refs.editCateRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          'categories/' + this.editCateList.cat_id,
        //   this.editCateList
          {params:{cat_name:this.editCateList}}
        )
        console.log(this.editCateList)
        if (res.meta.status !== 200) {
            console.log(res)
          return this.$message.error('编辑分类信息失败')
        }
        this.editCateDialog = false
        this.$message.success('编辑成功')
        this.getCateList()
      })
    },
    editCateDialogClosed() {
      this.$refs.editCateRef.resetFields()
    },
    deleteCateDialogClosed() {
      this.deleteCateList = []
    },
    handleSizeChange(newSize) {
      this.querInfo.pagesize = newSize
      this.getCateList()
    },
    handleCurrentChange(newPage) {
      this.querInfo.pagenum = newPage
      this.getCateList()
    },
    showAddCateDialog() {
      this.getParentCateList()
      this.addCateVisible = true
    },
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类失败')
      }
      this.parentCateList = res.data
    },
    parentCateChanged() {
      if (this.selectedKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectedKeys[
          this.selectedKeys.length - 1
        ]
        this.addCateForm.cat_level = this.selectedKeys.length
        return
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level
      }
    },
    addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(
          'categories',
          this.addCateForm
        )
        if (res.meta.status !== 201) {
          this.$message.error('添加分类失败')
        }
        this.$message.success('添加分类成功')
        this.getCateList()
        this.addCateVisible = false
      })
    },
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    }
  }
}
</script>

<style lang="less" scoped>
</style>