<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="addRoleDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表区域 -->
      <el-table :data="rolesList" stripe border>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row :class="['borderbottom', i1 === 0 ? 'bordertop' : '', 'row-center']"
              v-for="(item1, i1) in scope.row.children" :key="item1.id">
              <!-- 一级权限 -->
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 二三级权限 -->
              <el-col :span="19">
                <el-row :class="[i2 === 0 ? '' : 'bordertop', 'row-center']" v-for="(item2, i2) in item1.children"
                  :key="item2.id">
                  <!-- 二级权限 -->
                  <el-col :span="6">
                    <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">{{item2.authName}}
                    </el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 三级权限 -->
                  <el-col :span="18">
                    <el-tag type="warning" v-for="item3 in item2.children" :key="item3.id" closable
                      @close="removeRightById(scope.row, item3.id)">
                      {{item3.authName}}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit" @click="openEditRoleDialog(scope.row.id)">编辑
            </el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete" @click="deleteRole(scope.row.id)">删除</el-button>
            <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加角色对话框 -->
    <el-dialog title="添加角色" :visible.sync="addRoleDialogVisible" width="50%" @close="restAddRoleDialog">
      <el-form ref="addRoleFormRef" :model="addRoleForm" :rules="addRoleFormRules" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="addRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改角色对话框 -->
    <el-dialog title="修改角色" :visible.sync="editRoleDialogVisible" width="50%" @close="editAddRoleDialog">
      <el-form ref="editRoleFormRef" :model="rolesList" :rules="addRoleFormRules" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="rolesList.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="rolesList.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRole">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限对话框 -->
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClosed">
      <!-- 树形控件 -->
      <el-tree :data="rightsList" :props="treeProps" ref="rightsListRef" show-checkbox node-key="id" default-expand-all
        :default-checked-keys="defaultKeys"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allocateRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 角色列表
      rolesList: [],
      // 添加角色的参数
      addRoleForm: {
        roleName: '',
        roleDesc: ''
      },
      // 添加角色的验证规则
      addRoleFormRules: {
        roleName: [{
          required: true,
          message: '请输入角色名称',
          trigger: 'blur'
        },
        {
          min: 2,
          max: 6,
          message: '长度在 2 到 6 个字符',
          trigger: 'blur'
        }
        ]
      },
      // 添加角色对话框是否显示
      addRoleDialogVisible: false,
      // 编辑角色对话框是否显示
      editRoleDialogVisible: false,
      // 分配权限对话框是否显示
      setRightDialogVisible: false,
      // 权限列表
      rightsList: [],
      // 树形控件属性的对应
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      // 默认勾选的权限的id
      defaultKeys: [],
      roleId: ''
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    // 获取角色列表
    getRolesList () {
      this.$http.get('roles')
        .then(res => {
          this.rolesList = res.data.data
        })
        .catch(() => this.$message.error('获取角色列表失败'))
    },
    // 关闭添加角色对话框后重置对话框中的内容
    restAddRoleDialog () {
      this.$refs.addRoleFormRef.resetFields()
    },
    // 添加角色
    addRole () {
      this.$refs.addRoleFormRef.validate(async valid => {
        if (!valid) return false
        const {
          data: res
        } = await this.$http.post('roles', {
          roleName: this.addRoleForm.roleName,
          roleDesc: this.addRoleForm.roleDesc
        })
        if (res.meta.status !== 201) {
          return this.$message.error('添加角色失败')
        }
        this.addRoleDialogVisible = false
        this.$message.success('添加角色成功')
        this.getRolesList()
      })
    },
    // 关闭修改角色对话框后重置对话框中的内容
    editAddRoleDialog () {
      this.$refs.editRoleFormRef.resetFields()
    },
    // 点击"编辑"按钮,打开编辑角色对话框
    async openEditRoleDialog (id) {
      const {
        data: res
      } = await this.$http.get(`roles/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色信息失败')
      }
      this.rolesList = res.data
      this.editRoleDialogVisible = true
    },
    // 编辑角色信息
    editRole () {
      this.$refs.editRoleFormRef.validate(async valid => {
        if (!valid) return false
        const {
          data: res
        } = await this.$http.put(`roles/${this.rolesList.roleId}`, {
          roleName: this.rolesList.roleName,
          roleDesc: this.rolesList.roleDesc
        })
        if (res.meta.status !== 200) {
          return this.$message.error('修改角色信息失败')
        }
        this.$message.success('修改角色信息成功')
        this.editRoleDialogVisible = false
        this.getRolesList()
      })
    },
    // 删除角色
    async deleteRole (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(error => error)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除角色')
      }
      const {
        data: res
      } = await this.$http.delete(`roles/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除角色失败')
      }
      this.$message.success('删除角色成功')
      this.getRolesList()
    },
    // 删除角色的权限
    async removeRightById (role, rightId) {
      const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(error => error)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除该权限')
      }
      const {
        data: res
      } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败')
      }
      // res.data是当前角色下的最新权限数据
      role.children = res.data
    },
    // 打开分配权限对话框
    async showSetRightDialog (role) {
      this.setRightDialogVisible = true
      const {
        data: res
      } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限列表失败')
      }
      this.getLeafKey(role, this.defaultKeys)
      this.rightsList = res.data
      // 记录当前打开的角色的id
      this.roleId = role.id
      // console.log(res)
    },
    // 递归获取所有三级权限的id放到defaultKeys数组中
    getLeafKey (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKey(item, arr)
      })
    },
    // 关闭分配权限对话框后清空defaultKeys数组
    setRightDialogClosed () {
      this.defaultKeys = []
    },
    // 在树形控件里点击“确定”分配权限
    async allocateRights () {
      const checkedKeys = [...this.$refs.rightsListRef.getCheckedKeys(),
        ...this.$refs.rightsListRef.getHalfCheckedKeys()
      ]
      const checkedKeysStr = checkedKeys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: checkedKeysStr })
      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败')
      }
      this.$message.success('分配权限成功')
      this.getRolesList()
      this.setRightDialogVisible = false
    }
  }
}
</script>

<style lang="less" scoped="scoped">
  .el-tag {
    margin: 7px;
  }

  .bordertop {
    border-top: 1px solid #eee;
  }

  .borderbottom {
    border-bottom: 1px solid #eee;
  }

  // 让一二级权限标签垂直居中
  .row-center {
    display: flex;
    align-items: center;
  }
</style>
