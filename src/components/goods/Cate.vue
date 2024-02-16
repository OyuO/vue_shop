<script>
export default {
  name: 'Cate',
  data () {
    return {
      addDialogVisible: false,
      cateList: [],
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          type: 'template',
          template: 'isok'
        }
      ],
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      total: 0
    }
  },
  methods: {
    async getCategories () {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败!')
      }
      this.cateList = res.data.result
      this.total = res.data.total
      console.log(this.cateList)
    },
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getCategories()
    },
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getCategories()
    }
  },
  created () {
    this.getCategories()
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
      <el-button type="primary" @click="addDialogVisible = true">添加分类</el-button>
      <el-row></el-row>
      <tree-table :selection-type="false" :expand-type="false" show-index index-text="#" border :show-row-hover="false"
                  :data="cateList"
                  :columns="columns">
        <template slot="isok" slot-scope="scope">
          <i class="el-icon-success" v-if="scope.row.cat_deleted == false" style="color: lightgreen"></i>
          <i class="el-icon-error" v-if="scope.row.cat_deleted == true" style="color: red"></i>
        </template>
      </tree-table>
    </el-card>
  </div>
</template>

<style scoped lang="less">
.el-button {
  margin-bottom: 20px;
}
</style>
