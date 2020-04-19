<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 警告区域 -->
      <el-alert title="注意：只允许第三级分类设置相关参数" type="warning" show-icon :closable="false"></el-alert>
      <!-- 选择商品分类区域 -->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <!-- 选择商品分类级联选择框 -->
          <!-- options绑定数据源 -->
          <el-cascader
            v-model="selectCateKeys"
            :options="catelist"
            :props="{ expandTrigger: 'hover', value:catePros.value,label:catePros.label ,children:catePros.children}"
            @change="handleChange"
          ></el-cascader>
        </el-col>
      </el-row>
      <!-- tab标签页区域 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!-- 添加动态参数面板 -->
        <el-tab-pane label="动态参数" name="many">
          <!-- 添加参数按钮 -->
          <el-button type="primary" :disabled="isBtnDisabled" @click="addDialogVisible = true">添加参数</el-button>
          <!-- 添加参数表格 -->
          <el-table :data="manyTableData" style="width: 100%" stripe border>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 渲染Tag标签 -->
                <el-tag
                  closable
                  @close="handleClose(i,scope.row)"
                  v-for="(item,i) in scope.row.attr_vals"
                  :key="i"
                >{{item}}</el-tag>
                <!-- 输入的文本框 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                ></el-input>
                <!-- 添加按钮 -->
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                >+ New Tag</el-button>
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column prop="attr_name" label="参数名称"></el-table-column>
            <!-- 操作 -->
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  @click="showEditDialog(scope.row.attr_id)"
                >编辑</el-button>
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  @click="removeParams(scope.row.attr_id)"
                >删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 添加静态属性面板 -->
        <el-tab-pane label="静态属性" name="only">
          <!-- 添加属性按钮 -->
          <el-button type="primary" :disabled="isBtnDisabled" @click="addDialogVisible = true">添加属性</el-button>
          <!-- 静态属性表格 -->
          <el-table :data="onlyTableData" style="width: 100%" stripe border>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 渲染Tag标签 -->
                <el-tag
                  closable
                  @close="handleClose(i,scope.row)"
                  v-for="(item,i) in scope.row.attr_vals"
                  :key="i"
                >{{item}}</el-tag>
                <!-- 输入的文本框 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                ></el-input>
                <!-- 添加按钮 -->
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                >+ New Tag</el-button>
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column prop="attr_name" label="属性名称"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  @click="showEditDialog(scope.row.attr_id)"
                >编辑</el-button>
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  @click="removeParams(scope.row.attr_id)"
                >删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加参数对话框 -->
    <el-dialog
      :title="'添加' + textTitle"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogVClose"
    >
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <el-form-item :label="textTitle" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改参数对话框 -->
    <el-dialog
      :title="'修改' + textTitle"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogVClose"
    >
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
        <el-form-item :label="textTitle" prop="attr_name">
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
  data() {
    return {
      //获取商品分类数据
      catelist: [],
      //级联选择框的配置对象
      catePros: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      //级联选择框双向绑定
      selectCateKeys: [],
      //被激活标签页的名称
      activeName: 'many',
      //动态参数数据
      manyTableData: [],
      //静态参数数据
      onlyTableData: [],
      //控制显示添加参数对话框
      addDialogVisible: false,
      //添加参数表单对象
      addForm: {
        attr_name: ''
      },
      //添加参数表单验证规则对象
      addFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      },
      //控制显示修改参数对话框
      editDialogVisible: false,
      //修改参数表单对象
      editForm: {},
      //修改参数表单验证规则对象
      editFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    //获取所有商品分类列表
    async getCateList() {
      const { data } = await this.$http.get('categories')
      if (data.meta.status !== 200) {
        this.$message.error('获取商品分类失败')
      }
      this.catelist = data.data
      // console.log(this.catelist);
    },
    //级联选择框选中项变化，会触发这个函数
    async handleChange() {
      this.getParamsData()
    },
    //tab标签页单击事件函数
    handleTabClick() {
      this.getParamsData()
      //   console.log(this.activeName)
    },
    //获取参数别表数据
    async getParamsData() {
      if (this.selectCateKeys.length !== 3) {
        this.selectCateKeys = []
        this.manyTableData = []
        this.onlyTableData = []
        return
      }
      //根据所选分类的Id和当前所处的面板，获取对应的参数
      const { data } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        {
          params: { sel: this.activeName }
        }
      )
      if (data.meta.status !== 200) {
        return this.$message.error('获取参数列表失败')
      }
      data.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        item.inputVisible = false
        item.inputValue = ''
      })
      //   console.log(data.data)
      if (this.activeName === 'many') {
        this.manyTableData = data.data
      } else {
        this.onlyTableData = data.data
      }
    },
    //监听添加对话框的关闭事件
    addDialogVClose() {
      this.$refs.addFormRef.resetFields()
    },
    //点击按钮，添加新的参数
    addParams() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data } = await this.$http.post(
          `categories/${this.cateId}/attributes`,
          {
            attr_name: this.addForm.attr_name,
            attr_sel: this.activeName
          }
        )
        if (data.meta.status !== 201) {
          this.$message.error('添加新参数失败')
        }
        this.$message.success('添加新的参数成功')
        this.getParamsData()
        this.addDialogVisible = false
      })
    },
    //点击按钮，展示修改参数对话框
    async showEditDialog(attr_id) {
      const { data } = await this.$http.get(
        `categories/${this.cateId}/attributes/${attr_id}`
      )
      if (data.meta.status !== 200) {
        return this.$message.error('获取参数失败')
      }
      this.editForm = data.data
      this.editDialogVisible = true
    },
    //监听修改对话框的关闭事件
    editDialogVClose() {
      this.$refs.addFormRef.resetFields()
    },
    //点击按钮修改参数
    editParams() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data } = await this.$http.put(
          `categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
          {
            attr_name: this.editForm.attr_name,
            attr_sel: this.activeName
          }
        )
        if (data.meta.status !== 200) {
          return this.$message.error('更新参数失败')
        }
        this.$message.success('更新参数成功')
        this.getParamsData()
        this.editDialogVisible = false
      })
    },
    //点击按钮删除参数
    async removeParams(attr_id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该参数, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      // console.log(confirmResult);
      //如果用户确认删除，则返回值为字符串  confirm
      //如果用户取消删除，则返回值为字符串  cancel

      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data } = await this.$http.delete(
        `categories/${this.cateId}/attributes/${attr_id}`
      )
      if (data.meta.status !== 200) {
        return this.$message.error('删除参数失败')
      }
      this.$message.success('删除参数成功')
      this.getParamsData()
    },
    //文本框失去焦点或者按下Enter都会触发
    async handleInputConfirm(row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      //如果没有return则证明输入了内容 做后续处理
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      //需要发起请求，保存这次操作
      this.saveAttrVals(row)
    },
    //对attr_vals的操作，保存到数据库
    async saveAttrVals(row) {
      const { data } = await this.$http.put(
        `categories/${this.cateId}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(' ')
        }
      )
      if (data.meta.status !== 200) {
        return this.$message.error('修改参数项失败')
      }
      this.$message.success('修改参数项成功')
    },
    showInput(row) {
      row.inputVisible = true
      //文本框自动获得焦点
      //$nextTick 方法的作用，就是当页面上的元素被重新渲染之后
      //才会指定回调函数中的代码
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    //删除对应参数的可选项
    handleClose(i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrVals(row)
    }
  },
  computed: {
    //如果按钮需要被禁用则返回true ，否则返回false
    isBtnDisabled() {
      if (this.selectCateKeys.length !== 3) {
        return true
      }
      return false
    },
    //当前选中的三级分类的Id
    cateId() {
      if (this.selectCateKeys.length === 3) {
        return this.selectCateKeys[2]
      }
      return null
    },
    textTitle() {
      if (this.activeName === 'many') {
        return '动态参数'
      } else {
        return '静态属性'
      }
    }
  }
}
</script>

<style lang="less" scoped>
.cat_opt {
  margin: 15px 0;
}
.input-new-tag {
  width: 120px;
}
</style>