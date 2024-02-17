<script>
export default {
  name: 'Cate',
  data () {
    return {
      cateAddDialogVisible: false,
      cateEditDialogVisible: false,
      cateList: [],
      parentCateList: [],
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          type: 'template',
          template: 'isok'
        },
        {
          label: '排序',
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          type: 'template',
          template: 'opt'
        }
      ],
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      cateAddForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      cateEditForm: {
        cat_id: 0,
        cat_name: ''
      },
      cateAddRules: {
        cat_name: [
          {
            required: true,
            message: '请输入分类名称',
            trigger: 'blur'
          }]
      },
      cateEditRules: {
        cat_name: [
          {
            required: true,
            message: '请输入分类名称',
            trigger: 'blur'
          }]
      },
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      selectedKeys: []
    }
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败!')
      }
      this.cateList = res.data.result
      this.total = res.data.total
    },
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类数据失败!')
      }
      this.parentCateList = res.data
    },
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    showAddCateDialog () {
      this.getParentCateList()
      this.cateAddDialogVisible = true
    },
    parentCateChanged () {
      if (this.selectedKeys.length > 0) {
        this.cateAddForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        this.cateAddForm.cat_level = this.selectedKeys.length
      } else {
        this.cateAddForm.cat_pid = 0
        this.cateAddForm.cat_level = 0
      }
    },
    addCate () {
      this.$refs.cateAddFormRef.validate(async valid => {
        if (!valid) return

        const { data: res } = await this.$http.post('categories', this.cateAddForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败!')
        }
        this.$message.success('添加分类成功!')
        this.getCateList()
        this.cateAddDialogVisible = false
      })
    },
    editCate () {
      this.$refs.cateEditFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(`categories/${this.cateEditForm.cat_id}`, this.cateEditForm)
        if (res.meta.status !== 200) {
          return this.$message.error('修改分类失败!')
        }
        this.$message.success('修改分类成功!')
        this.getCateList()
        this.cateEditDialogVisible = false
      })
    },
    async deleteCate (id) {
      const { data: res } = await this.$http.delete(`categories/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除分类失败!')
      }
      this.$message.success('删除分类成功!')
      this.getCateList()
    },
    cateAddDialogClosed () {
      this.$refs.cateAddFormRef.resetFields()
      this.selectedKeys = []
      this.cateAddForm.cat_level = 0
      this.cateAddForm.cat_pid = 0
    },
    cateEditDialogClosed () {
      this.$refs.cateEditFormRef.resetFields()
    },
    showCateEditDialog (row) {
      this.cateEditForm.cat_id = row.cat_id
      this.cateEditForm.cat_name = row.cat_name
      this.cateEditDialogVisible = true
    }
  },
  created () {
    this.getCateList()
  }
}
</script>

<template>
  <!--面包屑导航区域-->
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片视图区域-->
    <el-card>
      <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
      <el-row></el-row>
      <tree-table class="treeTable" :selection-type="false" :expand-type="false" show-index index-text="#" border
                  :show-row-hover="false"
                  :data="cateList"
                  :columns="columns">
        <template slot="isok" slot-scope="scope">
          <i class="el-icon-success" v-if="scope.row.cat_deleted == false" style="color: lightgreen"></i>
          <i class="el-icon-error" v-if="scope.row.cat_deleted == true" style="color: red"></i>
        </template>
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag type="success" size="mini" v-else-if="scope.row.cat_level === 1">二级</el-tag>
          <el-tag type="warning" size="mini" v-else-if="scope.row.cat_level === 2">三级</el-tag>
        </template>
        <template slot="opt" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini" @click="showCateEditDialog(scope.row)">编辑
          </el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteCate(scope.row.cat_id)">删除
          </el-button>
        </template>
      </tree-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 20, 50]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>
    <el-dialog title="添加分类" :visible.sync="cateAddDialogVisible" width="50%" @close="cateAddDialogClosed">
      <el-form :rules="cateAddRules" ref="cateAddFormRef" :model="cateAddForm" label-width="100px">
        <el-form-item label="分类名称: " prop="cat_name">
          <el-input v-model="cateAddForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类: ">
          <el-cascader :options="parentCateList" :props="cascaderProps" v-model="selectedKeys"
                       @change="parentCateChanged" clearable change-on-select>
          </el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="cateAddDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
    <el-dialog title="修改分类" :visible.sync="cateEditDialogVisible" width="50%" @close="cateEditDialogClosed">
      <el-form :rules="cateEditRules" ref="cateEditFormRef" :model="cateEditForm" label-width="100px">
        <el-form-item label="分类名称: " prop="cat_name">
          <el-input v-model="cateEditForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="cateEditDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<style scoped lang="less">
.treeTable {
  margin-top: 15px;
}

.el-cascader {
  width: 100%;
}
</style>
