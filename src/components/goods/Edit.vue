<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>修改商品</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 警告区域 -->
      <el-alert title="修改商品信息" type="info" center show-icon :closable="false"></el-alert>
      <!-- 步骤区域 -->
      <el-steps :space="200" :active="activeIndex - 0" finish-status="success" align-center>
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <!-- tab栏区域 -->
      <el-form
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="100px"
        label-position="top"
      >
        <el-tabs
          v-model="activeIndex"
          :tab-position="'left'"
          :before-leave="beforeTabLeave"
          @tab-click="tabClicked"
        >
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="editForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="editForm.goods_price" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="editForm.goods_weight" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="editForm.goods_number" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <el-cascader
                v-model="editForm.goods_cat"
                :options="catelist"
                :props="{ expandTrigger: 'hover',label:cateProps.label,value:cateProps.value,children:cateProps.children}"
                @change="handleChange"
              ></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <!-- 渲染表单的Item项 -->
            <el-form-item
              :label="item.attr_name"
              v-for="item in manyTableData "
              :key="item.attr_id"
            >
              <!-- 复选框组 -->
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox border :label="cb" v-for="(cb,i) in item.attr_vals" :key="i"></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item :label="item.attr_name" v-for="item in onlyTableData" :key="item.attr_id">
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <!-- action 代表后台上传的地址-->
            <el-upload
              :action="actionUrl"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              list-type="picture"
              :headers="headerObj"
              :on-success="handleSuccess"
            >
              <el-button size="small" type="primary">点击上传</el-button>
              <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <!-- 富文本编辑器组件 -->
            <quill-editor v-model="editForm.goods_introduce"></quill-editor>
            <!-- 修改商品按钮 -->
            <el-button type="primary" class="btnEdit" @click="edit">修改商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
    <!-- 浏览图片对话框 -->
    <el-dialog title="浏览图片" :visible.sync="previewVisible" width="50%">
      <img :src="previewPath" alt class="previewImg" />
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'
export default {
  data() {
    return {
      //步骤条的索引
      activeIndex: '0',
      //修改商品表单数据对象
      editForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        //商品分类所属的分类数组
        goods_cat: [],
        //商品图片的临时路径
        pics: [],
        //富文本内容
        goods_introduce: '',
        //商品的参数
        attrs: []
      },
      //修改商品表单验证规则对象
      editFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' }
        ],
        goods_cat: [
          { required: true, message: '请选择商品分类', trigger: 'blur' }
        ]
      },
      //商品分类列表
      catelist: [],
      cateProps: {
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      //动态参数列表数据
      manyTableData: [],
      //静态属性列表数据
      onlyTableData: [],
      //图片上传的URL地址
      actionUrl: 'http://127.0.0.1:8888/api/private/v1/upload',
      headerObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      //预览图片地址
      previewPath: '',
      //控制浏览图片对话框的显示与隐藏
      previewVisible: false
    }
  },
  created() {
    this.getCateList()
    this.findGoods()
  },
  methods: {
    //获取所有商品分类数据
    async getCateList() {
      const { data } = await this.$http.get('categories')
      if (data.meta.status !== 200) {
        return this.$message.error('获取商品分类数据失败')
      }
      this.catelist = data.data
      // console.log(this.catelist)
    },
    //级联选择器选中项变化，会触发这个函数
    handleChange() {
      // console.log(this.editForm.goods_cat)
      if (this.editForm.goods_cat.length !== 3) {
        this.editForm.goods_cat.length = []
      }
    },
    beforeTabLeave(activeName, oldActiveName) {
      if (oldActiveName === '0' && this.editForm.goods_cat.length !== 3) {
        this.$message.error('请选择商品分类')
        return false
      }
    },
    //tab栏切换触发
    tabClicked() {
      //证明访问的时动态参数面板
      if (this.activeIndex === '1') {
        //获取动态参数
        this.manyCateGories()
      } else if (this.activeIndex === '2') {
        //获取静态属性
        this.onlyCateGories()
      }
    },
    //获取动态参数列表
    async manyCateGories() {
      const { data } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        {
          params: { sel: 'many' }
        }
      )

      if (data.meta.status !== 200) {
        this.$message.error('获取动态参数列表失败')
      }
      data.data.forEach(item => {
        item.attr_vals =
          item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
      })
      this.manyTableData = data.data
      // console.log(this.manyTableData);
    },
    //获取静态属性列表
    async onlyCateGories() {
      const { data } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        {
          params: { sel: 'only' }
        }
      )

      if (data.meta.status !== 200) {
        this.$message.error('获取静态参数列表失败')
      }
      this.onlyTableData = data.data
      // console.log(this.onlyTableData)
    },
    //处理图片浏览效果
    handlePreview(file) {
      // console.log(file);
      this.previewPath = file.response.data.url
      this.previewVisible = true
    },
    //处理图片移除操作
    handleRemove(file) {
      //1.获取将要删除的图片的临时路经
      const filePath = file.response.data.tmp_path
      //2.从pics数组中，找到这个图片对应的索引值
      const i = this.editForm.pics.findIndex(x => x.pic === filePath)
      //3.调用数组的splice方法，把图片信息对象，从pics数组中移除
      this.editForm.pics.splice(i, 1)
      // console.log(this.editForm);
    },
    //图片上传成功后
    handleSuccess(response) {
      //1.拼接得到一个图片信息对象
      const picInfo = { pic: response.data.tmp_path }
      //2.将图片信息对象，push到pics数组中
      this.editForm.pics.push(picInfo)
      // console.log(this.editForm);
    },
    //查找商品
    async findGoods() {
      // console.log(id);
      const goodsId = window.sessionStorage.getItem('goodsId')
      // console.log(goodsId);
      const { data } = await this.$http.get('goods/' + goodsId)
      // console.log(data);

      if (data.meta.status !== 200) {
        return this.$message.error('获取商品数据失败')
      }
      data.data.goods_cat = data.data.goods_cat.split(',').map(Number)
      //  newGoodsCat.map(item =>{
      //     this.goods_cat.push(+item)
      //  })
      // console.log(daa.data);
      // console.log(this.goods_cat);

      this.editForm = data.data
    },
    //修改商品
    edit() {
      // console.log(this.editForm);
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('请填写必要的表单项')
        }
        //执行修改的业务逻辑
        const form = _.cloneDeep(this.editForm)
        form.goods_cat = form.goods_cat.join(',')

        //处理动态参数
        this.manyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(' ')
          }
          this.editForm.attrs.push(newInfo)
        })
        //处理静态属性
        this.onlyTableData.forEach(item => {
          const newInfo = { attr_id: item.attr_id, attr_value: item.attr_vals }
          this.editForm.attrs.push(newInfo)
        })
        form.attrs = this.editForm.attrs
        const goodsId = window.sessionStorage.getItem('goodsId')
        // console.log(form);
        //发起请求修改商品
        //商品的名称，必须是唯一的
        const { data } = await this.$http.put(`goods/${goodsId}`, form)
        console.log(data)

        if (data.meta.status !== 200) {
          return this.$message.error('修改商品失败')
        }
        this.$message.success('修改商品成功')
        this.$router.push('/goods')
      })
    }
  },
  computed: {
    cateId() {
      if (this.editForm.goods_cat.length === 3) {
        return this.editForm.goods_cat[2]
      }
      return null
    }
  }
}
</script>

<style lang="less" scoped>
.el-checkbox {
  margin: 0 10px 0 0 !important;
}
.previewImg {
  width: 100%;
}
.btnEdit {
  margin-top: 15px;
}
</style>