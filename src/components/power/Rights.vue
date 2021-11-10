<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card>
      <el-table :data="rights" stripe border>
        <el-table-column type="index" label="#">
        </el-table-column>
        <el-table-column prop="authName" label="权限名称">
        </el-table-column>
        <el-table-column prop="path" label="路径">
        </el-table-column>
        <el-table-column label="权限等级" v-slot="{ row }">
          <el-tag v-if="row.level === '0'">一级</el-tag>
          <el-tag type="success" v-else-if="row.level === '1'">二级</el-tag>
          <el-tag type="warning" v-else>三级</el-tag>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 权限列表数组
      rights: [],
      reflect: {
        0: '一级',
        1: '二级',
        2: '三级'
      }
    }
  },
  created () {
    this.getRightsList()
  },
  methods: {
    // 获得权限列表
    async getRightsList () {
      const { data: res } = await this.$http.get('rights/list')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限列表失败')
      }
      this.rights = res.data
    }
  }
}
</script>

<style lang="less" scoped="scoped">
</style>
