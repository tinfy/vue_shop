<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryParams.query" clearable @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="6">
          <el-button type="primary" @click="dialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>
      <!-- 表格区域 -->
      <el-table :data="users" stripe border>
        <el-table-column type="index" label="#">
        </el-table-column>
        <el-table-column prop="username" label="姓名">
        </el-table-column>
        <el-table-column prop="email" label="邮箱">
        </el-table-column>
        <el-table-column prop="mobile" label="电话">
        </el-table-column>
        <el-table-column prop="role_name" label="角色">
        </el-table-column>
        <el-table-column label="状态">
          <!-- 作用域插槽 -->
          <template slot-scope="scope">
            <el-switch v-model="scope.row.mg_state" @change="userStateChange(scope.row)">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)"></el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteUserById(scope.row.id)">
            </el-button>
            <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
              <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
        :current-page="queryParams.pagenum" :page-sizes="[1, 2, 5, 10]" :page-size="queryParams.pagesize"
        layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </el-card>
    <!-- 添加用户对话框区域 -->
    <el-dialog title="添加用户" :visible.sync="dialogVisible" width="50%" @close="addDialogClosed">
      <!-- 表单区域 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <!-- 对话框的按钮区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改用户对话框区域 -->
    <el-dialog title="修改用户" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
      <!-- 表单区域 -->
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
        <el-form-item label="用户名">
          <el-input v-model="editForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <!-- 对话框的按钮区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    // 验证邮箱的规则
    var checkEmail = (rule, value, callback) => {
      const regEmail = /^[A-Za-z\d]+([-_.][A-Za-z\d]+)*@([A-Za-z\d]+[-.])+[A-Za-z\d]{2,4}$/
      if (regEmail.test(value)) {
        return callback()
      }
      return callback(new Error('请输入合法的邮箱'))
    }
    // 验证手机号的规则
    var checkMobile = (rule, value, callback) => {
      const regMobile = /^1[3|4|5|8][0-9]\d{4,8}$/
      if (regMobile.test(value)) {
        return callback()
      }
      return callback(new Error('请输入合法的手机号'))
    }
    return {
      // 获得用户列表数据的参数对象
      queryParams: {
        // 查询条件
        query: '',
        // 当前页码
        pagenum: 1,
        // 每页显示条数
        pagesize: 2
      },
      // 用户列表集合
      users: [],
      // 总记录数
      total: 0,
      // 是否显示添加用户对话框
      dialogVisible: false,
      // 添加用户的表单数据
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 添加用户表单的验证规则
      addFormRules: {
        username: [{
          required: true,
          message: '请输入用户名',
          trigger: 'blur'
        },
        {
          min: 5,
          max: 8,
          message: '长度在 5 到 8 个字符',
          trigger: 'blur'
        }
        ],
        password: [{
          required: true,
          message: '请输入密码',
          trigger: 'blur'
        },
        {
          min: 6,
          max: 12,
          message: '长度在 6 到 12 个字符',
          trigger: 'blur'
        }
        ],
        email: [{
          required: true,
          message: '请输入邮箱',
          trigger: 'blur'
        },
        {
          validator: checkEmail,
          trigger: 'blur'
        }
        ],
        mobile: [{
          required: true,
          message: '请输入手机号',
          trigger: 'blur'
        },
        {
          validator: checkMobile,
          trigger: 'blur'
        }
        ]
      },
      // 是否显示修改用户对话框
      editDialogVisible: false,
      // 根据id查询到的数据对象
      editForm: {},
      // 修改用户表单的验证规则
      editFormRules: {
        email: [{
          required: true,
          message: '请输入邮箱',
          trigger: 'blur'
        },
        {
          validator: checkEmail,
          trigger: 'blur'
        }
        ],
        mobile: [{
          required: true,
          message: '请输入手机号',
          trigger: 'blur'
        },
        {
          validator: checkMobile,
          trigger: 'blur'
        }
        ]
      }
    }
  },
  created () {
    this.getUserList()
  },
  methods: {
    // 获得用户列表数据
    async getUserList () {
      const {
        data: res
      } = await this.$http.get('users', {
        params: this.queryParams
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取用户列表失败！')
      }
      this.users = res.data.users
      this.total = res.data.total
      // console.log(res)
    },
    // 每页显示条数改变触发的事件
    handleSizeChange (newSize) {
      // console.log(newSize)
      this.queryParams.pagesize = newSize
      this.getUserList()
    },
    // 当前页码改变触发的事件
    handleCurrentChange (newPage) {
      // console.log(newPage)
      this.queryParams.pagenum = newPage
      this.getUserList()
    },
    // 用户状态改变触发的事件
    async userStateChange (userinfo) {
      // console.log(userinfo)
      const {
        data: res
      } = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
      if (res.meta.status !== 200) {
        userinfo.mg_state = !userinfo.mg_state
        return this.$message.error('修改用户状态失败！')
      }
      this.$message.success('修改用户状态成功！')
    },
    // 添加用户对话框关闭后清空表单里的内容
    addDialogClosed () {
      // this.$refs.addFormRef可以获取到表单对象
      this.$refs.addFormRef.resetFields()
    },
    // 点击表单里的确定按钮,添加新用户
    addUser () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return false
        // 校验成功,发起网络请求添加用户
        const {
          data: res
        } = await this.$http.post('users', this.addForm)
        if (res.meta.status !== 201) {
          this.$message.error('添加用户失败')
        }
        this.$message.success('添加用户成功')
        // 隐藏对话框
        this.dialogVisible = false
        // 刷新表格中的数据
        this.getUserList()
      })
    },
    // 点击修改按钮触发的事件
    async showEditDialog (id) {
      const {
        data: res
      } = await this.$http.get(`users/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('查询用户信息失败')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 修改用户对话框关闭后清空表单里的内容
    editDialogClosed () {
      // this.$refs.addFormRef可以获取到表单对象
      this.$refs.editFormRef.resetFields()
    },
    // 验证用户信息并提交
    editUserInfo () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return false
        // 发起修改用户信息的请求
        const {
          data: res
        } = await this.$http.put(`users/${this.editForm.id}`, {
          email: this.editForm.email,
          mobile: this.editForm.mobile
        })
        if (res.meta.status !== 200) {
          return this.$message.error('修改用户信息失败')
        }
        // 关闭对话框
        this.editDialogVisible = false
        // 刷新表格数据
        this.getUserList()
        // 提示用户信息修改成功
        this.$message.success('修改用户信息成功')
      })
    },
    // 根据id来删除用户信息
    async deleteUserById (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
        center: true
      }).catch(err => err)
      // 点击了"确定",返回字符串confirm
      // 点击了"取消",返回字符串cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete(`users/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败')
      }
      this.$message.success('删除用户成功')
      this.getUserList()
    }
  }
}
</script>

<style>
</style>
