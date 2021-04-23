<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 警告栏区域 -->
    <el-alert title="添加商品信息" type="info" center show-icon :closable="false">
    </el-alert>
    <!-- 步骤条区域 -->
    <el-steps :space="200" :active="activeIndex - 0" finish-status="success" align-center>
      <el-step title="基本信息"></el-step>
      <el-step title="商品参数"></el-step>
      <el-step title="商品属性"></el-step>
      <el-step title="商品图片"></el-step>
      <el-step title="商品内容"></el-step>
      <el-step title="完成"></el-step>
    </el-steps>
    <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" label-position="top">
      <!-- 标签页区域 -->
      <el-tabs v-model="activeIndex" :tab-position="'left'" :before-leave="tabBeforeLeave" @tab-click="tabClick">
        <el-tab-pane label="基本信息" name="0">
          <el-form-item label="商品名称" prop="goods_name">
            <el-input v-model="addForm.goods_name"></el-input>
          </el-form-item>
          <el-form-item label="商品价格" prop="goods_price">
            <el-input v-model="addForm.goods_price" type="number"></el-input>
          </el-form-item>
          <el-form-item label="商品重量" prop="goods_weight">
            <el-input v-model="addForm.goods_weight" type="number"></el-input>
          </el-form-item>
          <el-form-item label="商品数量" prop="goods_number">
            <el-input v-model="addForm.goods_number" type="number"></el-input>
          </el-form-item>
          <el-form-item label="商品分类" prop="goods_cat">
            <el-cascader v-model="addForm.goods_cat" :options="cateList" :props="cateProps" @change="handleChange">
            </el-cascader>
          </el-form-item>
        </el-tab-pane>
        <el-tab-pane label="商品参数" name="1">
          <el-form-item :label="item.attr_name" v-for="item in manyTableData" :key="item.attr_id">
            <!-- 复选框组 -->
            <el-checkbox-group v-model="item.attr_vals">
              <el-checkbox :label="cb" v-for="(cb, i) in item.attr_vals" :key="i" border></el-checkbox>
            </el-checkbox-group>
          </el-form-item>
        </el-tab-pane>
        <el-tab-pane label="商品属性" name="2">
          <el-form-item :label="item.attr_name" v-for="item in onlyTableData" :key="item.attr_id">
            <el-input v-model="item.attr_vals"></el-input>
          </el-form-item>
        </el-tab-pane>
        <el-tab-pane label="商品图片" name="3">
          <!-- action: 图片上传的地址 -->
          <el-upload :action="uploadURL" :on-preview="handlePreview" :on-remove="handleRemove" list-type="picture"
            :headers="headerObj" :on-success="handleSuccess">
            <el-button size="small" type="primary">点击上传</el-button>
          </el-upload>
        </el-tab-pane>
        <el-tab-pane label="商品内容" name="4">
          <quill-editor v-model="addForm.goods_introduce" />
          <el-button type="primary" class="addBtn" @click="addGood()">添加商品</el-button>
        </el-tab-pane>
      </el-tabs>
    </el-form>
    <el-dialog title="图片预览" :visible.sync="previewImgVisible" width="50%">
      <img class="previewImg" :src="previewImgUrl" alt="" />
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'

