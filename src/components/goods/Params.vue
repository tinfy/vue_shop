<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card>
      <el-alert title="注意:只允许为第三级分类设置相关参数!" type="warning" :closable="false" show-icon>
      </el-alert>
      <el-row class="cate_opt">
        <el-col>
          <span>选择商品分类：</span>
          <!-- options：数据源 -->
          <!-- props：让数据源中的数据怎么展示在级联框中 value：选项的实际值 label：选项的显示值 children：选项的子分类 -->
          <el-cascader v-model="selectedCateKeys" :options="cateList" :props="cateProps" @change="handleChange"
            popper-class="popper">
          </el-cascader>
        </el-col>
      </el-row>
      <!-- Tab标签页 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <el-tab-pane label="动态参数" name="many">
          <el-button type="primary" :disabled="isBtnDisabled" @click="addDialogVisible = true">添加参数</el-button>
          <!-- 动态参数表格 -->
          <el-table :data="manyTableData" border stripe>
            <el-table-column type="expand">
              <!-- Tag标签 -->
              <template slot-scope="scope">
                <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable @close="deleteAttr(i, scope.row)">
                  {{item}}</el-tag>
                <el-input class="input-new-tag" v-if="scope.row.inputVisible" v-model="scope.row.inputValue"
                  ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag
                </el-button>
              </template>
            </el-table-column>
            <el-table-column type="index" label="#">
            </el-table-column>
            <el-table-column prop="attr_name" label="参数名称">
            </el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑
                </el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteParams(scope.row.attr_id)">
                  删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <el-button type="primary" :disabled="isBtnDisabled" @click="addDialogVisible = true">添加属性</el-button>
          <!-- 静态属性表格 -->
          <el-table :data="onlyTableData" border stripe>
            <el-table-column type="expand">
              <!-- Tag标签 -->
              <template slot-scope="scope">
                <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable @close="deleteAttr(i, scope.row)">
                  {{item}}</el-tag>
                <el-input class="input-new-tag" v-if="scope.row.inputVisible" v-model="scope.row.inputValue"
                  ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag
                </el-button>
              </template>
            </el-table-column>
            <el-table-column type="index" label="#">
            </el-table-column>
            <el-table-column prop="attr_name" label="属性名称">
            </el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑
                </el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteParams(scope.row.attr_id)">删除
                </el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加参数对话框 -->
    <el-dialog :title="'添加' + titleText" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改参数对话框 -->
    <el-dialog :title="'添加' + titleText" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
      <el-form :model="editForm" :rules="addFormRules" ref="editFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 获取到的商品分类信息
      cateList: [],
      // 级联选择框选中的值对应的id
      selectedCateKeys: [],
      // 级联选择框选数据源的相关配置
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        expandTrigger: 'hover'
      },
      // Tab 标签页选中的值
      activeName: 'many',
      // 获取到的动态参数信息
      manyTableData: [],
      // 获取到的静态属性信息
      onlyTableData: [],
      // 是否显示添加参数对话框
      addDialogVisible: false,
      // 添加参数/属性的表单信息
      addForm: {
        attr_name: ''
      },
      // 添加参数/属性的验证规则
      addFormRules: {
        attr_name: [{
          required: true,
          message: '请输入参数名称',
          trigger: 'blur'
        }]
      },
      // 是否显示修改参数对话框
      editDialogVisible: false,
      // 修改参数/属性的表单信息
      editForm: {}
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    // 获取商品列表
    async getCateList () {
      const {
        data: res
      } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败')
      }
      this.cateList = res.data
      // console.log(this.cateList)
    },
    // 级联框选中的值发生改变时触发的事件
    handleChange () {
      this.getParamsData()
    },
    // tab 被选中时触发的事件
    handleTabClick () {
      this.getParamsData()
      // console.log(this.activeName)
    },
    // 获取参数信息
    async getParamsData () {
      // 选中的不是三级分类
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        // 清空动态参数和静态属性信息
        this.manyTableData = []
        this.onlyTableData = []
        return false
      }
      // console.log(this.selectedCateKeys)
      const {
        data: res
      } = await this.$http.get(`categories/${this.cateId}/attributes`, {
        params: {
          sel: this.activeName
        }
      })
      if (res.meta.status !== 200) {
        return false // this.$message.error('获取参数列表失败')
      }
      // 把attr_vals字符串转换为数组
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        // 为data里面的每个对象添加一个 inputVisible 属性和 inputValue 属性
        // 提供最后一个标签使用
        item.inputVisible = false
        item.inputValue = ''
      })
      // console.log(res.data)
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
    },
    // 添加参数对话框关闭触发的事件
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    // 点击“确定”按钮,添加参数
    addParams () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          return false
        }
        const {
          data: res
        } = await this.$http.post(`categories/${this.cateId}/attributes`, {
          attr_name: this.addForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 201) {
          return this.$message.error('添加参数失败')
        }
        this.$message.success('添加参数成功')
        this.getParamsData()
        this.addDialogVisible = false
      })
    },
    // 显示编辑参数对话框
    async showEditDialog (attrid) {
      const {
        data: res
      } = await this.$http.get(`categories/${this.cateId}/attributes/${attrid}`, {
        params: {
          attr_sel: this.activeName
        }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数失败')
      }
      this.editForm = res.data
      this.editDialogVisible = true
      // this.attrid = attrid
    },
    // 编辑参数对话框关闭触发的事件
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    // 点击“确定”按钮,修改参数
    editParams () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) {
          return false
        }
        const {
          data: res
        } = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`, {
          attr_name: this.editForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 200) {
          return this.$message.error('修改参数失败')
        }
        this.$message.success('修改参数成功')
        this.getParamsData()
        this.editDialogVisible = false
      })
    },
    // 删除参数
    async deleteParams (attrid) {
      const confirmResult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(error => error)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除参数')
      }
      const {
        data: res
      } = await this.$http.delete(`categories/${this.cateId}/attributes/${attrid}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除参数失败')
      }
      this.$message.success('删除参数成功')
      this.getParamsData()
    },
    // 文本框失去焦点或点击 Enter 键触发的事件
    handleInputConfirm (row) {
      // 如果文本框中输入的是空格,则重置文本框的内容
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return false
      }
      // 如果不为空,则把文本框中的值存入到attr_vals数组中
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      // 把attr_vals数组中的值存到数据库中
      this.saveAttr(row)
    },
    // 点击Tag标签,显示文本框
    showInput (row) {
      row.inputVisible = true
      // $nextTick函数在页面元素被重新渲染时触发，调用里面的回调函数
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    // 删除参数项
    deleteAttr (i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttr(row)
    },
    // 把对参数项的修改保存到数据库中
    async saveAttr (row) {
      const {
        data: res
      } = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, {
        attr_name: row.attr_name,
        attr_sel: row.attr_sel,
        attr_vals: row.attr_vals.join(' ')
      })
      if (res.meta.status !== 200) {
        return this.$message.error('修改参数项失败')
      }
      this.$message.success('修改参数项成功')
    }
  },
  computed: {
    // 两个按钮是否被禁用
    isBtnDisabled () {
      if (this.selectedCateKeys.length !== 3) {
        return true
      }
      return false
    },
    // 级联选择框的第三级标签的id
    cateId () {
      if (this.selectedCateKeys.length === 3) {
        return this.selectedCateKeys[2]
      }
      return null
    },
    // 添加参数对话框的标题
    titleText () {
      if (this.activeName === 'many') {
        return '动态参数'
      }
      return '静态属性'
    }
  }
}
</script>

<style lang="less" scoped>
  .cate_opt {
    margin: 15px 0;
  }

  .el-cascader {
    width: 300px;
  }

  .popper {
    max-height: 200px;
    overflow: scroll;
  }

  .el-tag {
    margin: 10px;
  }

  .input-new-tag {
    width: 100px;
  }
</style>
