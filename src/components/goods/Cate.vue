<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showaddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 表格区域 -->
      <tree-table class="treetable" :data="cateList" :columns="columns" show-index index-text="#" border
        :show-row-hover="false" :selection-type="false" :expand-type="false">
        <template slot="isok" slot-scope="scope">
          <i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color: lightgreen;"></i>
          <i class="el-icon-error" v-else style="color: red;"></i>
        </template>
        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level === 0" size="mini">一级</el-tag>
          <el-tag type="success" v-else-if="scope.row.cat_level === 1" size="mini">二级</el-tag>
          <el-tag type="warning" v-else size="mini">三级</el-tag>
        </template>
        <template slot="opt" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini" @click="openEditCateDialog(scope.row.cat_id)">编辑
          </el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteCate(scope.row.cat_id)">删除
          </el-button>
        </template>
      </tree-table>
      <!-- 分页区域 -->
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum" :page-sizes="[3, 5, 10, 15]" :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper" :total="totalnum">
      </el-pagination>
    </el-card>
    <!-- 添加分类对话框 -->
    <el-dialog title="添加分类" :visible.sync="showaddCateDialogVisible" width="50%" @close="closeAddCateDialog">
      <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <!-- v-model用于和显示框里面的值双向绑定，需要定义为一个数组 -->
          <!-- option用于指定数据源 -->
          <!-- props用于配置级联选择器相关属性 -->
          <el-cascader v-model="selectedKeys" :options="parentCateList" :props="cascaderProp"
            @change="selectedKeyChange" clearable>
          </el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="showaddCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改分类对话框 -->
    <el-dialog title="修改分类" :visible.sync="showEditCateDialogVisible" width="50%" @close="closeEditCateDialog">
      <el-form :model="editCateForm" :rules="addCateFormRules" ref="editCateFormRef" label-width="100px">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="showEditCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCateInfo()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 获取商品分类需要的参数
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 所有的商品分类组成的数组
      cateList: [],
      // 共有多少个商品分类
      totalnum: 0,
      // 为tree-table绑定列的相关属性
      columns: [{
        label: '分类名称',
        prop: 'cat_name'
      }, {
        label: '是否有效',
        // 表示这一列是模板列
        type: 'template',
        // 模板的名字
        template: 'isok'
      }, {
        label: '排序',
        // 表示这一列是模板列
        type: 'template',
        // 模板的名字
        template: 'order'
      }, {
        label: '操作',
        // 表示这一列是模板列
        type: 'template',
        // 模板的名字
        template: 'opt'
      }],
      // 是否显示添加分类对话框
      showaddCateDialogVisible: false,
      // 添加分类需要的参数对象
      addCateForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      // 添加分类时的验证规则
      addCateFormRules: {
        cat_name: [{
          required: true,
          message: '请输入分类名称',
          trigger: 'blur'
        }]
      },
      // 一二级商品分类组成的数组
      parentCateList: [],
      // 级联选择器选中的值
      selectedKeys: [],
      // 级联选择器的相关属性
      cascaderProp: {
        expandTrigger: 'hover',
        checkStrictly: true,
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 是否显示编辑分类的对话框
      showEditCateDialogVisible: false,
      // 根据id查询到的分类信息
      editCateForm: {}
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    // 获取所有的商品分类
    async getCateList () {
      const {
        data: res
      } = await this.$http.get('categories', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类数据失败')
      }
      this.cateList = res.data.result
      this.totalnum = res.data.total
    },
    // page size（每页显示多少条数据）改变触发的事件
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // page num（当前页码值）改变触发的事件
    handleCurrentChange (newNum) {
      this.queryInfo.pagenum = newNum
      this.getCateList()
    },
    // 点击按钮,显示添加分类对话框
    showaddCateDialog () {
      this.getParentCateList()
      this.showaddCateDialogVisible = true
    },
    // 获取一二级商品分类
    async getParentCateList () {
      const {
        data: res
      } = await this.$http.get('categories', {
        params: {
          type: 2
        }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级商品分类失败')
      }
      // console.log(res.data)
      this.parentCateList = res.data
    },
    // 级联选择器选中的值发生改变时触发的事件
    selectedKeyChange () {
      if (this.selectedKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
      // console.log(this.selectedKeys)
    },
    // 点击确定,添加分类
    addCate () {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return false
        const {
          data: res
        } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败')
        }
        this.$message.success('添加分类成功')
        this.getCateList()
        this.showaddCateDialogVisible = false
      })
    },
    // 关闭添加分类对话框
    closeAddCateDialog () {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    },
    // 修改分类对话框
    async openEditCateDialog (id) {
      const {
        data: res
      } = await this.$http.get(`categories/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败')
      }
      this.editCateForm = res.data
      this.showEditCateDialogVisible = true
    },
    // 点击"确认"按钮,修改分类信息
    editCateInfo () {
      this.$refs.editCateFormRef.validate(async valid => {
        if (!valid) return false
        const {
          data: res
        } = await this.$http.put(`categories/${this.editCateForm.cat_id}`, {
          cat_name: this.editCateForm.cat_name
        })
        if (res.meta.status !== 200) {
          return this.$message.error('修改商品分类信息失败')
        }
        this.$message.success('修改商品分类信息成功')
        this.getCateList()
        this.showEditCateDialogVisible = false
      })
    },
    // 关闭修改分类对话框触发的事件
    closeEditCateDialog () {
      this.$refs.editCateFormRef.resetFields()
    },
    // 删除分类
    async deleteCate (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该分类, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(error => error)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除分类')
      }
      const { data: res } = await this.$http.delete(`categories/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除分类失败')
      }
      this.$message.success('删除分类成功')
      this.getCateList()
    }
  }
}
</script>

<style lang="less">
  .treetable {
    margin-top: 15px;
  }

  .el-cascader {
    width: 100%;
  }
</style>