export default {
  data () {
    return {
      // 步骤条激活的索引值
      activeIndex: '0',
      // 添加商品的表单对象
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        goods_cat: [],
        // 图片数组
        pics: [],
        // 富文本编辑器里的内容
        goods_introduce: '',
        // 商品的参数,包括动态参数和静态属性
        attrs: []
      },
      // 添加商品表单对象的验证规则
      addFormRules: {
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
        }],
        goods_number: [{
          required: true,
          message: '请输入商品数量',
          trigger: 'blur'
        }],
        goods_cat: [{
          required: true,
          message: '请选择商品分类',
          trigger: 'blur'
        }]
      },
      // 商品分类数组
      cateList: [],
      // 商品分类级联选择器数据源的相关配置
      cateProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 根据商品分类的id得到的商品动态参数数组
      manyTableData: [],
      // 根据商品分类的id得到的商品静态数组
      onlyTableData: [],
      // 图片上传的地址
      uploadURL: 'http://127.0.0.1:8888/api/private/v1/upload',
      // 上传图片的请求头header
      headerObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      // 预览图片的url
      previewImgUrl: '',
      // 预览图片对话框是否显示
      previewImgVisible: false
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    // 获取商品分类列表数据
    async getCateList () {
      const {
        data: res
      } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类数据失败')
      }
      this.cateList = res.data
    },
    // 级联选择器选中项改变时触发的事件
    handleChange () {
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = []
      }
    },
    // Tab标签页切换标签之前的钩子函数
    // activeName:要前往的标签的name属性值,
    // oldActiveName:当前的标签的那么属性值
    tabBeforeLeave (activeName, oldActiveName) {
      if (this.addForm.goods_cat.length !== 3 && oldActiveName === '0') {
        this.$message.error('请选择商品分类')
        return false
      }
    },
    // 点击Tab标签并选中时触发的事件
    async tabClick () {
      // console.log(this.activeIndex)
      // activeIndex等于1说明选中的是动态参数的标签项
      if (this.activeIndex === '1') {
        const {
          data: res
        } = await this.$http.get(`categories/${this.cateId}/attributes`, {
          params: {
            sel: 'many'
          }
        })
        if (res.meta.status !== 200) {
          return this.$message.error('获取参数列表失败')
        }
        // console.log(res)
        // 把attr_vals字符串切割为数组
        res.data.forEach(item => {
          item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
        })
        this.manyTableData = res.data
      } else if (this.activeIndex === '2') {
        const {
          data: res
        } = await this.$http.get(`categories/${this.cateId}/attributes`, {
          params: {
            sel: 'only'
          }
        })
        if (res.meta.status !== 200) {
          return this.$message.error('获取属性列表失败')
        }
        // console.log(res.data)
        this.onlyTableData = res.data
      }
    },
    // 点击已上传图片时触发的事件
    handlePreview (file) {
      // console.log(file)
      const imgUrl = file.response.data.url
      this.previewImgUrl = imgUrl
      this.previewImgVisible = true
    },
    // 移除图片时触发的事件
    handleRemove (file) {
      // console.log(file)
      // 获取移除的图片的tmp_path
      const tmpPath = file.response.data.tmp_path
      console.log(tmpPath)
      // 获得移除的图片在 addForm.pics 数组中的索引
      const index = this.addForm.pics.findIndex(x => x.pic === tmpPath)
      this.addForm.pics.splice(index, 1)
      // console.log(this.addForm.pics)
    },
    // 图片上传成功触发的事件
    handleSuccess (response) {
      // console.log(response)
      // 上传图片成功后,生成一个对象
      const picObj = {
        pic: response.data.tmp_path
      }
      // 把生成的对象push到 addForm.pics 数组中
      this.addForm.pics.push(picObj)
      console.log(this.addForm.pics)
    },
    // 添加商品
    addGood () {
      // console.log(this.addForm)
      // 校验表单
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('请添加必要的表单项')
        }
        // 校验成功则执行下面的代码
        // 深拷贝 addForm 对象
        const newForm = _.cloneDeep(this.addForm)
        newForm.goods_cat = newForm.goods_cat.join(',')
        // 处理表单里的动态参数，并 push 到 addForm.attrs 数组中
        this.manyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(' ')
          }
          this.addForm.attrs.push(newInfo)
        })
        // 处理表单里的静态属性，并 push 到 addForm.attrs 数组中
        this.onlyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals
          }
          this.addForm.attrs.push(newInfo)
        })
        newForm.attrs = this.addForm.attrs

        // 发起添加商品的请求
        const {
          data: res
        } = await this.$http.post('goods', newForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加商品失败')
        }
        this.$message.success('添加商品成功')
        this.$router.push('/goods')
      })
    }
  },
  computed: {
    // 选中的商品id
    cateId () {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      }
      return null
    }
  }
}
</script>

<style lang="less" scoped>
  .el-cascader {
    width: 300px;
  }

  .el-checkbox {
    margin: 0 10px 0 0 !important;
  }

  .previewImg {
    width: 100%;
  }

  .addBtn {
    margin-top: 15px;
  }
</style>
