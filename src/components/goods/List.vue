<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card>
      <!-- gutter: 栅格间隔  span：栅格占据的列数 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getGoodsList">
            <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="6">
          <el-button type="primary" @click="goAddPage">添加商品</el-button>
        </el-col>
      </el-row>
      <!-- 表格区域 -->
      <el-table :data="goodsList" stripe border>
        <el-table-column type="index" label="#">
        </el-table-column>
        <el-table-column prop="goods_name" label="商品名称">
        </el-table-column>
        <el-table-column prop="goods_price" label="商品价格(元)" width="120px">
        </el-table-column>
        <el-table-column prop="goods_weight" label="商品重量" width="90px">
        </el-table-column>
        <el-table-column prop="add_time" label="创建时间" width="150px">
          <template v-slot:default="slotProps">
            <!-- slotProps.row 代表这一行的数据 -->
            {{slotProps.row.add_time | dateFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="190px">
          <template v-slot:default="slotProps">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditGoodDialog(slotProps.row.goods_id)">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeById(slotProps.row.goods_id)">删除
            </el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum" :page-sizes="[10, 15, 20, 30]" :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </el-card>
    <el-dialog
      title="修改商品"
      :visible.sync="editGooddialogVisible"
      width="50%" @close="editGooddialogClosed">
      <el-form :model="editGoodForm" :rules="editGoodFormRules" ref="editGoodFormRef" label-width="120px">
        <el-form-item label="商品名称" prop="goods_name">
          <el-input v-model="editGoodForm.goods_name"></el-input>
        </el-form-item>
        <el-form-item label="商品价格(元)" prop="goods_price">
          <el-input v-model="editGoodForm.goods_price" type="number"></el-input>
        </el-form-item>
        <el-form-item label="商品重量" prop="goods_weight">
          <el-input v-model="editGoodForm.goods_weight" type="number"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editGooddialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editGood">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 查询参数
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      // 商品列表数组
      goodsList: [],
      // 总记录数
      total: 0,
      // 编辑商品对话框是否显示
      editGooddialogVisible: false,
      // 修改商品的表单对象
      editGoodForm: {},
      // 修改商品的表单对象的验证规则
      editGoodFormRules: {
        goods_name: [{
          required: true,
          message: '请输入商品名称',
          trigger: 'blur'
        }],
        goods_price: [{
          required: true,
          message: '请输入商品价格',
          trigger: 'blur'
        }],
        goods_weight: [{
          required: true,
          message: '请输入商品重量',
          trigger: 'blur'
        }]
      }
    }
  },
  created () {
    this.getGoodsList()
  },
  methods: {
    async getGoodsList () {
      const {
        data: res
      } = await this.$http.get('goods', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品列表数据失败')
      }
      // this.$message.success('获取商品列表成功')
      this.goodsList = res.data.goods
      this.total = res.data.total
    },
    // 每页显示条数发生改变时触发的事件
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getGoodsList()
    },
    // 当前页码值发生改变时触发的事件
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getGoodsList()
    },
    // 根据id删除商品
    async removeById (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该商品, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(error => error)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除该商品')
      }
      const { data: res } = await this.$http.delete(`goods/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除商品失败')
      }
      this.$message.success('删除商品成功')
      this.getGoodsList()
    },
    // 跳转添加商品页面
    goAddPage () {
      this.$router.push('/goods/add')
    },
    // 显示编辑商品对话框
    async showEditGoodDialog (id) {
      const { data: res } = await this.$http.get(`goods/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品数据失败')
      }
      this.editGoodForm = res.data
      // console.log(this.editGoodForm)
      this.editGooddialogVisible = true
    },
    // 编辑商品对话框关闭后触发的事件
    editGooddialogClosed () {
      this.$refs.editGoodFormRef.resetFields()
    },
    // 点击"确定",提交编辑商品的表单
    editGood () {
      this.$refs.editGoodFormRef.validate(async valid => {
        if (!valid) {
          return false
        }
        const { data: res } = await this.$http.put(`goods/${this.editGoodForm.goods_id}`, this.editGoodForm)
        // console.log(res.meta)
        if (res.meta.status !== 200) {
          return this.$message.error('修改商品数据失败')
        }
        this.$message.success('修改商品数据成功')
        this.getGoodsList()
        this.editGooddialogVisible = false
      })
    }
  }
}
</script>

<style lang="less" scoped>
</style>
